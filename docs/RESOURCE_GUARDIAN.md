# RESOURCE GUARDIAN & GAMING PRE-FLIGHT

> Gerenciamento inteligente de processos, monitoramento de recursos e preparação segura para jogos.

**Estado: 📝 Planejado — específicado em 2026-10-31, nenhum código escrito. Implementação: FASE 6B do ROADMAP (após fundação, diagnóstico, monitoramento, segurança e gaming).**

Este documento é a especificação normativa do módulo. Nada aqui descreve funcionalidade implementada.

## 1. O que é (e o que não é)

O Resource Guardian **não é um limpador de processos**. É um gerenciador que **entende o ambiente** antes de agir: o que está instalado, o que está executando, o que está de fato consumindo recursos e o que pode ser pausado com segurança — e o que **jamais** pode ser tocado.

Sequência obrigatória para **qualquer** operação que modifique o estado do computador:

```text
IDENTIFICAR → ENTENDER → RECOMENDAR → AUTORIZAR → EXECUTAR → VERIFICAR → RESTAURAR
```

Ela se concretiza na arquitetura existente (docs/ARCHITECTURE.md §5): toda ação do Guardian é uma `MutationRequest` que passa pelo portão único `IMutationExecutor` (valida → consentimento → snapshot → aplica → auditoria → rollback). O Guardian adiciona a frente intelectual (as três primeiras etapas) e operações de ciclo de vida controladas (§5) que se tornam entradas de snapshot/restauração.

## 2. Detecção inteligente de aplicativos

### 2.1 Fontes de inventário (leitura, sem elevação)

| Alvo | Fonte técnica | Notas |
|---|---|---|
| Win32 instalados | Registro `Uninstall` nas 3 hives (HKLM64, HKLM32, HKCU) | Mesma base da RF-DIAG-05 — reutilizada |
| Apps Microsoft Store | `Windows.Management.Deployment.PackageManager` via CsWinRT | Se indisponível → inventário parcial honesto ("Store: indisponível"), nunca varredura de disco improvisada |
| Serviços | Service Control Manager (API nativa) + WMI | Estado, tipo de início, processo hospedeiro |
| Inicialização | Chaves `Run`/`RunOnce`, `StartupApproved`, pastas Startup | Aponta impacto documentado de boot |
| Tarefas agendadas | COM Task Scheduler (`TaskService`) | Apenas tarefas não-Microsoft em primeiro plano |
| Processos ativos | Toolhelp32 / APIs de processo | §3 |
| Gerenciadores de pacotes | winget (leitura), quando presente | Somente consulta de inventário, nunca install/remove automático |
| Ambientes de virtualização | `hcsdiag`/WMI, estado Hyper-V, presença de VHDX referenciado | WSL2/Docker também em §2.3 |
| Servidores locais | Portas LISTEN de processos inventariados (API `GetExtendedTcpTable`) | Correlaciona processo↔porta sem inspecionar conteúdo |

**Regras duras:**

- **Sem varredura indiscriminada de arquivos pessoais.** Inventário vem de fontes de registro/APIs, não de caminhar o disco do usuário.
- **Instalação ≠ execução ≠ consumo.** São três estados distintos no modelo de dados: um app pode estar instalado e fechado; aberto e ocioso; aberto e pesado. A UI nunca mistura.
- Novas instalações são detectadas por **re-inventário periódico** (diff de hives de uninstall e Startup), não por watchers de filesystem.

### 2.2 Modelo de inventário (classificação por processo)

Cada processo observado é rotulado com flags acumuláveis:

```text
INSTALADO · EM_EXECUÇÃO · SERVIÇO_ATIVO · EM_SEGUNDO_PLANO ·
CONSUMINDO · USANDO_GPU · USANDO_MEM_COMPARTILHADA · CARGA_INTENSIVA
```

### 2.3 Detecção especial de IA local (prioridade do módulo)

Ambientes de IA consomem RAM/VRAM/CPU/disco em grande escala e são o conflito nº 1 entre os perfis Dev e Gaming do produto. Detecção por **integrações específicas plugáveis**, nunca por heurística de nome:

| Alvo | Sinais de instalação | Sinais de execução/consumo |
|---|---|---|
| Ollama | `ollama.exe` no PATH, `%LocalAppData%\Programs\Ollama`, serviço/app | **API local oficial** `http://localhost:11434`: `/api/ps` (modelos **carregados** + RAM/VRAM), `/api/tags` (modelos **armazenados**) |
| LM Studio | Diretório de instalação padrão, processo | API do servidor local LM Studio (OpenAI-compatible), quando o servidor estiver ativo |
| ComfyUI | Pasta/portátil identificável, `python.exe` com comando ComfyUI | Fila/estado via endpoint local do ComfyUI; consumo via processo Python hospedeiro |
| Stable Diffusion / A1111 / TextGen WebUI | Padrões de diretório + processo Python com argumentos característicos | Estado via endpoints locais quando expostos; senão consumo bruto do processo |
| Python + PyTorch / TensorFlow | `torch`/`tensorflow` importados? **Não** inspecionamos scripts do usuário | Processo Python com VRAM/CPU de inferência identificável → rotulado `POSSIVEL_IA` (honestidade: sem afirmar o que é) |
| Docker | Serviço `com.docker.service`, Docker Desktop | Containers ativos via **Docker Engine API** no named pipe local (`//./pipe/docker_engine`) |
| WSL2 | `wsl.exe -l -v` (somente leitura), estado `Running` | Distribuições em execução, consumo do processo `vmmemWSL` |

**Regras duras de IA:**

1. **Modelo no SSD ≠ modelo na memória.** Presença de arquivo de modelo nunca é apresentada como uso de VRAM/RAM — só a API oficial (`/api/ps` e equivalentes) diz o que está carregado. (Cenário de teste 1.)
2. **Integrações somente por APIs locais oficialmente documentadas** e compatíveis com a versão detectada; versão desconhecida → integração "indisponível: versão não suportada", nunca tentativa cega.
3. **Nunca descarregar modelo nem pausar inferência em andamento** sem verificar estado e obter autorização explícita (Cenário de teste 4).
4. Toda integração implementa `ILocalAIProvider` (abaixo) — adicionar suporte a um novo app = novo provider, **sem alterar o núcleo**.

```csharp
public interface ILocalAIProvider
{
    string ProviderId { get; }              // "ollama", "lmstudio", "comfyui"...
    Task<DetectionResult> DetectAsync(CancellationToken ct);        // instalado? onde?
    Task<RuntimeState> GetStateAsync(CancellationToken ct);         // servidor ativo? modelos carregados? inferência em andamento?
    Task<SensorReading<UnloadResult>> UnloadAsync(ModelRef model, CancellationToken ct);   // via API oficial, se suportado
    Task<bool> CanPauseInferenceAsync(CancellationToken ct);        // capability honesta
}
```

## 3. Process Scanner

Varredura diagnóstica de todos os processos relevantes. Por processo:

| Campo | Fonte | Disponibilidade |
|---|---|---|
| Nome, PID, caminho do executável | Toolhelp32 | Sempre |
| Fabricante / assinatura digital | `WinVerifyTrust` + metadados de versão | Quando assinado; não assinado ≠ suspeito automaticamente |
| Processo pai | Toolhelp32 (PID pai + resolução) | Sempre* |
| CPU | PDH por processo (descoberta independente de idioma, RNF-10) | Envelope `SensorReading<T>` |
| RAM (working set) / memória comprometida | PDH / API de processo | Sempre* |
| Atividade de armazenamento | PDH I/O por processo | Envelope |
| GPU por processo | Contadores PDH `GPU Engine` por PID | Envelope (mirrors do Gerenciador de Tarefas) |
| **VRAM por processo** | **Não há API documentada confiável** no Windows | Exibido como **"indisponível"** — nunca estimado. Uso total de VRAM vem do NvAPI (GPU inteira) |
| Estado, serviços relacionados, app responsável | SCM + correlação PID↔serviço | Sempre |

*Sujeito a direitos de acesso — processo protegido do SO listado sem detalhes, com indicação.

### 3.1 Classificação de impacto (baseada em métrica + contexto, nunca em nome)

```text
CRITICAL  → consumo significativo E risco concreto de prejudicar o jogo (ex.: VRAM quase esgotada com jogo ativo)
HIGH      → consumo elevado de recursos
MEDIUM    → pode disputar recursos dependendo da carga
LOW       → impacto irrelevante no cenário analisado
ESSENTIAL → necessário ao Windows, segurança, drivers ou aplicação protegida (§6) — imune a ações
```

- Limiares são **relativos ao hardware detectado** (RAM total, VRAM) e ao **perfil ativo** (§8) — não existe número universal. Em 16 GB de RAM, 4 GB de modelo de IA é HIGH; a mesma leitura em 64 GB pode ser MEDIUM. (Motor de decisão, docs §7.)
- **Estar aberto não é ser prejudicial.** App ocioso a ~0% de CPU e RAM estável é LOW mesmo sendo "grande".
- Toda classificação carrega **explicação** ("por que"): métricas observadas, limiar aplicado, contexto. A UI mostra o porquê.

## 4. Gaming Pre-Flight Check

Fluxo em 7 etapas acionado pelo perfil **PUBG Competitive** (manual ou automático, §8):

| Etapa | Conteúdo | Produz |
|---|---|---|
| 1. HARDWARE | CPU/RAM/commit/GPU/VRAM livres, temperaturas, atividade de SSD, processos ativos | Baseline do momento (via envelopes) |
| 2. APLICAÇÕES | IAs abertas, servidores locais, containers, VMs, navegadores pesados, sync, Windows Update em andamento, downloads, builds, renders | Lista correlacionada de trabalhos |
| 3. ANÁLISE | Pressão de RAM/commit/VRAM/CPU/GPU/disco/picos × necessidade do jogo | Quem realmente disputa recurso |
| 4. RECOMENDAÇÕES | Ações ordenadas por prioridade (descarregar modelo ocioso → pausar inferência pausável → encerrar app pesado ocioso → parar serviço auxiliar dispensável) | Plano de ação |
| 5. AUTORIZAÇÃO | Cada item apresentado com: processo, motivo, recursos em uso, benefício esperado, risco, **trabalho não salvo?**, reversibilidade, como restaurar. Aprovar/ignorar **item a item** | Consentimento granular |
| 6. EXECUÇÃO | Somente aprovados; **métodos naturais primeiro** (API do app → WM_CLOSE → serviço Stop) — nunca kill por padrão | Operações via `IMutationExecutor` |
| 7. VERIFICAÇÃO FINAL | Segunda varredura: processo saiu? modelo descarregado? memória liberada de fato? serviços no estado esperado? recurso melhorou? dependência afetada? | Estado final honesto + auditoria |

Bloqueios automáticos do fluxo: se Windows Update está no meio de instalação, se a telemetria PDH está quebrada (métricas indisponíveis → recomendações limitadas e **declaradas** como limitadas), ou se o anti-cheat do jogo está em estado inconsistente — o Pre-Flight **avisa e não improvisa**.

## 5. Gestão inteligente de aplicações — as 4 operações

Todas passam pelo `IMutationExecutor` com snapshot de estado (o que foi pausado/descarregado/parado) para restauração posterior:

| Operação | O que faz | Pré-condições |
|---|---|---|
| **PAUSE** | Pausa processamento via mecanismo da própria aplicação (fila do ComfyUI, job do Docker) | App suporta pausa documentada; verificação de estado antes/depois |
| **UNLOAD** | Descarrega modelo de IA da memória via API oficial | Sem inferência ativa naquele modelo (ou pausa autorizada); confirmação de liberação medida |
| **GRACEFUL SHUTDOWN** | Encerramento normal: janelas abertas detectadas, trabalho não salvo **bloqueia** (Cenário 7), aguarda saída, confirma liberação | Sem diálogo de salvamento pendente; timeout configurável; nunca força |
| **RESTART** (app/serviço, **nunca PC**) | Registra estado → verifica dependências e trabalhos ativos → autoriza → reinicia → confirma retorno saudável | Autorização explícita; dependências mapeadas |

**Encerramento forçado** (kill) desabilitado por padrão: existe apenas como ferramenta EXPERT com confirmação explícita, justificativa registrada e nunca para itens da lista de proteção.

## 6. Lista de proteção (bloqueio arquitetural)

Nunca encerrar/pausar, sequer propor: processos e serviços essenciais do Windows, drivers, segurança (Defender, Firewall, autenticação), áudio, rede, componentes da interface (shell/DWM), **anti-cheat**, **o próprio jogo em execução**, e **qualquer processo cuja identidade não foi verificada**.

- Identidade **não confia no nome do executável**: valida caminho, **assinatura digital**, processo pai, serviço associado e contexto. `svchost.exe` fora do `System32` não é svchost.
- Itens ESSENTIAL aparecem na UI com motivo da imunidade — transparência, não buraco negro.
- Sobreposição com a lista bloqueada de tweaks (docs/SECURITY.md §2.1): Defender/Update/Firewall/anti-cheat já são intocáveis globalmente.

## 7. Motor de decisão (determinístico e explicável)

Regras verificáveis sobre: hardware detectado, RAM/commit/VRAM disponíveis, consumo atual, tipo de jogo, aplicações ativas, dependências, estado das tarefas e **perfil selecionado** (§8).

- **Sem valor universal** de RAM/VRAM para "fechar app" — os limiares derivam do hardware e do perfil.
- Toda conclusão carrega sua justificativa (regra disparada + métricas que a alimentaram), exibida na UI.
- **IA generativa nunca é autoridade para operação administrativa.** Execução depende de regras determinísticas + verificações de segurança + autorização do usuário.
- Reinicialização do Windows: **medida excepcional** — só recomendada com justificativa técnica (update/driver pendente, estado irrecuperável por meios menores, instabilidade persistente), apresentando motivo, benefício, consequência, trabalhos a salvar e momento sugerido. **Nunca automática, nunca durante partida, decisão sempre do usuário.**

## 8. Resource Guardian durante o jogo + perfis

### 8.1 Monitoramento em sessão (impacto mínimo)

- Loop leve: amostragem reduzida durante jogo em tela cheia exclusiva (padrão 2–5 s, configurável) e sem redesenho de gráficos em 1º plano. Implementado sobre `Nexus.Monitoring` pela camada `Nexus.ResourceMonitor` — mesma telemetria, ritmo adaptativo.
- Observa: processos novos, serviços que subiram de consumo, apps que retomaram processamento, novos jobs de IA, novas pressões de RAM/VRAM/disco.
- Processo pesado novo durante a partida → identificar → verificar origem → medir impacto → checar essencialidade → **notificação discreta** (nunca janela intrusiva sobre o jogo, nunca overlay) → ação segura proposta.
- **Nenhuma mutação automática durante partida** — exceto ações SAFE que o usuário **pré-autorizou explicitamente** nas configurações do perfil. Nunca interromper o jogo; nunca tocar no anti-cheat.

### 8.2 Game Mode automático

Configurável: detectar início do PUBG (presença do processo do jogo) → rodar Pre-Flight resumido → alertar → propor → preparar → monitorar → ao fechar o jogo: re-verificação, resumo do que foi pausado/encerrado e **restauração opt-in** (recarregar modelos, reiniciar serviços/apps). Nada que o usuário escolheu não restaurar volta sozinho.

### 8.3 Perfis de recursos (independentes e editáveis)

| Perfil | Regra-mestra |
|---|---|
| GAMING | Máxima disponibilidade de recursos e estabilidade para jogos |
| DEVELOPMENT | Preserva toolchains, IDEs e serviços de desenvolvimento |
| LOCAL AI | Preserva modelos carregados e servidores de inferência |
| BALANCED | Cotidiano, sem alterações invasivas (padrão) |
| CUSTOM | Regras próprias do usuário, mesma estrutura dos demais |

Perfis alteram limiares e lista de preservação; nunca alteram a lista de proteção (§6), que é absoluta.

## 9. Interface (seção própria na navegação)

Novo item **Guardian** na navegação (UI_UX.md §3 passa a 10 itens). Subseções:

1. Live Resource Monitor · 2. Running Processes · 3. Heavy Applications · 4. Local AI Detection · 5. Gaming Pre-Flight · 6. Active Background Tasks · 7. Resource Recommendations · 8. Application Management · 9. Restart Manager · 10. Activity History.

Visão em tabela: processo · tipo de aplicação · CPU · RAM · GPU · VRAM (ou "indisponível") · impacto · recomendação · estado — com ordenação por consumo, busca e filtros, e o "por quê" sempre a um clique.

## 10. Privacidade e desempenho

- **100% local**: nada de processos, apps, hardware ou arquivos sai do PC (RNF-05). Sem inspeção de conteúdo de documentos.
- Histórico local em SQLite, retenção configurável; auditoria de operações na tela Security.
- Baixo footprint: monitoramento adaptativo, intervalos configuráveis, tudo assíncrono com cancelamento, proteção contra travamento de telemetria, tratamento de erro com degradação honesta (sensor falha → "indisponível", módulo segue).

## 11. Arquitetura — novos projetos

Doc §3 do ARCHITECTURE.md ganha:

| Projeto | Responsabilidade |
|---|---|
| **Nexus.ProcessScanner** | Inventário de processos + atributos (PID, pai, assinatura, serviços, I/O, GPU PDH) |
| **Nexus.ApplicationInventory** | Instalados (3 hives, Store, Startup, tarefas), diff de novas instalações |
| **Nexus.AIIntegration** | `ILocalAIProvider` + providers (Ollama primeiro) — núcleo isolado dos providers |
| **Nexus.ResourceGuardian** | Classificação de impacto, motor de regras, recomendações, perfis, orquestração das 4 operações (via `IMutationExecutor`) |
| **Nexus.GamingPreflight** | Fluxo das 7 etapas + Game Mode automático + restauração pós-jogo |
| **Nexus.ResourceMonitor** | Amostragem adaptativa em sessão de jogo sobre `Nexus.Monitoring` (sem duplicar telemetria) |
| **Nexus.RestartManager** | Reinícios controlados de app/serviço + recomendação excepcional de reboot do Windows |

Regras de dependência preservadas: módulos dependem só de Abstractions/Core; mutação só pelo portão; SO real só em Infrastructure/Elevated (providers de IA falam HTTP/named pipe locais — fronteira de rede **loopback apenas** documentada, nunca saída externa).

## 12. Cenários de teste obrigatórios

| # | Cenário | Resultado esperado |
|---|---|---|
| 1 | Ollama instalado, modelo baixado, **sem inferência** | Detecta instalação e armazenamento; **não** afirma uso de GPU |
| 2 | Ollama com modelo carregado (RAM+VRAM) | Identifica processo/impacto via API; oferece UNLOAD |
| 3 | PUBG com Docker + navegador + dev tools abertos | Diferencia pesados/leves/essenciais |
| 4 | PUBG com inferência em andamento | Informa operação ativa; alternativas sem interromper silenciosamente |
| 5 | App pesado abre **durante** a partida | Detecção + notificação discreta |
| 6 | Usuário aprova encerramento | Tenta encerramento normal, verifica, registra auditoria |
| 7 | App com documentos não salvos | **Não** força encerramento |
| 8 | Alvo é essencial ao Windows/anti-cheat | Operação bloqueada, com motivo |
| 9 | PUBG fechado | Opções de restauração do que foi pausado |
| 10 | Sistema precisa de reboot | Somente informa; decisão do usuário |

## 13. Critérios de aceitação (selo ✔️)

Detectar instalados · identificar em execução · diferenciar instalado×ativo · detectar IAs suportadas · consultar hardware real · executar Pre-Flight · classificar impacto alto · oferecer ações seguras · registrar auditoria · verificar resultado · restaurar quando possível · proteger essenciais · nunca reiniciar automaticamente · demonstrar baixo impacto medido durante jogo (benchmark do próprio Guardian na máquina de referência).

## 14. Diretrizes de implementação

1. Arquitetura apresentada neste doc **antes** de código (feito — FASE 1).
2. Interfaces e providers primeiro; testes por integração; documentação por integração.
3. Nenhuma funcionalidade falsa na UI; nenhuma medição simulada; nenhum encerramento sem autorização.
4. PUBG, anti-cheat e trabalho em andamento do usuário são intocáveis.
5. Implementação por etapas na FASE 6B do ROADMAP, após fundação (FASE 2), diagnóstico (FASE 3), monitoramento (FASE 4), segurança (FASE 5) e gaming (FASE 6).
