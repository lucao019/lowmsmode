---
inclusion: always
---

# Stack técnica — NEXUS OPTIMIZER

## Plataforma

- **.NET 10 LTS** (C# 13), publicação single-file self-contained `win-x64` (SDK de referência: 10.0.401).
- **UI: WPF + Wpf.Ui** (Fluent). MVVM com **CommunityToolkit.Mvvm** (source generators). Gráficos: **LiveCharts2**.
- Host/DI: **Microsoft.Extensions.Hosting**. Logging: **Serilog** rolling file em `%LocalAppData%\Nexus\logs`, retenção configurável.
- Persistência: **SQLite** (Microsoft.Data.Sqlite) + **Dapper**; migrações em código via `PRAGMA user_version`.

## Hardware/telemetria

- GPU NVIDIA: **NvAPIWrapper** (user-mode, sem admin).
- Temperaturas: **LibreHardwareMonitorLib** como **plugin opcional isolado** (exige admin); ausente → "indisponível", resto funciona.
- Contadores PDH: descoberta **independente de idioma** (enumerar via ListSet/padrões, nunca strings fixas; fallback WMI). Nome localizado ausente = sensor indisponível, não número inventado. Máquina-alvo pt-BR — campo de prova real.

## Elevação

- `Nexus.App` sem elevação. `Nexus.Elevated.exe` separado: named pipe com ACL, whitelist de operações **compilada no binário**, payload tipado (nunca shell livre), 1 UAC por sessão, helper encerra ocioso.

## Convenções

- UI e logs em **pt-BR**; termos técnicos podem ficar em inglês (ex.: frametime, VBS, HVCI).
- Telemetria nunca bloqueia a UI (coleta em background, cache com timestamp). RAM do app alvo < 300 MB em repouso.
- **Zero rede externa** no app: sem telemetria de saída, sem analytics.
- Instalador: **Inno Setup** (`installer/nexus.iss`).

## Comandos (quando o código existir)

```bash
dotnet build
dotnet test
dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true
```
