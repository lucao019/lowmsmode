---
inclusion: always
---

# Produto — NEXUS OPTIMIZER

Plataforma desktop profissional de otimização do Windows 11 para dois perfis: **Gaming competitivo** (PUBG, anti-stutter, frametime estável, baixa latência) e **Development/IA** (Kiro, VS Code, Python, Node, Git, Ollama/Qwen3, Docker na mesma máquina).

## Princípios inegociáveis (nunca sugerir código que viole)

1. **Nunca inventar métricas** — sensor indisponível = UI mostra "indisponível" com motivo, nunca 0/placeholder/simulado. Todo sensor trafega em `SensorReading<T>` com `IsAvailable` (ver docs/ARCHITECTURE.md §4).
2. **Toda mutação é reversível** — passa pelo portão único `IMutationExecutor`: valida → consentimento (ADVANCED/EXPERT) → snapshot → aplica → auditoria → rollback disponível.
3. **Sem promessa de FPS** — otimizações têm justificativa técnica e são validadas por medição; sem ganho mensurável, sugerir reversão.
4. **Lista bloqueada permanente** — nunca oferecer mexer em: Defender, Windows Update, Firewall, anti-cheat, BIOS, overclock/undervolt do usuário.
5. **Sem admin por padrão** — app principal roda sem elevação; operações privilegiadas vão para `Nexus.Elevated.exe` (helper separado, whitelist compilada, named pipe).

## Fontes de verdade

- Visão e escopo: `docs/PRODUCT_VISION.md` · Requisitos: `docs/REQUIREMENTS.md`
- Segurança/reversibilidade: `docs/SECURITY.md` · Decisões (ADRs): `docs/DECISIONS.md`
- Status por fase: `docs/ROADMAP.md` — **nunca marcar nada como implementado sem código real**; usar selos 📝/🚧/✅/🧪/✔️.
