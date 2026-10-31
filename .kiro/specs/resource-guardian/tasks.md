# Spec: resource-guardian — Tasks (FASE 6B)

**Estado: 📝 Planejado — nenhuma tarefa iniciada. Pré-requisito: FASES 2–6 concluídas (portão, auditoria, monitoramento, tweaks de gaming).**

## Tasks

- [ ] 1. ApplicationInventory — fontes de leitura e diff de re-inventário
  - Uninstall (3 hives), SCM, Run/StartupApproved/pastas, Task Scheduler COM; Store via PackageManager com fallback honesto; eventos `Installed/Removed/Changed`.
  - Tabela `app_inventory` + migração.
  - _Requirements: 1.1–1.5_

- [ ] 2. ProcessScanner — snapshot de processos
  - Toolhelp32 + WinVerifyTrust + PDH por processo (CPU/RAM/commit/I/O/GPU Engine) com `SensorReading<T>` independente de idioma; correlação PID↔serviço; VRAM por processo = "indisponível" fixo.
  - _Requirements: 3.1–3.5_

- [ ] 3. AIIntegration — núcleo + provider Ollama
  - `ILocalAIProvider`; provider Ollama (`/api/ps`, `/api/tags`, unload oficial); teste com servidor loopback falso (Cenários 1, 2, 4); capability de versão.
  - _Requirements: 2.1–2.6_

- [ ] 4. ResourceGuardian — classificação e motor de regras
  - `ImpactLevel` + `Rationale` obrigatório; regras relativas ao hardware/perfil; lista de proteção composta (Cenário 8); perfis GAMING/DEVELOPMENT/LOCAL_AI/BALANCED/CUSTOM em `resource_profiles`.
  - _Requirements: 7.1–7.3_

- [ ] 5. Operações de ciclo de vida via portão
  - `PAUSE_PROCESS`/`UNLOAD_MODEL`/`STOP_APP`/`RESTART_SERVICE` + snapshots restauráveis em `guardian_actions`; bloqueio por trabalho não salvo (Cenário 7); kill EXPERT opt-in (Cenários 6, 8).
  - RestartManager: restart de app/serviço + recomendação excepcional de reboot (Cenário 10).
  - _Requirements: 6.1–6.6_

- [ ] 6. GamingPreflight — fluxo das 7 etapas
  - UI step-by-step, consentimento item a item, verificação final com segunda varredura; declaração de limitação quando PDH indisponível (Cenário 3).
  - _Requirements: 4.1–4.6_

- [ ] 7. Game Mode + ResourceMonitor adaptativo
  - Máquina de estados `Idle→Preflight→InGame→PostGame`; amostragem 2–5 s em sessão; notificação discreta de novo processo pesado (Cenário 5); restauração opt-in pós-jogo (Cenário 9).
  - _Requirements: 5.1–5.5_

- [ ] 8. UI Guardian (10 subseções)
  - Seção própria na navegação; tabela ordenável/filtrável com impacto e "por quê"; histórico; sem dados simulados.
  - _Requirements: 3.4, 7.2_

- [ ] 9. Benchmark do próprio módulo
  - Overhead do loop de monitoramento medido em sessão de jogo na máquina de referência; evidência registrada.
  - _Requirements: 5.1_

- [ ] 10. Critério de saída da FASE 6B
  - 10 cenários de docs/RESOURCE_GUARDIAN.md §12 verdes como testes nomeados; critérios de aceitação §13 todos ✔️ na máquina de referência; selos atualizados (ROADMAP/REQUIREMENTS).
  - _Requirements: todos_

## Notas

- Nenhuma implementação antes da FASE 6 estar validada (regra ROADMAP: diagnóstico/medição precedem ação).
- Desvios deste design exigem atualização do `design.md` na mesma entrega.
