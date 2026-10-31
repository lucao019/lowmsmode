# Spec: resource-guardian — Requirements

**Estado: 📝 Planejado — dependente da FASE 2 (fundação). Documento normativo: docs/RESOURCE_GUARDIAN.md.**

Sequência obrigatória de toda operação: IDENTIFICAR → ENTENDER → RECOMENDAR → AUTORIZAR → EXECUTAR → VERIFICAR → RESTAURAR.

## Requirement 1 — Inventário de aplicativos

**User Story:** Como usuário, quero que o app saiba o que está instalado, o que está aberto e o que está consumindo, para entender meu PC sem varrer meus arquivos pessoais.

### Acceptance Criteria
1. WHEN executada a varredura de inventário THEN SHALL listar Win32 (3 hives Uninstall), Store (PackageManager; indisponível → honesto "parcial"), serviços (SCM), inicialização (Run/StartupApproved/pastas), tarefas agendadas não-Microsoft e processos ativos.
2. WHEN um pacote gerenciável existe THEN SHALL enriquecer o inventário via consulta de leitura ao winget, quando presente.
3. WHILE qualquer inventário é produzido SHALL NOT varrer indiscriminadamente arquivos pessoais do usuário.
4. WHEN o inventário muda (nova instalação) THEN SHALL ser detectado por diff de re-inventário periódico, sem watchers de filesystem.
5. WHEN um item é exibido THEN SHALL diferenciar explicitamente INSTALADO × EM_EXECUÇÃO × CONSUMINDO.

## Requirement 2 — Detecção de IA local

**User Story:** Como usuário de IA local (Ollama, LM Studio, ComfyUI...), quero que o app reconheça meus ambientes e o que está carregado, para jogar sem conflito de RAM/VRAM.

### Acceptance Criteria
1. WHEN um ambiente de IA suportado existe THEN SHALL ser detectado por provider específico implementando `ILocalAIProvider`, sem heurística por nome.
2. WHEN o Ollama está acessível THEN SHALL consultar a API local oficial (`/api/ps` modelos carregados; `/api/tags` armazenados).
3. WHEN arquivos de modelo existem no disco mas nenhum modelo está carregado THEN SHALL NOT afirmar uso de RAM/VRAM de IA.
4. IF uma inferência está em andamento THEN SHALL NOT descarregar/pausar sem verificação de estado e autorização explícita.
5. WHEN a versão do ambiente de IA não é reconhecida THEN a integração SHALL reportar "indisponível: versão não suportada" em vez de tentar cegamente.
6. WHEN Docker/WSL2 estão presentes THEN SHALL consultar Engine API (named pipe local) e `wsl --list` apenas leitura, respectivamente.

## Requirement 3 — Process Scanner

**User Story:** Como usuário, quero ver todos os processos com seu impacto real, para saber o que pesa.

### Acceptance Criteria
1. WHEN a varredura roda THEN SHALL coletar nome, PID, caminho, assinatura/fabricante, processo pai, CPU, RAM, commit, I/O, GPU (PDH `GPU Engine`), estado, serviços relacionados e app responsável.
2. WHEN VRAM por processo for solicitada THEN SHALL exibir "indisponível" (inexistência de API documentada) — nunca estimativa; VRAM total via NvAPI.
3. WHEN métricas PDH por processo são lidas THEN SHALL usar descoberta independente de idioma (RNF-10) com envelope `SensorReading<T>`.
4. WHEN classificado THEN SHALL usar métricas observadas + hardware + perfil ativo (CRITICAL/HIGH/MEDIUM/LOW/ESSENTIAL), nunca apenas o nome, e SHALL carregar explicação da classificação.
5. WHEN um processo protegido do SO não permite leitura detalhada THEN SHALL ser listado com essa indicação, sem dados inventados.

## Requirement 4 — Gaming Pre-Flight (7 etapas)

**User Story:** Como jogador, quero uma verificação antes de abrir o PUBG que me explique conflitos e só mexa no que eu autorizar.

### Acceptance Criteria
1. WHEN o perfil PUBG Competitive é acionado THEN SHALL executar: hardware → aplicações → análise → recomendações priorizadas → autorização → execução → verificação final, nesta ordem.
2. WHEN recomendações são apresentadas THEN SHALL exibir por item: processo, motivo, recursos em uso, benefício esperado, risco, trabalho não salvo (sim/não), reversibilidade e como restaurar; aprovação SHALL ser item a item.
3. WHEN executando THEN SHALL aplicar apenas itens aprovados, priorizando APIs próprias do app e encerramento normal — kill SHALL estar desabilitado por padrão.
4. WHEN concluída a execução THEN SHALL realizar segunda varredura confirmando liberação real de recursos e estado de dependências, apresentando resultado honesto.
5. IF telemetria PDH estiver indisponível THEN o Pre-Flight SHALL declarar recomendações limitadas e o motivo, sem improvisar valores.
6. IF Windows Update instalando ou anti-cheat inconsistente THEN SHALL avisar e SHALL NOT prosseguir com ações arriscadas.

## Requirement 5 — Monitoramento em sessão e Game Mode

**User Story:** Como jogador, quero vigilância leve durante a partida e restauração oferecida depois dela.

### Acceptance Criteria
1. WHILE o jogo está ativo SHALL monitorar com amostragem adaptativa (padrão 2–5 s, configurável) e impacto mínimo medido; novo processo pesado SHALL gerar notificação discreta — nunca janela intrusiva/overlay sobre o jogo.
2. WHILE o jogo está ativo SHALL NOT executar mutações automáticas, exceto ações SAFE pré-autorizadas explicitamente; SHALL NOT interromper o jogo nem tocar anti-cheat.
3. WHEN o PUBG fecha THEN SHALL re-verificar, resumir o que foi pausado/encerrado e oferecer restauração opt-in (modelos, serviços, apps).
4. WHEN o usuário optou por não restaurar um item THEN SHALL NOT reiniciá-lo automaticamente.
5. WHEN Game Mode automático está habilitado THEN a detecção do processo do jogo SHALL disparar Pre-Flight resumido conforme configuração.

## Requirement 6 — Operações de ciclo de vida e proteção

**User Story:** Como usuário, quero pausar/descarregar/encerrar/reiniciar apps com segurança — e certeza de que o essencial está intocável.

### Acceptance Criteria
1. WHEN PAUSE/UNLOAD/GRACEFUL SHUTDOWN/RESTART são executadas SHALL passar pelo `IMutationExecutor` com snapshot restaurável e auditoria.
2. WHEN GRACEFUL SHUTDOWN encontra trabalho não salvo SHALL bloquear a operação e SHALL NOT forçar encerramento.
3. WHEN RESTART é autorizado SHALL registrar estado anterior, verificar dependências e confirmar retorno saudável — SHALL NOT reiniciar o computador.
4. IF o alvo consta na lista de proteção (Windows essencial, drivers, segurança, áudio, rede, shell/DWM, anti-cheat, jogo em execução, identidade não verificada) THEN a operação SHALL ser bloqueada com motivo exibido.
5. WHEN identidade é avaliada SHALL validar caminho + assinatura + processo pai + serviço associado — SHALL NOT confiar só no nome do executável.
6. IF reboot do Windows for julgado necessário THEN SHALL apenas recomendar com justificativa técnica e aguardar decisão do usuário; SHALL NOT reiniciar automaticamente em nenhuma circunstância.

## Requirement 7 — Motor de decisão, perfis e privacidade

### Acceptance Criteria
1. WHEN recomendando SHALL usar regras determinísticas sobre hardware detectado, disponibilidade, consumo, tipo de jogo, dependências e perfil — SHALL NOT usar limiar universal de RAM/VRAM.
2. WHEN uma conclusão é exibida SHALL incluir a justificativa (regra + métricas).
3. WHEN configurado SHALL suportar perfis GAMING, DEVELOPMENT, LOCAL AI, BALANCED (padrão) e CUSTOM, independentes e editáveis; perfis SHALL NOT alterar a lista de proteção.
4. WHILE executando SHALL processar 100% localmente (sem envio de dados de processos/hardware), com IA generativa SHALL NOT sendo autoridade para operações administrativas.
5. WHEN executando SHALL manter histórico local configurável, operações assíncronas com cancelamento e degradação honesta em falha de sensor.
