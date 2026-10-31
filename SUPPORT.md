# Suporte — NEXUS OPTIMIZER

Obrigado por usar/contribuir! Este arquivo explica **onde pedir ajuda** e como fazer isso da melhor forma possível.

## Antes de perguntar

1. Leia o [README](README.md) — ele explica o projeto em linguagem simples.
2. Veja se a sua dúvida já está respondida em [docs/](docs/) (especialmente [docs/ROADMAP.md](docs/ROADMAP.md) para saber o que já existe vs. o que está planejado).
3. Busque nas [issues](https://github.com/lucao019/lowmsmode/issues) (abertas **e** fechadas) — sua pergunta pode já ter sido respondida.

> **Importante:** o aplicativo ainda **não tem código** — estamos na fase de documentação (FASE 1 ✔️). Não existe executável para baixar nem erro de execução para reportar ainda. Dúvidas sobre o planejamento, a arquitetura e a documentação são muito bem-vindas.

## Onde pedir ajuda

| Tipo | Onde | Observação |
|---|---|---|
| Pergunta / dúvida de uso ou conceito | Issue com template **Pergunta** (label `documentation` ou `question`) | Sem pergunta boba — o projeto existe para ensinar |
| Bug (quando houver código) | Issue com template **Relato de bug** (label `bug`) | Precisa de passos de reprodução |
| Incompatibilidade de hardware | Issue com template **Compatibilidade de hardware** | Informe CPU, GPU, RAM e versão do Windows |
| Sugestão de funcionalidade | Issue com template **Sugestão** (label `enhancement`) | Explique o problema que a sugestão resolve |
| Vulnerabilidade de segurança | **NUNCA issue pública** — siga [.github/SECURITY.md](.github/SECURITY.md) | Tratada em privado |
| Violação de conduta | Ver [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) § Aplicação | Confidencial para quem reporta |

## Como fazer um bom relato (aumenta muito a chance de resolução rápida)

- **Uma issue por problema.** Não misture assuntos.
- **Passos para reproduzir**: o que você fez, o que esperava, o que aconteceu.
- **Ambiente**: versão do Windows 11 (ex.: 23H2 build 22631), CPU, GPU, RAM, versão do .NET (se aplicável).
- **Evidência**: prints, logs (o app logará em `%LocalAppData%\Nexus\logs` — remova qualquer informação pessoal antes de anexar).
- **Honestidade de estado**: se é algo que você leu na documentação (planejado) e não algo que executou, diga isso.

## O que esperar

- Este é um projeto comunitário mantido no tempo livre — respostas podem demorar dias.
- Toda dúvida legítima recebe resposta respeitosa, independente do seu nível técnico (ver [docs/PHILOSOPHY.md](docs/PHILOSOPHY.md)).
- Issues que violem as regras do projeto (ex.: pedir telemetria, pedir mexer em Defender/Windows Update) serão explicadas e fechadas com a justificativa técnica.

## Quer contribuir em vez de só perguntar?

Melhor ainda — comece por [CONTRIBUTING.md](CONTRIBUTING.md). Documentação, tradução e revisão de textos contam tanto quanto código.
