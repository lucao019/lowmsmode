# PRODUCT VISION — NEXUS OPTIMIZER

**Estado: 📝 Planejado**

## 1. Visão

Ser a plataforma de referência para quem leva desempenho de Windows a sério: um painel único, honesto e auditável que **diagnostica**, **monitora**, **otimiza** e **mede** — sem mágica, sem promessa vazia, sem risco cego.

O NEXUS não é mais um "PC cleaner". É uma ferramenta de **engenharia de desempenho** com dois perfis de usuário de alto exigência:

- **PERFIL A — Gaming competitivo**: jogador de PUBG que não aceita perder 1 frame. Prioridade absoluta: anti-stutter, frametime estável, baixa latência de entrada.
- **PERFIL B — Desenvolvimento/IA**: dev que trabalha com Kiro, VS Code, Python, Node, Git, Ollama (Qwen3) e Docker no mesmo Windows, sem sacrificar estabilidade.

## 2. Problema que resolve

1. **Otimizadores genéricos mentem.** Exibem "ganhos" inventados, aplicam tweaks sem justificativa e quebram sistemas sem caminho de volta.
2. **Otimizar Windows de verdade exige método.** Diagnóstico → hipótese técnica → aplicação reversível → medição em condições controladas → decidir. Hoje isso é manual e artesanal.
3. **Dados de telemetria falham e ninguém avisa.** Contadores PDH/WMI dependem de idioma do sistema (pt-BR) e de arquivos de perfil que podem estar corrompidos/ausentes — apps comuns simplesmente exibem valores falsos ou travados.
4. **Admin é um tudo-ou-nada perigoso.** Ferramentas que exigem elevação total para tudo ampliam a superfície de dano.

## 3. Origem (caso real)

O projeto é derivado de uma sessão completa de engenharia executada manualmente em uma máquina de produção (2026-10-31), com resultados verificados:

| Ação manual | Resultado medido |
|---|---|
| Limpeza seletiva de disco (DXCache NVIDIA, temps, DISM StartComponentCleanup, hibernação off, TRIM) | C: de 0,3 GB → **51,3 GB livres** |
| Remoção de 15 entradas órfãs de uninstall (3 hives) com backup .reg individual por chave | 15/15 removidas, 0 falsos positivos |
| Otimizações de gaming (plano Alto Desempenho, Game DVR off, HVCI off, NIC power saving off, Storage Sense) | Aplicadas com **script completo de reversão** |
| Diagnóstico de contadores de desempenho corrompidos (arquivos pt-BR ausentes) | Identificada causa raiz e caminho de reparo |

Cada lição virou regra de produto (ver [DECISIONS.md](DECISIONS.md)), incluindo:

- Falta de nome de contador localizado = **sensor indisponível**, não número inventado.
- Nenhuma deleção de registro sem export `.reg` por chave antes.
- "Pasta ausente" não prova desinstalação — critérios múltiplos.
- Condições de medição devem ser **idênticas** no antes/depois (PC em repouso, mesma carga).

## 4. Objetivos do produto

### 4.1 Objetivos primários

1. Centralizar diagnóstico, monitoramento, otimização, limpeza e benchmark do Windows 11 em um app único e profissional.
2. Garantir que **100% das mutações** sejam: justificadas, medidas, auditadas e reversíveis.
3. Entregar perfis prontos (Gaming competitivo / Dev-IA) baseados em base técnica, não em placebo.

### 4.2 Objetivos secundários

4. Educar o usuário: cada tweak explicado em linguagem clara (pt-BR), com o "porquê" técnico.
5. Criar base de conhecimento persistente (SQLite) de diagnósticos, medições e auditoria.
6. Servir como portfólio de engenharia: arquitetura limpa, código testável, documentação completa.

## 5. Público-alvo

- Jogadores competitivos (perfil anti-stutter).
- Desenvolvedores Windows que rodam stacks locais pesadas (IDE + IA local + Docker).
- Entusiastas de hardware que querem dados reais, não marketing.

## 6. Proposta de valor

> **"Otimização de Windows com rigor de engenharia: mede de verdade, muda com segurança, reverte sempre."**

Diferenciais:

- Honestidade de dados como regra de arquitetura (`IsAvailable` em todo sensor).
- Reversibilidade total com auditoria (portão `IMutationExecutor`).
- Privilégio mínimo: app sem elevação; helper `Nexus.Elevated.exe` com whitelist.
- Perfis orientados a casos de uso reais e verificados.

## 7. Escopo

### Dentro do escopo

- Windows 11 (referência: build 26200).
- Hardware desktop comum (CPU AMD/Intel, GPU NVIDIA primeiro; AMD GPU depois).
- Otimizações de OS seguras e reversíveis; limpeza seletiva; monitoramento; benchmarking.

### Fora do escopo (explícito)

- Defender, Windows Update, Firewall — não configurar, não desativar.
- Anti-cheat de jogos — não inspecionar, não burlar, não tocar.
- BIOS, overclock, undervolt — domínio do usuário; o app no máximo lê.
- Drivers e firmware — o app pode *detectar/obsoleto*; instalação é decisão do usuário fora do app (fase inicial).
- Windows 10 e anteriores (fase inicial).

## 8. Métricas de sucesso do produto

| Métrica | Alvo |
|---|---|
| Mutações sem rollback disponível | **0** |
| Valores de sensor falsificados/estáticos exibidos como reais | **0** |
| Reversões completas testadas em máquina real | 100% dos tweaks SAFE |
| Diagnosticar com SDK .NET sem admin | Sempre possível |
