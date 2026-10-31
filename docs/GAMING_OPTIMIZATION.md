# GAMING OPTIMIZATION — Módulo Gaming (perfil PUBG competitivo)

**Estado: 📝 Planejado**

Módulo voltado ao PERFIL A: jogador competitivo de PUBG que prioriza **anti-stutter, frametime estável e baixa latência de entrada**. Toda ação sai do catálogo de tweaks auditáveis — nada é aplicado silenciosamente (RF-GAME-01).

## 1. Filosofia

- **Sem promessa de FPS** (RF-GAME-03): o módulo nunca exibe "+X FPS". O que ele faz: aplica otimizações com justificativa técnica e mede o resultado real no Benchmark Lab. Sem ganho mensurável → sugere reverter (RF-OPT-04).
- **Estabilidade > pico**: frametime consistente vale mais que média maior com micro-stutter.
- **Tudo reversível**: cada item tem rollback de 1 clique na tela Security & Restore.

## 2. Perfil PUBG competitivo — conjunto-alvo

Origem: tuning aplicado e validado na sessão de origem (2026-10-31) com script completo de reversão.

| Item | Tipo | Nível | Justificativa técnica |
|---|---|---|---|
| Plano de energia **Alto Desempenho** | powercfg | SAFE | Elimina transições de frequência/parked cores durante o jogo; latência de rampa de clock tende a zero |
| **Game DVR / captura background off** | registro (GameDVR) | SAFE | Remove captura contínua da Xbox Game Bar: menos overhead de GPU, CPU e E/S |
| **Game Bar / notificações em jogo** | configuração | SAFE | Menos overlays e hooks no pipeline de renderização |
| **NIC power saving off** (energia do adaptador de rede) | driver/PNP | ADVANCED | Evita micro-interrupções de energia no adaptador — latência de rede estável em jogo online |
| **HVCI off** (opcional) | segurança de memória | EXPERT | Reduz custo de virtualização de código no kernel; ganho possível em frametime. **Trade-off de segurança explicitado no consentimento** — decisão do usuário |
| **Startup enxuto** | boot entries | SAFE→ADV | Menos processos residentes disputando CPU/disco em sessão de jogo; classificado por diagnóstico, nunca removido sem confirmação item a item |
| **Storage Sense / manutenção automática fora do jogo** | configuração | SAFE | Impede tarefas de manutenção disparando durante sessão de jogo |
| **FOG (Fullscreen Optimizations)** | configuração | SAFE | Uso de **modo exclusivo fullscreen** quando o jogo suporta, evitando compositor DWM no caminho do frame |

Todos os itens passam por `IMutationExecutor` com snapshot + auditoria + rollback (RF-OPT-02).

## 3. O que o módulo NÃO faz (lista bloqueada herdada)

- Anti-cheat (PUBG usa anti-cheat em kernel): **não inspecionar, não burlar, não tocar** — fora do escopo do produto.
- Defender / Windows Update / Firewall: bloqueados por padrão (RF-OPT-05). Guias online que mandam "desativar o Defender para FPS" são explicitamente **contra-indicados** na UI.
- BIOS, overclock, undervolt: domínio do usuário; o app no máximo exibe leitura informativa (monitor já tem XMP ativo — o app nunca altera).
- "Otimizadores de memória" (RAM cleaners): não existem como tweak — explicados como placebo/risco na base de conhecimento.

## 4. Checklists manuais guiados (RF-GAME-02)

Itens que o app **não automatiza**, apresentados como guia passo a passo (nunca como automação oculta):

### 4.1 NVIDIA Control Panel (NVCP)

| Configuração | Valor sugerido | Motivo |
|---|---|---|
| Power management mode | Prefer maximum performance | Clock estável durante o jogo |
| Low Latency Mode | Ultra (ou reflex do jogo quando disponível) | Reduz fila de renderização → latência de entrada |
| Vertical sync | Off (em jogo competitivo) | Evita latência adicional do vsync |
| Shader cache | On (tamanho padrão) | Menos stutter de compilação de shader |
| Monitor | Taxa máxima nativa, G-Sync conforme preferência | Display da referência: 1080p60 |

### 4.2 Opções do PUBG (guia em texto)

- Modo de exibição: fullscreen (não borderless) quando estável para o usuário.
- Texturas/sombras/pós-processamento: presets baixos nos itens de maior custo de frametime, mantendo visibilidade competitiva.
- FOV e render scale: padrões do usuário — o guia não prescreve; apenas aponta impacto de cada um.

## 5. Verificações anti-stutter (diagnóstico do módulo)

| Verificação | Fonte |
|---|---|
| Integridade dos contadores de desempenho (telemetria base para medir) | RF-DIAG-04 — se quebrada, Gaming não promete medição e aponta reparo |
| Estado do Game Mode / HAGS / VBS / HVCI | RF-DIAG-02 |
| Plano de energia ativo | powercfg |
| Processos de startup com impacto | RF-DIAG-03 |
| GPU: driver, clocks, VRAM | NvAPI (sem admin) |
| Disco: fila e % tempo durante jogo | PDH (quando disponível) |

## 6. Medição (integração com Benchmark Lab)

Fluxo honesto:

1. **Baseline** antes de aplicar o perfil (condições registradas: uptime, carga, térmico).
2. Aplicação do perfil via catálogo.
3. **Pós-medição** exigindo condições equivalentes (RF-BEN-02) — se uptime/carga/térmico divergirem, o app **alerta** e não compara.
4. Relatório: média, desvio e aviso "variação dentro do ruído" quando aplicável (RF-BEN-03).
5. Tweak sem melhora mensurável → sugestão explícita de reversão (RF-OPT-04).

## 7. Critérios de aceitação do módulo

1. Perfil PUBG aplica e reverte ponta a ponta em máquina real sem intervenção externa.
2. Nenhum item do perfil toca a lista bloqueada.
3. Checklists manuais são visivelmente separados do que é automático.
4. Em hipótese alguma a UI exibe promessa ou estimativa de FPS antes da medição real.
