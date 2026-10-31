# DEVELOPMENT MODE — Módulo Development/IA

**Estado: 📝 Planejado**

Módulo voltado ao PERFIL B: desenvolvedor que roda stack pesada **no mesmo Windows** — Kiro, VS Code, Python, Node, Git, Ollama (modelos locais como Qwen3) e Docker — sem sacrificar estabilidade do sistema.

## 1. Filosofia

- **Convivência, não guerra**: o objetivo não é "matar tudo que não é dev", é fazer IDE + IA local + Docker + browser coexistirem sem thrash de RAM/VRAM/disco.
- **Detecção antes de sugestão**: o módulo só recomenda para ferramentas que **detectou** (RF-DEV-01). Nenhuma presunção de stack.
- **Mesmas regras do produto**: tudo classificado SAFE/ADVANCED/EXPERT via catálogo, com justificativa e rollback.

## 2. Detecção de ferramentas (RF-DEV-01)

| Ferramenta | Como detectar (planejado) |
|---|---|
| Kiro | executável no PATH / pasta de instalação do usuário |
| VS Code | instalador por usuário/sistema, `code` no PATH, extensões (leitura, sem modificar) |
| Python | launchers (`py`, `python`), versões instaladas, pip cache size |
| Node.js | `node`/`npm` no PATH, versão, npm/pnpm/yarn caches |
| Git | `git` no PATH, config global de credenciais (apenas existência, nunca leitura de segredo) |
| Ollama | serviço/processos, modelos baixados (leitura de manifesto local), uso de RAM/VRAM |
| Docker | Docker Desktop, engine, WSL2 backend, tamanho de VHDX |

Cada detecção vira card no módulo com versão detectada, consumo atual (quando mensurável) e atalhos de manutenção.

## 3. Modelos locais e limites (RF-DEV-02)

Base de hardware da máquina de referência: 16 GB RAM, RTX 2060 6 GB VRAM, Ryzen 5 5600X.

| Cenário | Recomendação gerada (exemplo de regra) |
|---|---|
| VRAM 6 GB + modelo &gt; 4B Q8 | Sugerir quantização menor (ex.: Q4) ou offload parcial para CPU com aviso de velocidade |
| RAM 16 GB + IDE + Docker + modelo 7B | Alertar pressão de memória; sugerir ordem de grandeza de limite para `--num_ctx`/workers |
| Modelos não usados há X dias | Listar para remoção manual (o app nunca remove modelo automaticamente) |

Regras são **sugestivas e explicadas** ("por que este limite"): nunca aplicadas sem consentimento. Medições reais de RAM/VRAM vêm do Monitoring (envelope `SensorReading<T>`).

## 4. Otimizações de ambiente dev (RF-DEV-03)

Catálogo de tweaks específicos do perfil (classificação prevista):

| Tweak | Nível | Justificativa |
|---|---|---|
| WSL2: ajustar `.wslconfig` (memória/CPUs) | ADVANCED | Evita que WSL2 reserve RAM além do necessário em máquina de 16 GB; valores sugeridos conforme hardware |
| Docker Desktop: revisar recursos alocados | ADVANCED | Mesmo motivo do WSL2 (backend compartilhado) |
| Caches de pacotes (npm/pip/NuGet): relatório de tamanho + limpeza guiada | SAFE (dry-run) | Caches crescem silenciosamente; limpeza é segura e reversível por re-download |
| Desfragmentação de VHDX (Docker/WSL) via helper elevado | ADVANCED | Disco virtual cresce e fragmenta; compactação recupera espaço real |
| Índices do Windows Search sobre pastas de código: manter e revisar escopo | SAFE | Busca rápida no IDE vs custo de indexação — decisão informada do usuário |
| Telemetria de ferramentas dev (quando a própria ferramenta expõe opt-out) | SAFE | Privacidade; cada item aponta a configuração oficial da ferramenta |

## 5. Convivência Gaming ↔ Dev

A máquina de referência alterna entre os dois perfis. Regras:

- Perfis são **conjuntos de tweaks independentes**; ativar Gaming não desfaz o ambiente dev (e vice-versa).
- Conflitos explícitos (ex.: HVCI off para gaming vs WSL2 que prefere virtualização): a UI **mostra o trade-off** e deixa o usuário escolher por sessão — com rollback em 1 clique.
- Benchmark Lab registra em qual perfil cada medição foi feita.

## 6. Critérios de aceitação do módulo

1. Na máquina de referência, detecta corretamente Kiro/VS Code/Python/Node/Git/Ollama/Docker presentes.
2. Sugestões de limites de modelo citam a medição de RAM/VRAM real que as motivou.
3. Nenhum arquivo de projeto, modelo ou cache é removido sem dry-run + confirmação.
4. Nada no módulo exige que o usuário desligue segurança do sistema.
