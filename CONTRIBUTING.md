# Contribuindo com o NEXUS OPTIMIZER

Obrigado por considerar contribuir! Este projeto existe para **compartilhar conhecimento** — e isso inclui dar boas-vindas a quem está começando. Você não precisa ser expert para contribuir: documentação, tradução, relatos de bug bem descritos e testes valem tanto quanto código.

Leia antes: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) e [docs/PHILOSOPHY.md](docs/PHILOSOPHY.md).

## Formas de contribuir (por nível)

| Nível | Exemplos |
|---|---|
| Iniciante / sem código | Reportar bugs com passos de reprodução, melhorar explicações dos docs, traduzir, revisar textos, sugerir ideias |
| Intermediário | Testes unitários, pequenas correções, melhorias de UI/UX, novos contadores de diagnóstico, compatibilidade de hardware |
| Avançado | Módulos novos, IPC/elevação, pipeline de mutação, benchmark, estatística |

Categorias organizadas por label nas issues: `bug`, `performance`, `hardware-compatibility`, `enhancement`, `documentation`, `ui-ux`, `translation`, `tests`.

## Preparando o ambiente

> O código ainda não existe (FASE 2 planejada). Estas instruções valem a partir do primeiro commit de código.

1. Pré-requisitos: Windows 11, **.NET 10 SDK**, Git. (IDE: Visual Studio 2026+ ou VS Code com C# Dev Kit.)
2. Fork o repositório → clone o seu fork.
3. `dotnet restore && dotnet build` na raiz.
4. `dotnet test` para garantir base verde.
5. Crie uma branch: `git checkout -b tipo/descricao-curta` (ex.: `fix/pdh-locale-fallback`, `docs/contributing-windows`).

## Regras do projeto (não negociáveis)

Estas vêm do produto, não de preferência — PRs que as violarem serão rejeitados:

1. **Nunca exibir métrica falsa** — sensor indisponível mostra "indisponível" com motivo (ver `SensorReading<T>` em docs/ARCHITECTURE.md §4).
2. **Toda mutação via `IMutationExecutor`** com snapshot, auditoria e rollback — nenhum módulo chama o SO diretamente para alterar o sistema.
3. **Lista bloqueada**: nada de Defender, Windows Update, Firewall, anti-cheat, BIOS/OC (docs/SECURITY.md §2.1).
4. **Sem telemetria de saída** — nenhuma chamada de rede no app principal.
5. Descoberta de contadores PDH **independente de idioma** (RNF-10).
6. Lógica de domínio testável sem SO real (docs/ARCHITECTURE.md §8).
7. Commits em estilo convencional (`feat:`, `fix:`, `docs:`, `test:`, `chore:`...).

## Enviando a contribuição

1. Abra/atualize uma issue descrevendo o problema ou a proposta (templates em `.github/ISSUE_TEMPLATE/`).
2. Referencie a issue no PR (`Closes #123`).
3. PR pequeno e focado > PR gigante. Descreva: **o quê**, **por quê**, **como testar**, riscos e rollback.
4. Inclua testes quando alterar comportamento; atualize docs quando alterar decisões ou status.
5. Não marque nada como implementado/testado/validado sem evidência (selos de docs/REQUIREMENTS.md).

## Reportando vulnerabilidades de segurança

**Não abra issue pública.** Siga [.github/SECURITY.md](.github/SECURITY.md).

## Reconhecimento

Toda contribuição relevante (código, docs, testes, tradução, relatos, sugestões) é creditada na seção de colaboradores do README.
