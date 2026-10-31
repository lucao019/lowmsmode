---
inclusion: always
---

# Estrutura do repositório — NEXUS OPTIMIZER

```text
nexus-optimizer/
├── docs/                      # fonte de verdade do produto (12 documentos)
│   ├── PRODUCT_VISION.md      # visão, perfis, escopo, métricas de sucesso
│   ├── ARCHITECTURE.md        # módulos, elevação, persistência, empacotamento
│   ├── REQUIREMENTS.md        # RF-* e RNF-* com status 📝/🚧/✅/🧪/✔️
│   ├── SECURITY.md            # privilégio mínimo, whitelist, backups, auditoria
│   ├── ROADMAP.md             # FASE 0..8 com critérios de saída
│   ├── DECISIONS.md           # ADR-001..015
│   ├── UI_UX.md               # identidade visual e padrões de componentes
│   ├── GAMING_OPTIMIZATION.md # módulo Gaming / perfil PUBG
│   ├── DEVELOPMENT_MODE.md    # módulo Dev/IA
│   ├── HARDWARE_MONITORING.md # diagnóstico e monitoramento
│   └── BENCHMARK.md           # estratégia de medição antes/depois
├── src/                       # solução .NET (FASE 2 — ainda vazio)
│   └── (Nexus.App, Nexus.Abstractions, Nexus.Core, Nexus.Application,
│        Nexus.Infrastructure, Nexus.Hardware, Nexus.Monitoring,
│        Nexus.Gaming, Nexus.Development, Nexus.Cleaner,
│        Nexus.Optimization, Nexus.Security, Nexus.Benchmark,
│        Nexus.Persistence, Nexus.Elevated)
├── tests/                     # Nexus.Core.Tests, Nexus.Application.Tests,
│                              # Nexus.Optimization.Tests, Nexus.Benchmark.Tests
├── scripts/                   # utilitários de desenvolvimento (PowerShell)
├── assets/                    # ícones e imagens
├── installer/                 # nexus.iss (Inno Setup; output/ ignorado)
└── .kiro/
    ├── steering/              # este contexto permanente
    └── specs/nexus-optimizer/ # spec ativa (requirements/design/tasks)
```

## Regras de organização

- Dependência permitida: módulos dependem de **Nexus.Abstractions / Nexus.Core**, nunca uns dos outros diretamente; acoplamento via **Nexus.Application**.
- **Nenhum** módulo chama o SO diretamente para mutação — sempre via `IMutationExecutor`.
- Lógica de domínio não pode depender de SO real (interfaces de ambiente injetadas): requisito de testabilidade (docs/ARCHITECTURE.md §8).
- `.gitignore` bloqueia bin/obj/.vs, logs, segredos, `*.reg`, dumps de máquina. **Nunca** versionar dados privados do PC de desenvolvimento.
