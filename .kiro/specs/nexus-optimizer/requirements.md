# Spec: nexus-optimizer — Requisitos

**Estado: 📝 Planejado (FASE 2 ainda não iniciada)**

Esta spec transforma os requisitos de produto ([docs/REQUIREMENTS.md](../../../docs/REQUIREMENTS.md)) em formato de especificação para desenvolvimento. Escopo desta spec: **FASE 2 — Fundação técnica** (o projeto inteiro é planejado; a fase ativa é a fundação).

## Introdução

Fundação técnica do NEXUS OPTIMIZER: solução .NET 10 com shell WPF, contratos estáveis abstraídos, persistência local, logging e o pipeline de elevação auditado — sem funcionalidade de usuário final, mas com toda a espinha dorsal que as fases seguintes exigem.

## Glossário

- **Envelopamento de sensor**: toda leitura de hardware/sistema trafega em `SensorReading<T>` (`IsAvailable`, `Value`, `Source`, `Timestamp`, `UnavailableReason`).
- **Portão de mutação**: `IMutationExecutor`, caminho único de qualquer alteração no sistema (valida → consentimento → snapshot → aplica → auditoria).
- **Helper elevado**: `Nexus.Elevated.exe`, processo separado com whitelist de operações compilada, acessado via named pipe.
- **Máquina de referência**: ambiente de validação (Ryzen 5 5600X, 16 GB DDR4-3200, RTX 2060, Windows 11 build 26200 pt-BR).

## Requisitos da FASE 2

### Req 1 — Solução e projetos base

**User Story:** Como mantenedor do projeto, quero a solução .NET com todos os projetos e dependências corretas, para que as fases seguintes encontrem a arquitetura já fisicamente organizada.

#### Critérios de aceitação

1. WHEN a solução é criada THEN SHALL existir `Nexus.slnx` com os 15 projetos de docs/ARCHITECTURE.md §3 (App, Abstractions, Core, Application, Infrastructure, Hardware, Monitoring, Gaming, Development, Cleaner, Optimization, Security, Benchmark, Persistence, Elevated).
2. WHEN as referências são configuradas THEN módulos SHALL depender apenas de Abstractions/Core, e o acoplamento entre módulos SHALL passar por Application; quaisquer referências proibidas causam falha de build documentada (analyser ou test de arquitetura).
3. WHEN `dotnet build` é executado na máquina de referência (SDK 10.0.401) THEN SHALL compilar sem erros nem avisos tratados como bloqueantes.

### Req 2 — Contratos de abstração

**User Story:** Como desenvolvedor dos módulos, quero interfaces estáveis de sensores, mutação, auditoria e IPC, para implementar sem acoplar ao SO real.

#### Critérios de aceitação

1. WHEN Nexus.Abstractions é compilado THEN SHALL expor `ISensorSource`, `IMutationExecutor`, `IAuditSink`, `IIpcChannel` e o record `SensorReading<T>` exatamente conforme docs/ARCHITECTURE.md §4.
2. WHEN qualquer implementação retorna leitura indisponível THEN `IsAvailable` SHALL ser `false`, `Value` SHALL ser `default` (nunca placeholder), e `UnavailableReason` SHALL estar preenchido com motivo legível.

### Req 3 — Host, DI e logging

**User Story:** Como operador do app, quero logging diagnóstico local e configuração padronizada desde o primeiro build, para depurar campo sem telemetria externa.

#### Critérios de aceitação

1. WHEN o app inicia THEN Microsoft.Extensions.Hosting SHALL compor DI, configuração e Serilog (rolling file em `%LocalAppData%\Nexus\logs`, retenção configurável).
2. WHEN qualquer log é gravado THEN SHALL NOT conter credenciais, tokens, chaves ou dados pessoais além do operacional (RNF-07).
3. WHEN o app é executado THEN SHALL NOT realizar nenhuma chamada de rede externa (RNF-05) — validável com monitoramento de conexões.

### Req 4 — Persistência local

**User Story:** Como produto, quero banco SQLite migrável desde o início, para acumular diagnósticos, auditoria e benchmarks nas próximas fases.

#### Critérios de aceitação

1. WHEN o app roda pela primeira vez THEN SHALL criar o arquivo SQLite e aplicar migrações versionadas via `PRAGMA user_version`, sem intervenção manual.
2. WHEN o schema sobe THEN SHALL conter as tabelas de docs/ARCHITECTURE.md §6: `diagnostics`, `sensor_samples`, `mutations`, `restore_points`, `benchmark_runs`, `registry_backups`, `settings`.
3. WHEN uma migração falha THEN o app SHALL registrar erro completo no log e abortar inicialização limpa (sem banco parcial silencioso).

### Req 5 — Shell WPF de navegação

**User Story:** Como usuário, quero a casca do app com navegação entre módulos e tema escuro, para que cada funcionalidade futura tenha onde morar.

#### Critérios de aceitação

1. WHEN o app abre THEN SHALL executar **sem elevação**; se iniciado como administrador, SHALL exibir aviso recomendando execução sem privilégio.
2. WHEN a janela principal renderiza THEN SHALL apresentar navegação lateral Wpf.Ui com os 9 itens de docs/UI_UX.md §3 (Painel, Monitor, Gaming, Dev, Cleaner, Tweaks, Security, Bench, Config), cada um abrindo página placeholder honesta ("planejado — ver ROADMAP").
3. WHEN o app fica em repouso THEN uso de RAM SHOULD ficar abaixo de 300 MB (RNF-04).

### Req 6 — Portão de mutação em modo dry-run

**User Story:** Como arquiteto, quero o pipeline completo de mutação funcionando com uma operação de teste, para validar snapshot → auditoria → rollback antes de qualquer tweak real existir.

#### Critérios de aceitação

1. WHEN uma `MutationRequest` de teste é submetida THEN `IMutationExecutor` SHALL executar validação → snapshot → aplicação simulada (dry-run) → gravação na tabela `mutations` com estado antes/depois e método de rollback.
2. WHEN a operação é de nível ADVANCED/EXPERT THEN o fluxo SHALL exigir consentimento explícito registrado antes de prosseguir; negação SHALL abortar limpa e auditar `CANCELADO_USUARIO`.

### Req 7 — Helper elevado e IPC

**User Story:** Como requisito de segurança, quero o canal privilegiado isolado e com whitelist compilada, para que nenhuma operação arbitrária seja executável via IPC.

#### Critérios de aceitação

1. WHEN uma operação whitelisted de teste é solicitada THEN `Nexus.Elevated.exe` SHALL ser spawnado sob UAC (1 vez por sessão), executar e retornar resultado + evidências via named pipe com ACL do usuário atual.
2. WHEN um payload não whitelisted ou comando de shell livre chega ao pipe THEN o helper SHALL rejeitar e registrar tentativa.
3. WHEN o usuário nega o UAC THEN a operação SHALL abortar sem estado parcial, com auditoria; WHEN o helper fica ocioso além do timeout configurado THEN SHALL encerrar.

### Req 8 — Base de testes

**User Story:** Como mantenedor, quero testes de domínio rodando sem SO real, para refatorar com segurança desde o início.

#### Critérios de aceitação

1. WHEN `dotnet test` roda THEN SHALL executar Nexus.Core.Tests e Nexus.Application.Tests com infra falsa (ambiente injetado), sem tocar registro, disco real ou serviços.
2. WHEN código de domínio referencia API de SO diretamente THEN teste de arquitetura SHALL reprovar (regra de docs/ARCHITECTURE.md §8).
