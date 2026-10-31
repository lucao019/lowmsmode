# REQUIREMENTS — NEXUS OPTIMIZER

**Estado: 📝 Planejado** (nada implementado)

Status de cada requisito: 📝 Planejado · 🚧 Em desenvolvimento · ✅ Implementado · 🧪 Testado · ✔️ Validado.

## 1. Requisitos funcionais

### RF-DIAG — Diagnóstico

| ID | Requisito | Status |
|---|---|---|
| RF-DIAG-01 | O sistema deve inventariar CPU, placa-mãe, BIOS, RAM (canal/frequência/XMP), GPU + driver, discos + SMART, rede e versão do Windows | 📝 |
| RF-DIAG-02 | O sistema deve detectar tecnologias de segurança/virtuação: VBS, HVCI, Game Mode, HAGS, plano de energia ativo | 📝 |
| RF-DIAG-03 | O sistema deve analisar entradas de boot (Run keys, startup folders, tarefas agendadas de logon) e classificar impacto (seguro/candidato/risco) | 📝 |
| RF-DIAG-04 | O sistema deve verificar integridade de suporte à telemetria: contadores PDH, classes WMI de desempenho, presença dos arquivos de perfil por idioma | 📝 |
| RF-DIAG-05 | O sistema deve varrer as 3 hives de uninstall e classificar entradas: `SISTEMA`, `MSI`, `OK`, `ATENCAO`, `ORFAO_PROVAVEL` (regras em SECURITY) | 📝 |
| RF-DIAG-06 | Todo diagnóstico deve ser persistido com timestamp para comparação histórica | 📝 |

### RF-MON — Monitoramento

| ID | Requisito | Status |
|---|---|---|
| RF-MON-01 | Amostrar CPU (total/por núcleo), GPU (uso/clock/VRAM via NvAPI), RAM, disco (% tempo, fila), rede em intervalo configurável (padrão 1 s) | 📝 |
| RF-MON-02 | Temperaturas via plugin opcional (LibreHardwareMonitorLib); se ausente/sem admin, exibir "indisponível" | 📝 |
| RF-MON-03 | Toda leitura encapsulada em `SensorReading<T>` com `IsAvailable`, fonte e timestamp | 📝 |
| RF-MON-04 | Gráficos em tempo real não podem bloquear nem congelar a UI | 📝 |

### RF-OPT — Otimização

| ID | Requisito | Status |
|---|---|---|
| RF-OPT-01 | Catálogo de tweaks classificados SAFE / ADVANCED / EXPERT, cada um com: descrição, justificativa técnica, risco, pré-requisitos, impacto esperado, rollback | 📝 |
| RF-OPT-02 | Toda mutação passa pelo `IMutationExecutor`: snapshot → aplica → auditoria → disponibiliza rollback | 📝 |
| RF-OPT-03 | Tweaks ADVANCED/EXPERT exigem consentimento explícito com texto claro do que muda | 📝 |
| RF-OPT-04 | O sistema deve sugerir reverter qualquer tweak sem melhora mensurável (integração Benchmark) | 📝 |
| RF-OPT-05 | Bloquear por padrão (não oferecer): mexer em Defender, Windows Update, Firewall, anti-cheat, BIOS/OC do usuário | 📝 |

### RF-GAME — Gaming

| ID | Requisito | Status |
|---|---|---|
| RF-GAME-01 | Perfil PUBG competitivo: plano Alto Desempenho, DVR off, HVCI off (opcional), NIC power saving off, startup enxuto — tudo via catálogo, nunca silencioso | 📝 |
| RF-GAME-02 | Checklist de configuração manual guiada (NVCP, opções do jogo) apresentada como guia, não automação oculta | 📝 |
| RF-GAME-03 | O sistema não deve prometer ganhos de FPS; exibe apenas medições do Benchmark Lab | 📝 |

### RF-DEV — Development Mode

| ID | Requisito | Status |
|---|---|---|
| RF-DEV-01 | Detectar ferramentas: Kiro, VS Code, Python, Node, Git, Ollama (+ modelos, ex.: Qwen3), Docker | 📝 |
| RF-DEV-02 | Sugerir limites/perfis para consumo de RAM/VRAM de modelos locais conforme hardware | 📝 |
| RF-DEV-03 | Otimizações de ambiente dev: WSL2, Docker, caches de pacotes — classificadas como tweaks | 📝 |

### RF-CLN — System Cleaner

| ID | Requisito | Status |
|---|---|---|
| RF-CLN-01 | Simulação (dry-run) com tamanho/classificação antes de qualquer exclusão | 📝 |
| RF-CLN-02 | Categorias permitidas fase 1: Temp do usuário, caches de GPU (ex.: DXCache), lixeira, logs rotativos, DISM StartComponentCleanup (via helper elevado) | 📝 |
| RF-CLN-03 | Proibido: deleção automática de entradas de registro sem critérios múltiplos + backup por chave + confirmação por item ou lote explícito | 📝 |

### RF-SEC — Security & Restore

| ID | Requisito | Status |
|---|---|---|
| RF-SEC-01 | Ponto de restauração do sistema antes de qualquer lote de mutações | 📝 |
| RF-SEC-02 | Backup granular por operação (ex.: export `.reg` por chave) | 📝 |
| RF-SEC-03 | Tela de auditoria: linha do tempo de mutações, filtros, rollback por item | 📝 |
| RF-SEC-04 | Assistente "reverter tudo" baseado na trilha de auditoria | 📝 |

### RF-BEN — Benchmark

| ID | Requisito | Status |
|---|---|---|
| RF-BEN-01 | Capturar baseline antes de mudanças: boot (evento 100), CPU/disco/GPU em repouso, RAM em repouso | 📝 |
| RF-BEN-02 | Pós-medição deve exigir condições equivalentes (uptime, carga de processos, térmico) e alertar quando não equivalentes | 📝 |
| RF-BEN-03 | Relatório comparativo honesto: média, desvio, e aviso "variação dentro do ruído" quando aplicável | 📝 |

## 2. Requisitos não funcionais

| ID | Requisito | Status |
|---|---|---|
| RNF-01 | App principal executa **sem elevação**; elevação apenas via helper com whitelist | 📝 |
| RNF-02 | Nenhuma métrica simulada/estática pode ser exibida como real | 📝 |
| RNF-03 | UI em português brasileiro; termos técnicos podem permanecer em inglês | 📝 |
| RNF-04 | Telemetria não bloqueia UI; uso de RAM do app < 300 MB em repouso (alvo) | 📝 |
| RNF-05 | Sem envio de dados para fora da máquina; sem telemetria remota | 📝 |
| RNF-06 | Toda operação destrutiva requer consentimento explícito e é reversível | 📝 |
| RNF-07 | Logs locais com retenção limitada; sem dados sensíveis (credenciais, chaves) | 📝 |
| RNF-08 | Publicação single-file self-contained `win-x64`; não exigir SDK no destino | 📝 |
| RNF-09 | SDK alvo: .NET 10 LTS (máquina de referência: 10.0.401) | 📝 |
| RNF-10 | Contadores PDH/WMI devem ser descobertos independente de idioma do SO | 📝 |

## 3. Critérios de aceitação globais

1. Qualquer tweak do catálogo SAFE pode ser aplicado e revertido em máquina real sem intervenção manual externa.
2. Com contadores de desempenho corrompidos (cenário real reproduzível), o app informa "indisponível" com motivo e orienta reparo — nunca mostra números falsos.
3. Tela Security & Restore lista 100% das mutações realizadas pelo app com caminho de rollback.
