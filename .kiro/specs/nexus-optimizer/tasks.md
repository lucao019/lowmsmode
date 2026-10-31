# Spec: nexus-optimizer — Tasks (FASE 2, Fundação)

**Estado: 📝 Planejado — nenhuma tarefa iniciada**

Plano de implementação da FASE 2 derivado de `requirements.md` + `design.md`. Cada tarefa referencia os requisitos que satisfaz. Ordem em que as dependências aparecem; executar sequencialmente salvo indicação.

## Tasks

- [ ] 1. Scaffold da solução e convenções de build
  - Gerar `Nexus.slnx` + 15 projetos de `docs/ARCHITECTURE.md §3` via `scripts/new-solution.ps1` idempotente (versionado em scripts/).
  - Configurar referências: App → Application; Application → Core/Abstractions/módulos; módulos → Abstractions/Core apenas; Infrastructure único com APIs de SO; Elevated isolado.
  - Definir `Directory.Build.props` (TFM net10.0, LangVersion 13, Nullable enable, ImplicitUsings, tratamento de warnings).
  - _Requirements: 1.1, 1.2, 1.3_

- [ ] 2. Contratos em Nexus.Abstractions
  - Implementar `SensorReading<T>`, `ISensorSource`, `IMutationExecutor`, `IAuditSink`, `IIpcChannel` + tipos `MutationRequest/MutationResult/Elevated*/AuditEntry` conforme design §2.
  - Garantir que `SensorReading<T>` indisponível carrega `UnavailableReason` (teste de contrato simples).
  - _Requirements: 2.1, 2.2_

- [ ] 3. Host, DI e Serilog em Nexus.App
  - `App.xaml.cs` compõe `IHost` (Microsoft.Extensions.Hosting): DI, config JSON, Serilog rolling file em `%LocalAppData%\Nexus\logs` com retenção.
  - Política de conteúdo de log (sem dados sensíveis) documentada no código (filtro/enrichment mínimo).
  - _Requirements: 3.1, 3.2, 3.3_

- [ ] 4. Persistência SQLite + Dapper
  - Connection factory, migrações ordenadas com `PRAGMA user_version` em transação (falha → abort limpo com log).
  - Tabelas de `docs/ARCHITECTURE.md §6`; repositório mínimo `settings` + gravação em `mutations` (dry-run) + `restore_points`.
  - _Requirements: 4.1, 4.2, 4.3_

- [ ] 5. IMutationExecutor (dry-run) + auditoria
  - Executor em Nexus.Security: validação de nível, consentimento obrigatório ADVANCED/EXPERT (callback para UI), snapshot simulado, gravação em `mutations`, `MutationResult` sem exceções de negócio.
  - Tweak de teste SAFE + um ADVANCED para exercitar consentimento e cancelamento (`CANCELADO_USUARIO`).
  - _Requirements: 6.1, 6.2_

- [ ] 6. Nexus.Elevated.exe + canal IPC
  - Console app do helper: named pipe `nexus.elevated.<sid>` com ACL do usuário, validação do processo cliente, whitelist compilada com `TEST_ECHO_ELEVATED`, rejeição logada de payload inválido.
  - Cliente `IIpcChannel` na Security/Infrastructure: spawn sob UAC, timeout de ociosidade, tratamento de UAC negado → `CancelledByUser`.
  - _Requirements: 7.1, 7.2, 7.3_

- [ ] 7. Shell WPF de navegação
  - `MainWindow` com `Wpf.Ui` NavigationView, tema escuro, 9 itens de navegação (docs/UI_UX.md §3) com páginas placeholder seladas "📝 Planejado".
  - Detecção de elevação no startup com aviso (Req 5.1); banners e toasts básicos.
  - _Requirements: 5.1, 5.2, 5.3_

- [ ] 8. Testes de domínio e arquitetura
  - `tests/Nexus.Core.Tests` e `tests/Nexus.Application.Tests`: casos de uso da FASE 2 com infra falsa (ambiente injetado).
  - Teste de arquitetura: dependências proibidas (SO fora de Infrastructure/Elevated; módulo→módulo; App→Elevated) falham com mensagem explícita.
  - _Requirements: 8.1, 8.2_

- [ ] 9. Validação de empacotamento precoce
  - `dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true` do shell da FASE 2; executável abre sem SDK instalado (validar em máquina/VM sem SDK).
  - Registrar resultado e tamanho em `docs/` (atualizar selos de status).
  - _Requirements: 5.3, RNF-08_

- [ ] 10. Critério de saída da FASE 2 (captura de evidência)
  - Checklist: app abre sem elevação ✔ · navega entre módulos vazios ✔ · loga em rolling file ✔ · persiste settings ✔ · pipeline mutação→helper→auditoria com operação de teste ✔ · `dotnet test` verde ✔.
  - Atualizar selos em `docs/ROADMAP.md` (FASE 2: 📝 → ✔️) e `REQUIREMENTS.md` aplicáveis.
  - _Requirements: todos da FASE 2_

## Notas de execução

- Nenhuma tarefa desta FASE implementa funcionalidade de usuário final (diagnóstico, monitoramento, tweaks reais) — isso começa na FASE 3.
- Regra permanente durante a execução: qualquer desvio do design aqui registrado deve atualizar `design.md` na mesma entrega.
- Commits convencionais por task concluída (ex.: `feat(core): add SensorReading contract`).
