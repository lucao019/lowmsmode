# DECISIONS — NEXUS OPTIMIZER

**Estado: registro ativo**

Registro de decisões técnicas (ADR leve). Cada entrada: contexto, decisão, alternativas rejeitadas, consequências. Muitas decisões derivam de incidentes reais da sessão de origem (2026-10-31) — citados quando aplicável.

## ADR-001 — WPF + Wpf.Ui, não WinUI 3

- **Contexto:** app desktop Windows 11 com visual Fluent, single-file self-contained, helper elevado separado.
- **Decisão:** WPF com a biblioteca `Wpf.Ui` (Fluent), CommunityToolkit.Mvvm.
- **Rejeitado:** WinUI 3 — dependência do WinAppSDK e empacotamento MSIX complicam publicação single-file e a convivência com processo helper elevado. Avalonia/MAUI — fora do alvo "Windows-first".
- **Consequências:** ecossistema maduro, empacotamento simples com Inno Setup; UI Fluent consistente sem casar com runtime empacotado.

## ADR-002 — .NET 10 LTS

- **Contexto:** runtime alvo do projeto.
- **Decisão:** .NET 10 LTS, publicação single-file self-contained `win-x64` (SDK de referência: 10.0.401).
- **Consequências:** usuário final não instala nada; trimming/AOT avaliados na FASE 2 (WPF limita AOT completo — single-file é o caminho pragmático).

## ADR-003 — Sem elevação por padrão + helper com whitelist

- **Contexto:** otimizadores genéricos exigem admin total para tudo — superfície de dano desnecessária.
- **Decisão:** `Nexus.App` roda sem elevação; operações privilegiadas vão para `Nexus.Elevated.exe` via named pipe com **whitelist compilada no binário** e payload tipado (nunca shell livre). Detalhes em [SECURITY.md](SECURITY.md) §1.
- **Alternativa rejeitada:** app inteiro elevado (simples, mas quebra o princípio de privilégio mínimo e expõe UI a injeção).
- **Consequências:** 1 UAC por sessão; abort limpo se UAC negado; toda operação auditável.

## ADR-004 — Portão único de mutação (`IMutationExecutor`)

- **Contexto:** múltiplos módulos (Optimization, Gaming, Cleaner, Development) aplicam mudanças.
- **Decisão:** **nenhum** módulo chama SO diretamente para mutação. Tudo passa por `IMutationExecutor`: valida → consente (se ADVANCED/EXPERT) → snapshot → aplica → audita → disponibiliza rollback.
- **Consequências:** auditoria 100% coberta por construção (critério de aceitação global 3); rollback uniforme; módulos ficam testáveis com portão falso.

## ADR-005 — Honestidade de dados: `SensorReading<T>` com `IsAvailable`

- **Contexto (incidente real):** contadores PDH corrompidos em pt-BR — arquivos `perfc016.dat`/`perfh016.dat` ausentes do System32 **e** do WinSxS — derrubaram Get-Counter (qualquer idioma), classes WMI `Win32_PerfFormattedData_*` e o Gerenciador de Tarefas. Ferramentas comuns exibiriam zeros ou valores travados.
- **Decisão:** toda métrica trafega em envelope `SensorReading<T>` (`IsAvailable`, `Value`, `Source`, `Timestamp`, `UnavailableReason`). Se indisponível, a UI **exibe "indisponível" com motivo e orientação de reparo** — nunca 0, nunca último valor, nunca simulado (RNF-02, aceitação global 2).
- **Consequências:** custo de boilerplate em todo sensor; ganho de confiança absoluta nos números.

## ADR-006 — Descoberta de contadores independente de idioma (RNF-10)

- **Contexto (mesmo incidente):** nomes de contador localizados ("Processador", "Disco Físico"); PDH em pt-BR não tem fallback automático para en-US quando o catálogo local está corrompido.
- **Decisão:** enumerar contadores via `Get-Counter -ListSet`/`PdhEnum*` com padrões e wildcards, nunca strings fixas de idioma; fallback para classes WMI; falha reportada como indisponível (nunca exceção crua na UI).
- **Consequências:** código de descoberta mais complexo, mas robusto a idioma e a catálogos parciais.

## ADR-007 — NvAPIWrapper para GPU; LHM como plugin isolado

- **Contexto:** GPU NVIDIA na máquina de referência (RTX 2060). Temperaturas exigem driver de kernel (admin).
- **Decisão:** uso/clock/VRAM via **NvAPIWrapper** (user-mode, sem admin). Temperaturas via **LibreHardwareMonitorLib** como **plugin opcional e isolado**: se ausente ou sem admin, sensores exibem "indisponível" (RF-MON-02) e o app funciona normalmente.
- **Rejeitado:** embutir LHM como dependência obrigatória — quebraria o requisito "sem elevação por padrão".
- **Consequências:** AMD GPU fica para fase posterior (documentado como fora do escopo inicial).

## ADR-008 — Persistência SQLite + Dapper, migrações em código

- **Contexto:** histórico de diagnósticos, séries temporais, auditoria, benchmarks — tudo local.
- **Decisão:** SQLite (Microsoft.Data.Sqlite) + Dapper (queries explícitas); migrações versionadas via `PRAGMA user_version` em código, sem ORM pesado.
- **Rejeitado:** EF Core — overhead e magia desnecessários para schema pequeno e controle fino.
- **Consequências:** queries auditáveis, esquema simples de inspecionar, zero servidor.

## ADR-009 — Logging local apenas (Serilog rolling file)

- **Contexto:** diagnóstico em campo sem privacidade comprometida.
- **Decisão:** Serilog com rolling file em `%LocalAppData%\Nexus\logs`, retenção configurável, **nenhum envio externo** (RNF-05), nenhum dado sensível em log (RNF-07).
- **Consequências:** forense local suficiente; suporte depende de o usuário exportar logs manualmente.

## ADR-010 — Critérios múltiplos para órfãos de uninstall

- **Contexto (incidente evitado na sessão de origem):** heurística ingênua ("pasta não existe") marcaria falsos positivos entre entradas de uninstall.
- **Decisão:** classificação por pontuação com critérios múltiplos (SECURITY.md §4): `InstallLocation` inexistente + `UninstallString`/`DisplayIcon` quebrado + ausência de arquivos + publisher sem presença. Remoção só com export `.reg` individual por chave e confirmação. Resultado real: 15/15 removidas, 0 falsos positivos.
- **Consequências:** menos candidatos sugeridos, mas confiança alta; nenhuma deleção automática.

## ADR-011 — Nenhuma promessa de FPS; medição com condições equivalentes

- **Contexto:** marketing de otimizadores promete ganhos inventados.
- **Decisão:** o app **não promete ganho** (RF-GAME-03). Benchmark Lab só compara medições em condições equivalentes (uptime, carga, térmico — RF-BEN-02) e reporta "variação dentro do ruído" quando aplicável (RF-BEN-03). Tweak sem melhora mensurável → sugerir reversão (RF-OPT-04).
- **Consequências:** percepção honesta pode parecer "menos mágica" — é o posicionamento do produto.

## ADR-012 — Lista bloqueada permanente

- **Contexto:** segurança e estabilidade do usuário.
- **Decisão:** nunca oferecer mexer em Defender, Windows Update, Firewall, anti-cheat, BIOS/OC/undervolt (RF-OPT-05, escopo do produto). Não há flag que habilite — não existe caminho de UI.
- **Consequências:** escopo deliberadamente menor que concorrentes; risco operacional mínimo.

## ADR-013 — UI em pt-BR, termos técnicos em inglês

- **Contexto:** usuário-alvo brasileiro com vocabulário técnico misto.
- **Decisão:** interface em português brasileiro (RNF-03); nomes de tecnologias (VBS, HVCI, HAGS, XMP, frametime) permanecem em inglês quando é a forma usada pela comunidade.
- **Consequências:** glossário implícito; documentação segue a mesma regra.

## ADR-014 — Helper ocioso encerra; nada residente

- **Contexto:** minimizar superfície privilegiada.
- **Decisão:** `Nexus.Elevated.exe` é spawnado sob demanda e encerra após ociosidade configurável; o app principal não mantém serviço Windows residente.
- **Consequências:** uma UAC por sessão (não por operação); nada rodando quando o app fecha.

## ADR-015 — ~~Repositório privado~~ (REVOGADA por ADR-016)

- **Contexto:** estágio inicial do produto.
- **Decisão original (2026-10-31, início do dia):** repositório **privado por padrão**; licença definida antes de qualquer abertura (FASE 8).
- **Status:** **revogada no mesmo dia** pela ADR-016 — o mantenedor decidiu publicar o repositório aberto de imediato. Mantida aqui apenas para histórico: ADRs revogadas não são apagadas.

## ADR-016 — Repositório público e filosofia "Knowledge Should Be Free"

- **Contexto:** o mantenedor declarou que o projeto nasce para **compartilhar conhecimento**, não para gerar lucro, e quer a mentalidade e o "como usar" disponíveis para ajudar outras pessoas — inclusive iniciantes.
- **Decisão (2026-10-31):** repositório **público** desde a FASE 1 (só documentação — não há código nem builds para expor). Filosofia completa em [PHILOSOPHY.md](PHILOSOPHY.md). Documentos de comunidade adicionados: [CONTRIBUTING.md](../CONTRIBUTING.md), [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md) (Contributor Covenant 2.1 adaptado), [SUPPORT.md](../SUPPORT.md), política de reporte de vulnerabilidades em [.github/SECURITY.md](../.github/SECURITY.md) e templates de issue/PR em [.github/](../.github/).
- **Restrições da decisão:** nada sensível no repositório — sem senhas, API keys, tokens, arquivos `.env`, caminhos/usuários pessoais da máquina do mantenedor. Especificações de hardware do PC de referência são permitidas (aprovação explícita do mantenedor).
- **Licença:** **MIT**, aprovada pelo mantenedor em 2026-10-31 (avaliação em PHILOSOPHY.md §6; arquivo `LICENSE` na raiz).
- **Consequências:** o projeto aceita contribuição externa desde antes do primeiro código; regras de PR em CONTRIBUTING.md passam a ser o contrato. Selos de status honestos ficam ainda mais críticos (público pode confundir "planejado" com "pronto" — o README precisa deixar claro).

## ADR-017 — Resource Guardian como módulo de gerência de recursos com motor determinístico

- **Contexto:** o usuário convive com pesos reais no mesmo PC — IA local (Ollama), Docker/WSL, IDEs, navegadores — e joga competitivamente. A forma manual de preparar a máquina antes de jogar (FASE 0) é conhecimento valioso que precisa virar software **seguro**: um módulo que entenda o ambiente, recomende e só execute com autorização, podendo devolver tudo depois. Diretriz original do mantenedor: "não um simples limpador de processos".
- **Decisão (2026-10-31):** módulo **Resource Guardian** (docs/RESOURCE_GUARDIAN.md) com:
  1. **Sequência obrigatória** IDENTIFICAR → ENTENDER → RECOMENDAR → AUTORIZAR → EXECUTAR → VERIFICAR → RESTAURAR — materializada nas três primeiras etapas pelo módulo e nas demais pelo portão `IMutationExecutor` (ADR-005) com snapshots restauráveis das 4 operações de ciclo de vida (PAUSE, UNLOAD, GRACEFUL SHUTDOWN, RESTART).
  2. **Sete projetos novos** (ProcessScanner, ApplicationInventory, AIIntegration, ResourceGuardian, GamingPreflight, ResourceMonitor, RestartManager) — `ResourceMonitor` é uma camada de amostragem adaptativa sobre `Nexus.Monitoring`, deliberadamente sem duplicar o loop de telemetria (ADR-011).
  3. **Integrações de IA por providers** (`ILocalAIProvider`) usando apenas APIs locais oficiais em loopback; modelo no disco nunca é reportado como carregado (honestidade de métrica, ADR-002 estendida).
  4. **Motor de decisão determinístico e explicável** — regras relativas ao hardware/perfil; IA generativa proibida como autoridade de operações administrativas; toda recomendação carrega o "por quê".
  5. **Proteção dura** sobreposta à lista bloqueada (ADR-008): anti-cheat e jogo em execução intocáveis; nenhuma mutação automática durante partida; kill desabilitado por padrão (ferramenta EXPERT com justificativa); reboot do Windows apenas como recomendação, jamais automática.
  6. **VRAM por processo = "indisponível"** — não existe API documentada confiável; estimar seria inventar métrica.
- **Alternativas descartadas:** limpador/killer de processos genérico (viola princípios 1–2, risco a trabalho aberto e anti-cheat); varredura de disco para detectar IA (invasiva e frágil); overlays em jogo (intrusivo e arriscado com anti-cheat); automação agressiva estilo "modo turbo" (sem consentimento granular).
- **Consequências:** FASE 6B inserida no ROADMAP entre Otimização/Gaming e Development/Cleaner; navegação passa a 10 itens; tabelas novas (`app_inventory`, `guardian_actions`, `resource_profiles`, `game_sessions`); 10 cenários de teste normativos viram contrato de aceitação.
