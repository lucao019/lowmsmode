# UI/UX — NEXUS OPTIMIZER

**Estado: 📝 Planejado**

Identidade visual e experiência do usuário. Framework: WPF + [`Wpf.Ui`](https://github.com/lepoco/wpfui) (Fluent), navegação lateral estilo Windows 11 Settings.

## 1. Princípios de experiência

1. **Honestidade visual** — reflexo direto da arquitetura: dado indisponível aparece como "indisponível" (com motivo), jamais como zero ou placeholder animado.
2. **Sem dark patterns** — nenhum botão de "otimizar tudo" sem listar exatamente o que muda; nenhum consentimento escondido; nenhum ícone de "alerta vermelho" para métricas normais.
3. **Perfil-útil, não enfeitado** — o usuário alvo (gaming competitivo / dev pesado) quer densidade de informação real, não animações decorativas.
4. **Consistência com Windows 11** — cantos arredondados, Mica/Acrylic onde o Wpf.Ui oferece, tipografia Segoe UI Variable.

## 2. Identidade visual

| Elemento | Decisão |
|---|---|
| Nome exibido | NEXUS (wordmark em caixa alta, tracking aberto) |
| Tema padrão | Escuro (padrão para gaming/dev); claro disponível |
| Cor de acento | Ciano/azul elétrico (hardware/telemetria); reutiliza `SystemAccentColor` quando o usuário preferir |
| Semáforo | Verde = saudável · Âmbar = atenção recomendada · Vermelho = ação necessária · Cinza = indisponível |
| Ícones | Fluent System Icons (via Wpf.Ui), traço regular |
| Idioma | pt-BR; termos técnicos em inglês (RNF-03, ADR-013) |

## 3. Navegação e módulos

```text
┌────────────────────────────────────────────────┐
│ NEXUS                              [_][□][X]   │
├──────────┬─────────────────────────────────────┤
│ Painel   │                                     │
│ Monitor  │         Área do módulo              │
│ Gaming   │    (cards, gráficos, tabelas)       │
│ Guardian │                                     │
│ Dev      │                                     │
│ Cleaner  │                                     │
│ Tweaks   │                                     │
│ Security │                                     │
│ Bench    │                                     │
│ ──────── │                                     │
│ Config   │                                     │
└──────────┴─────────────────────────────────────┘
```

| Item de navegação | Conteúdo |
|---|---|
| **Painel** | Saúde geral: alertas reais, integridade de telemetria, ações recomendadas (só após diagnóstico) |
| **Monitor** | CPU/GPU/RAM/disco/rede/temperaturas com gráficos LiveCharts2 em tempo real |
| **Gaming** | Perfil PUBG competitivo, checklists guiados, estado dos tweaks do perfil |
| **Guardian** | Resource Guardian: processos por impacto, detecção de IA local, Pre-Flight, recomendações e histórico (docs/RESOURCE_GUARDIAN.md §9) |
| **Dev** | Ferramentas detectadas, limites sugeridos para modelos locais (Ollama/Qwen3), tweaks de ambiente |
| **Cleaner** | Categorias com dry-run, tamanho estimado, classificação de risco |
| **Tweaks** | Catálogo SAFE/ADVANCED/EXPERT com filtros e busca |
| **Security** | Pontos de restauração, linha do tempo de mutações, rollback por item, "reverter tudo" |
| **Bench** | Baselines, comparações antes/depois, relatório de significância |
| **Config** | Intervalo de amostragem, retenção de logs/séries, tema, plugin LHM |

## 4. Padrões de componentes

### 4.1 Cartão de sensor

```text
┌─────────────────────────┐
│ CPU                PDH  │  ← fonte da métrica sempre visível
│      37,2 %             │
│ └ 12 threads · 4,4 GHz  │
└─────────────────────────┘
```

- Fonte (`PDH`, `WMI`, `NvAPI`, `LHM`) exibida no canto — transparência de origem.
- Indisponível: valor substituído por **"indisponível"** + tooltip com `UnavailableReason` + link "ver diagnóstico" quando aplicável.
- Nunca mostrar `--` como se fosse zero; nunca congelar último valor sem indicar idade (`há 4 s`).

### 4.2 Item de tweak (catálogo)

```text
┌──────────────────────────────────────────────────────┐
│ ● SAFE   Desativar Game DVR                          │
│ Remove a captura em background da Xbox Game Bar.     │
│ Por quê: reduz overhead de GPU/memória em jogos.     │
│ Risco: nenhum conhecido · Rollback: 1 clique         │
│ Pré-requisito: nenhum           [Aplicar] [Detalhes] │
└──────────────────────────────────────────────────────┘
```

- Nível de risco sempre com cor e selo; ADVANCED/EXPERT abrem diálogo de consentimento com texto claro do efeito e do rollback.
- Estado do tweak (aplicado/pendente/não aplicável) visível sem clicar.

### 4.3 Consentimento (ADVANCED/EXPERT)

Diálogo modal com: o que muda (lista), por que (justificativa técnica), risco, como reverter, e botões **[Aplicar]** / **[Cancelar]**. EXPERT adiciona confirmação reforçada. Nenhum consentimento vale para o futuro — cada lote pede o seu.

### 4.4 Gráficos em tempo real

- LiveCharts2 com atualização no dispatcher apenas de agregados; amostragem em background thread (RF-MON-04 — gráfico nunca congela UI).
- Séries pausam quando a janela está minimizada (economia real de CPU).
- Alvo de RAM do app < 300 MB em repouso (RNF-04).

## 5. Estados vazios e de erro

| Situação | Comportamento |
|---|---|
| Primeira execução | Painel propõe **Diagnóstico inicial**; nenhuma sugestão antes de medir |
| Sensor indisponível | "indisponível" + motivo + link para diagnóstico/reparo |
| Contadores corrompidos | Banner no Painel: "Telemetria do Windows danificada — ver como reparar" (aceitação global 2) |
| UAC negado | Toast: operação cancelada, nada foi alterado; auditoria registra |
| Sem permissão p/ temperatura | Card mostra "requer plugin de temperatura (admin)" — sem insistência |

## 6. Acessibilidade & qualidade

- Contraste AA no tema escuro e claro; semáforo nunca apenas por cor (ícone + texto).
- Zoom de UI respeitado (DPI scaling do Windows 11).
- Tooltips explicando termos técnicos em uma frase.
- Nenhuma ação destrutiva acionável por teclado acidental (confirmação sempre via clique deliberado ou diálogo).

## 7. Referências

- [Wpf.Ui — documentação](https://wpfui.lepo.co/)
- [Fluent Design System](https://fluent2.microsoft.design/)
- [LiveCharts2](https://livecharts.dev/)
