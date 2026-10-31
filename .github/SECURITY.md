# Política de Segurança — Reporte de Vulnerabilidades

> Este arquivo trata de **como reportar vulnerabilidades no projeto**. A segurança do produto (modelo de privilégios, auditoria, lista bloqueada) está documentada em [docs/SECURITY.md](../docs/SECURITY.md).

## Reportando uma vulnerabilidade

**Não abra issue pública para vulnerabilidades.** Uma issue pública expõe o problema antes que exista correção e coloca outros usuários em risco.

Use um destes canais privados:

1. **GitHub Security Advisories** (preferido): aba **Security → Advisories → Report a vulnerability** neste repositório. O relatório fica privado e visível apenas aos mantenedores.
2. **E-mail do mantenedor**: disponível no perfil [@lucao019](https://github.com/lucao019) — use o assunto `[NEXUS] Vulnerabilidade`.

## O que incluir no relatório

- Descrição da vulnerabilidade e do impacto potencial.
- Passos para reproduzir (ou prova de conceito).
- Versão/commit afetado e ambiente (Windows, .NET).
- Se possível, sugestão de mitigação.

Prova de conceito deve ser o **mínimo necessário** para demonstrar — nunca desastrada ou destrutiva.

## O que esperar

| Etapa | Compromisso |
|---|---|
| Confirmação de recebimento | Até 7 dias corridos |
| Avaliação inicial (válido? severidade?) | Até 14 dias corridos |
| Comunicação do plano de correção | Assim que avaliada |
| Divulgação coordenada | Após correção disponível, com crédito ao relator (se desejar) |

Este é um projeto comunitário mantido no tempo livre — os prazos são compromissos de boa-fé, não SLAs comerciais.

## Escopo

- **Em escopo**: o código deste repositório (quando existir), o helper elevado `Nexus.Elevated.exe`, o canal IPC, a persistência e o instalador.
- **Relevante já na fase de documentação**: falhas de design documentadas (ex.: uma regra de segurança do [docs/SECURITY.md](../docs/SECURITY.md) que possa ser contornada) também podem ser reportadas — corrigir o design antes do código é mais barato.
- **Fora de escopo**: vulnerabilidades em dependências de terceiros (reporte ao projeto da dependência) e problemas gerais do Windows.

## Regras para pesquisadores

- Não acesse, modifique ou exfiltre dados de terceiros.
- Não execute testes destrutivos (DoS, deleção) em sistemas que não sejam seus.
- Não divulgue publicamente antes de acordarmos um prazo.

Seguindo estas regras, o pesquisador tem nosso compromisso de boa-fé e reconhecimento público do achado (se desejado) na seção de colaboradores do README.
