---
inclusion: always
---

# Produto — NEXUS OPTIMIZER

Plataforma desktop profissional de otimização do Windows 11 para dois perfis: **Gaming competitivo** (PUBG, anti-stutter, frametime estável, baixa latência) e **Development/IA** (Kiro, VS Code, Python, Node, Git, Ollama/Qwen3, Docker na mesma máquina).

**Projeto comunitário, sem fins lucrativos, repositório público.** Filosofia completa: `docs/PHILOSOPHY.md` — "Knowledge Should Be Free". Leia antes de qualquer sessão.

## Princípios inegociáveis (nunca sugerir código que viole)

1. **Nunca inventar métricas** — sensor indisponível = UI mostra "indisponível" com motivo, nunca 0/placeholder/simulado. Todo sensor trafega em `SensorReading<T>` com `IsAvailable` (ver docs/ARCHITECTURE.md §4).
2. **Toda mutação é reversível** — passa pelo portão único `IMutationExecutor`: valida → consentimento (ADVANCED/EXPERT) → snapshot → aplica → auditoria → rollback disponível.
3. **Sem promessa de FPS** — otimizações têm justificativa técnica e são validadas por medição; sem ganho mensurável, sugerir reversão.
4. **Lista bloqueada permanente** — nunca oferecer mexer em: Defender, Windows Update, Firewall, anti-cheat, BIOS, overclock/undervolt do usuário.
5. **Sem admin por padrão** — app principal roda sem elevação; operações privilegiadas vão para `Nexus.Elevated.exe` (helper separado, whitelist compilada, named pipe).

## Diretrizes da filosofia do projeto (docs/PHILOSOPHY.md §10 — valem para IAs e humanos)

- **Não introduzir monetização** por conta própria (nada de assinatura, anúncio, telemetria comercial, dependência paga obrigatória).
- **Não esconder conhecimento técnico** — documentação explica o *como* e o *porquê*, não só o *o quê*; nada de doc superficial para preencher arquivo.
- **Não dificultar contribuições** — iniciantes são público-alvo explícito; explicar decisões de arquitetura; não descartar contribuição sem justificativa técnica.
- **Comprometer a transparência nunca** — sem telemetria de saída, sem dados do hardware saindo do PC.
- **Licença sob avaliação** (docs/PHILOSOPHY.md §6) — recomendação MIT aguardando aprovação do mantenedor; **não criar arquivo `LICENSE` sem aprovação expressa**.
- **Zero dados sensíveis no repositório** — sem senhas, chaves, tokens, `.env`, nomes de usuário ou caminhos pessoais da máquina do mantenedor (specs de hardware foram autorizadas).
- Solução simples, livre, segura e eficiente antes de alternativa comercial. Gratuidade não é desculpa para descuido.

## Fontes de verdade

- Visão e escopo: `docs/PRODUCT_VISION.md` · Requisitos: `docs/REQUIREMENTS.md`
- Segurança/reversibilidade: `docs/SECURITY.md` · Decisões (ADRs): `docs/DECISIONS.md`
- Filosofia/licença/comunidade: `docs/PHILOSOPHY.md` · Contribuição: `CONTRIBUTING.md`
- Status por fase: `docs/ROADMAP.md` — **nunca marcar nada como implementado sem código real**; usar selos 📝/🚧/✅/🧪/✔️.
