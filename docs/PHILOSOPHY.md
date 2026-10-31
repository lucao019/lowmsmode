# PHILOSOPHY — Knowledge Should Be Free

**Estado: ✔️ Adotada (decisão do mantenedor, 2026-10-31)**
**Licença: em avaliação — ver §6, aguardando aprovação do mantenedor.**

> **CONTROL YOUR SYSTEM. UNLOCK YOUR PERFORMANCE.**
> **KNOWLEDGE IS MEANT TO BE SHARED.**

## 1. A essência do NEXUS OPTIMIZER

O NEXUS OPTIMIZER **não nasce com a intenção de gerar lucro**. Não é um projeto criado para vender licenças, cobrar assinaturas, inserir anúncios ou transformar conhecimento técnico em barreira financeira.

Ele nasce de uma ideia simples: **compartilhar conhecimento é uma das formas mais poderosas de contribuir com a comunidade de tecnologia.** O propósito é desenvolver uma ferramenta útil, disponibilizar seu código, documentar o conhecimento adquirido e permitir que outras pessoas aprendam, utilizem, modifiquem e contribuam.

O conhecimento não deve ficar preso a uma única pessoa, empresa ou ferramenta.

## 2. Princípios fundamentais

Toda decisão relacionada ao NEXUS deve respeitar:

1. **Conhecimento livre** — compartilhar soluções, experiências, descobertas, documentação e conhecimento técnico.
2. **Colaboração** — facilitar a participação de outros desenvolvedores, independentemente do nível de experiência.
3. **Transparência** — explicar como cada funcionalidade funciona, quais alterações realiza e quais são seus riscos.
4. **Acessibilidade** — ferramenta compreensível por pessoas com diferentes níveis de conhecimento técnico.
5. **Evolução coletiva** — qualquer pessoa pode estudar o código, identificar problemas, sugerir melhorias e contribuir.
6. **Respeito** — comunidade colaborativa, sem arrogância técnica, discriminação ou desvalorização de iniciantes.
7. **Responsabilidade** — compartilhar conhecimento sem comprometer a segurança, privacidade ou integridade dos computadores dos usuários.

## 3. Compromisso open source

O repositório é público e permite que a comunidade: consulte o código-fonte, compreenda a arquitetura, aprenda com as implementações, utilize o software respeitando a licença, crie melhorias, corrija problemas, compartilhe soluções, desenvolva extensões, proponha funcionalidades e contribua com documentação.

Regras:

- Respeitar integralmente as licenças de bibliotecas, dependências, ícones, fontes e demais recursos de terceiros.
- Não remover créditos de autores de componentes externos.
- Licença do próprio projeto: **sob avaliação, nunca escolhida arbitrariamente** (ver §6).

## 4. Política não comercial

O NEXUS não incluirá, por iniciativa própria:

- Assinaturas pagas ou funcionalidades bloqueadas por pagamento;
- Publicidade ou rastreamento comercial;
- Venda de dados ou coleta desnecessária de informações;
- Dependência obrigatória de serviços pagos;
- Limitações artificiais para forçar compras.

O aplicativo deve funcionar **localmente** sempre que tecnicamente possível. Nenhum sistema de monetização sem decisão expressa do mantenedor.

## 5. Documentação para ensinar

Não basta documentar **o que** o código faz — documentamos **como e por que** ele funciona. Sempre que possível:

- o problema que motivou a implementação;
- o conceito técnico envolvido e a tecnologia utilizada;
- as alternativas consideradas e a justificativa da decisão;
- os riscos e limitações;
- como reproduzir, testar e contribuir.

Documentação acessível a iniciantes, tecnicamente consistente para desenvolvedores experientes. Exemplos práticos, diagramas e referências oficiais. Nada de documentação superficial ou gerada para preencher arquivo.

## 6. Avaliação de licença (aguardando aprovação)

Candidatas avaliadas para um projeto comunitário sem fins lucrativos:

| Licença | O que permite | Implicações | Quando escolher |
|---|---|---|---|
| **MIT** | Uso, cópia, modificação, redistribuição e até uso comercial, exigindo apenas manter copyright + licença | Máxima adoção e simplicidade; **não** impede que terceiros vendam derivados | Padrão da indústria para projetos educacionais/comunitários |
| **Apache-2.0** | O mesmo que MIT, **mais** concessão expressa de patentes e exigência de NOTICE de modificações | Um pouco mais burocrática; protege contribuidores contra litígio de patentes | Quando houver preocupação com patentes ou contribuição corporativa |
| **GPLv3** (referência) | Uso livre, mas **derivados devem permanecer open source** (copyleft) | Garante que melhorias voltem à comunidade; incompatível com alguns ecossistemas .NET fechados | Quando a prioridade máxima for impedir versões proprietárias |

**Recomendação técnica: MIT — ✔️ APROVADA pelo mantenedor em 2026-10-31.**

Motivos: alinhamento direto com "knowledge should be free" (mínima fricção para estudar, usar e reutilizar), é a licença mais compreendida por iniciantes, combina com o ecossistema .NET (as próprias dependências do projeto — Wpf.Ui, CommunityToolkit.Mvvm, LiveCharts2, Dapper — são MIT ou Apache-2.0) e não impede futura migração se a comunidade pedir diferente. O risco aceito: alguém pode criar um derivado comercial — o projeto vive da comunidade, não da exclusividade do código.

> Licença **MIT adotada** (arquivo `LICENSE` na raiz) após aprovação expressa do mantenedor. Qualquer mudança futura de licenciamento exigiria nova decisão expressa dele.

## 7. Privacidade e autonomia dos usuários

- Processamento local como padrão; transparência sobre qualquer informação coletada.
- Ausência de telemetria não autorizada — o app não tem telemetria (RNF-05).
- Nenhum envio oculto de informações do hardware; nenhuma coleta de arquivos pessoais; nenhum compartilhamento sem consentimento.
- Qualquer funcionalidade futura que dependa de comunicação externa deve informar claramente sua finalidade antes de existir.

## 8. Contribuição da comunidade

Canais e regras: [CONTRIBUTING.md](../CONTRIBUTING.md) · [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md) · [SUPPORT.md](../SUPPORT.md) · [Política de vulnerabilidades](../.github/SECURITY.md) · templates em [.github/](../.github/).

Categorias organizadas: bug fixes, performance, compatibilidade de hardware, novas funcionalidades, documentação, UI/UX, traduções e testes — com incentivo explícito a estudantes e iniciantes (nenhuma contribuição exige conhecimento avançado; documentação e tradução são portas de entrada legítimas).

## 9. Reconhecimento

- Seção de colaboradores no README reconhece código, documentação, testes, traduções, relatos de problemas e sugestões relevantes.
- Nunca atribuir contribuição a quem não participou; nunca remover créditos de autores externos.

## 10. Diretriz para futuras sessões de IA (e de humanos)

Quem trabalhar neste repositório não está desenvolvendo apenas um produto — está contribuindo com um projeto comunitário. Portanto:

- Não introduzir monetização por conta própria; não esconder conhecimento técnico; não produzir documentação artificial.
- Não dificultar contribuições; não impor dependências comerciais desnecessárias; não comprometer a transparência.
- Não descartar contribuições sem justificativa técnica; explicar decisões de arquitetura.
- Quando existir solução simples, livre, segura e eficiente, considerá-la antes de recomendar alternativa comercial.
- A qualidade técnica permanece alta — gratuidade não é desculpa para descuido.

## 11. Mensagem institucional

> Construído com conhecimento compartilhado, colaboração e respeito pela comunidade de tecnologia.
>
> O NEXUS OPTIMIZER é um projeto independente, desenvolvido com o propósito de ajudar pessoas a compreender, otimizar e aproveitar melhor seus computadores.
>
> Não buscamos transformar conhecimento em produto inacessível. Buscamos aprender, compartilhar e evoluir juntos.

## 12. Declaração final

Se alguém aprender programação estudando este código, já existe um resultado positivo. Se alguém resolver um problema no computador usando a ferramenta, o projeto cumpriu parte de sua missão. Se outro desenvolvedor melhorar uma funcionalidade, todos ganham. Se alguém usar o conhecimento para criar algo ainda melhor, a comunidade evoluiu.

**Não estamos construindo conhecimento para guardá-lo. Estamos construindo para compartilhá-lo.**
