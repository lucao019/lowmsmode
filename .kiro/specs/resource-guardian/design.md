# Spec: resource-guardian — Design

**Estado: 📝 Planejado.** Normativo: `docs/RESOURCE_GUARDIAN.md`. Reutiliza contratos da FASE 2 (`SensorReading<T>`, `IMutationExecutor`) — não duplicar.

## Projetos (7 novos, ARCHITECTURE.md §3)

| Projeto | Depende de | Conteúdo-chave |
|---|---|---|
| Nexus.ProcessScanner | Abstractions, Infrastructure | Snapshot de processos: Toolhelp32, WinVerifyTrust, PDH por processo, correlação PID↔serviço |
| Nexus.ApplicationInventory | Abstractions, Infrastructure | Uninstall hives, PackageManager, SCM, Run/StartupApproved, Task Scheduler COM, diff periódico |
| Nexus.AIIntegration | Abstractions | `ILocalAIProvider` + providers HTTP/np loopback (Ollama primeiro) |
| Nexus.ResourceGuardian | Abstractions, Core | Classificação de impacto, motor de regras, recomendações, perfis, orquestração das 4 operações |
| Nexus.GamingPreflight | Abstractions, Application | Fluxo 7 etapas, Game Mode automático, restauração pós-jogo |
| Nexus.ResourceMonitor | Abstractions | Amostragem adaptativa em sessão delegando a Nexus.Monitoring |
| Nexus.RestartManager | Abstractions | Restart controlado app/serviço; recomendação excepcional de reboot |

## Contratos centrais

```csharp
public enum ImpactLevel { Critical, High, Medium, Low, Essential }

public sealed record ProcessInfo(
    int Pid, string Name, string? ExePath, string? Signer,
    int? ParentPid, SensorReading<double> CpuPercent,
    SensorReading<long> WorkingSetBytes, SensorReading<long> CommitBytes,
    SensorReading<long> IoBytesPerSec, SensorReading<double> GpuPercent,
    IReadOnlyList<string> HostingServices);

public sealed record ClassifiedProcess(
    ProcessInfo Info, ImpactLevel Impact, string Rationale); // "por quê" obrigatório

public interface ILocalAIProvider
{
    string ProviderId { get; }
    Task<DetectionResult> DetectAsync(CancellationToken ct);
    Task<RuntimeState> GetStateAsync(CancellationToken ct);
    Task<SensorReading<UnloadResult>> UnloadAsync(ModelRef model, CancellationToken ct);
    Task<bool> CanPauseInferenceAsync(CancellationToken ct);
}

public sealed record Recommendation(
    string TargetId, LifecycleAction Action, string Reason,
    string ResourcesInUse, string ExpectedBenefit, string Risk,
    bool HasUnsavedWork, bool IsReversible, string RestoreHow);

public enum LifecycleAction { Pause, Unload, GracefulShutdown, Restart }

public interface IRuleEngine
{
    IReadOnlyList<Recommendation> Evaluate(
        SystemSnapshot snapshot, ResourceProfile profile);
}
```

## Decisões de design

1. **4 operações = MutationRequests** tipadas (`PAUSE_PROCESS`, `UNLOAD_MODEL`, `STOP_APP`, `RESTART_SERVICE`) adicionadas à whitelist do `Nexus.Elevated` quando necessitarem elevação — snapshot guarda estado (modelos carregados, serviços parados) para restauração.
2. **Kill** existe apenas como ferramenta EXPERT (`FORCE_TERMINATE`), desabilitada por padrão, confirmação + justificativa, nunca para ESSENTIAL/protegidos.
3. **Lista de proteção** é avaliação composta (caminho + assinatura + pai + serviço) compilada em regras do Core, sobreposta à lista bloqueada global (SECURITY §2.1) — não é configurável por perfil.
4. **Descoberta de jogo**: ciclo de amostragem detecta processo do jogo → transição de estado `Idle→Preflight→InGame→PostGame` na máquina de estados do Guardian; anti-cheat detectado = modo somente-observação reforçado.
5. **Amostragem adaptativa**: `ISamplingPolicy` (1 s normal → 2–5 s em jogo fullscreen exclusivo, pausa de render de gráficos em 1º plano); ResourceMonitor delega coleta a `Nexus.Monitoring`, só muda ritmo/agregação.
6. **Re-inventário**: snapshot das hives + Startup a cada intervalo configurável; diff produz eventos `Installed/Removed/Changed` (histórico local).
7. **Loopback apenas**: providers falam 127.0.0.1/named pipes; regra de teste de arquitetura proíbe qualquer outro endpoint de rede no módulo.
8. **VRAM por processo**: não estimar. UI fixa "indisponível" no campo, com a única fonte válida sendo VRAM agregada da GPU (NvAPI).

## Persistência (novas tabelas)

`app_inventory` (diffs), `guardian_actions` (auditoria das 4 operações + resultados de verificação), `resource_profiles` (5 perfis, editáveis), `game_sessions` (pre-flight, in-game events, restaurações).

## Riscos

| Risco | Mitigação |
|---|---|
| API do provider muda entre versões | capability negotiation; "versão não suportada" honesta |
| Anti-cheat sensível a instrumentação | lista de proteção; nenhuma leitura invasiva de memória do jogo; somente métricas de processo |
| Falso ESSENTIAL por spoof de nome | validação composta de identidade |
| Overhead em jogo | política de amostragem adaptativa + benchmark do próprio Guardian (aceitação) |

## Testes (tests/Nexus.ResourceGuardian.Tests + providers)

Domínio puro (classificação, regras, proteção) com snapshots falsos; providers com servidores loopback falsos; 10 cenários normativos de docs/RESOURCE_GUARDIAN.md §12 como casos de teste nomeados `Cenario01`…`Cenario10`.
