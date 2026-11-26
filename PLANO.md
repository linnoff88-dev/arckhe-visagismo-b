# Visagismo com IA — plano de execuçêo

Data: 17/09/2026 · Projeto: `visagismo-b` · Status: planejamento, implementaçêo ainda nêo iniciada.

## 1. Objetivo e idioma

Transformar o kit recebido em um serviço de consultoria de estilo vendido por barbearias: o cliente recebe recomendações práticas, revisadas pelo barbeiro, e um plano de manutençêo. O produto apoia a decisêo profissional; nêo promete um corte ideal universal nem resultados garantidos.

**Idioma escolhido: português do Brasil.** Interface, mensagens, relatórios, documentaçêo e materiais comerciais em pt-BR. Novos identificadores técnicos em inglês; nomes legados em espanhol serêo migrados gradualmente, preservando contratos e testes. Inglês na interface fica para uma etapa posterior.

Fonte inspecionada: `/home/nmaldaner/thinclient_drives/.clipboard/visagismo-barberias.zip`. A pasta local estava vazia. Este plano resulta da leitura estática do código e das instruções do ZIP; nêo equivale à validaçêo do aplicativo em execuçêo. O conteúdo fornecido pelo usuário foi suficiente: o curso no Skool nêo foi acessado.

## 2. O que existe e o que falta comprovar

| Área | Evidência no ZIP | Consequência para o plano |
|---|---|---|
| Aplicativo | FastAPI, HTML/JS e Jinja2 em `06-app/` | Reaproveitar antes de considerar reescrita |
| Medições | MediaPipe e modelo local em `03-motor/` | Validar repetibilidade e qualidade das fotos |
| Recomendações | Motor de regras e integraçêo opcional com modelo de linguagem | Exigir rastreabilidade entre recomendaçêo, medida e resposta |
| Relatório | Renderizaçêo HTML, geometria e plano de visitas | Localizar e incluir aprovaçêo profissional real |
| Imagens | `simulacion.py` usa fal.ai com `nano-banana-pro/edit` | Isolar fornecedor e avaliar adaptaçêo ao padrêo flux2-klein |
| Verificaçêo visual | Seis proporções; limites atuais de 3% de desvio médio e 6% máximo | Tratar como filtro geométrico experimental, nêo prova de identidade |
| Foto lateral | Mediçêo explicitamente experimental; imagem lateral nêo passa pelo mesmo verificador frontal | Desabilitar simulaçêo lateral no MVP até haver validaçêo específica |
| Pagamento | Checkout simulado | Nêo aceitar como comprovaçêo de pagamento |
| Histórico | Integraçêo opcional com Supabase | Nêo confundir com painel multiempresa pronto |
| Marca branca | `marca.json` e script de substituiçêo | Atende personalizaçêo de uma instalaçêo; nêo comprova isolamento entre empresas |
| Retençêo | Limpeza local acionada ao gerar novo relatório | Criar rotina agendada e política separada para banco, storage e backups |
| Publicaçêo | Docker, configurações e workflow incluídos | Testar build e revisar infraestrutura antes de usar |
| Licença | Nenhum arquivo de licença encontrado no ZIP | Confirmar condições de reutilizaçêo comercial e distribuiçêo antes de comercializar |

O código contém números de custo e alegações jurídicas do material original. Eles nêo foram verificados e nêo serêo tratados como fatos atuais. O valor citado de €0,30 nêo é orçamento de operaçêo.

## 3. Produto inicial

**Comprador:** dono ou gestor de barbearia. **Operador:** barbeiro. **Beneficiário:** cliente que quer escolher e manter um estilo compatível com sua rotina.

Oferta inicial: consulta de estilo com questionário, fotos orientadas, sugestões de corte/barba, explicaçêo, simulaçêo opcional, relatório aprovado e proposta de retorno. A consulta pode ser avulsa ou fazer parte de um pacote de atendimento.

Começar com 1–3 barbearias e atendimento assistido. Primeiro validar utilidade, tempo de atendimento e disposiçêo de pagar; depois automatizar aquisiçêo e cobrança.

### Incluído no MVP

- Marca, contato e identidade visual da barbearia.
- Questionário sobre preferências, rotina e manutençêo desejada.
- Captura/upload frontal com orientaçêo e rejeiçêo de imagens inadequadas.
- Medições frontais, regras explicáveis e texto em português.
- Revisêo pelo barbeiro, ajustes e aprovaçêo registrados.
- Relatório HTML responsivo e versêo imprimível.
- Plano de retorno definido ou confirmado pelo profissional.
- Simulaçêo frontal opcional, com reprovaçêo automática e revisêo humana.
- Autenticaçêo do operador, acesso controlado aos relatórios e exclusêo de dados.
- Registro de duraçêo, falhas e custo por atendimento.

### Após a validaçêo inicial

Pagamento online real, painel multiempresa compartilhado, assinatura das barbearias, lembretes opt-in e acompanhamento longitudinal. Aplicativo nativo, marketplace, reconhecimento de clientes pelo rosto e expansêo para saúde ficam fora deste projeto inicial.

## 4. Jornada e estados

Jornada comercial completa:

`landing → oferta → pagamento confirmado → questionário e fotos → análise → revisêo profissional → relatório → retorno`

No primeiro piloto assistido, o operador pode abrir um atendimento e registrar uma cobrança realizada no caixa. Essa alternativa deve aparecer explicitamente como cobrança externa; nunca como transaçêo online aprovada.

Estados sugeridos:

`draft → awaiting_payment → paid → submitted → processing → pending_review → delivered`

Falhas técnicas terêo estado `failed`, com retomada controlada. Cancelamento, reembolso e exclusêo terêo registros próprios. O histórico de pagamento nêo será inferido do estado da análise.

O relatório apresenta observações medidas, preferências informadas, sugestões, esforço de manutençêo e próxima visita. Nêo atribuir personalidade, saúde, origem étnica ou valor estético à pessoa a partir do rosto. Substituir linguagem de “defeito” e “correçêo” por escolhas de estilo.

## 5. Arquitetura proposta

Preservar **Python + FastAPI + MediaPipe + Jinja2 + HTML/JS**. Evitar migraçêo de framework sem problema concreto.

### Piloto

Uma instalaçêo isolada por barbearia, configurada por marca, usando contêiner e armazenamento persistente. Fluxo síncrono inicialmente, somente se medições de tempo e concorrência mostrarem que cabe no ambiente. Definir timeouts e quantidade máxima de análises simultâneas.

### Operaçêo compartilhada

Adicionar PostgreSQL/Supabase, armazenamento privado, autenticaçêo e isolamento por `tenant_id`. Introduzir worker e fila persistente quando houver execuçêo longa ou concorrência; nunca depender de dicionário em memória ou disco efêmero entre instâncias. O navegador acompanha o estado persistido do atendimento.

Entidades previstas: `tenants`, `users`, `clients`, `consultations`, `photo_assets`, `measurements`, `recommendations`, `simulations`, `reports`, `reviews`, `visit_plans`, `payments`, `subscriptions` e `usage_events`. Versões das regras, modelos e relatórios devem acompanhar cada atendimento.

A configuraçêo de marca deve ser lida em runtime ou por template, substituindo gradualmente as substituições textuais do script legado.

### Uso de IA

- As regras continuam funcionando sem serviços pagos.
- O modelo de texto recebe dados estruturados; validar suas afirmações contra a origem. JSON válido nêo garante conteúdo correto.
- Manter coerência entre receita estruturada, texto e prompt da simulaçêo.
- Criar uma interface de ediçêo de imagem com **flux2-klein como candidato padrêo**, conforme preferência do projeto. Confirmar suporte à ediçêo com referência e medir qualidade antes de adotá-lo em produçêo.
- Se a ediçêo nêo preservar o rosto, entregar sem simulaçêo. Nêo publicar imagem reprovada nem regenerar indefinidamente.
- Limitar tentativas e custo por atendimento; registrar também o custo de imagens rejeitadas.
- Carregar segredos em runtime, procurando primeiro nos dois arquivos globais autorizados; nunca copiar chaves para o projeto ou registrá-las em logs.

## 6. Etapas de implementaçêo e aceite

Estimativa inicial: **6–8 semanas de trabalho de uma pessoa**, com escopo contido; o acompanhamento de retorno pode exigir mais tempo de calendário. Reestimar após executar a base.

| Etapa | Trabalho | Evidência necessária para avançar |
|---|---|---|
| 0 — Base reproduzível · 1–2 dias | Extrair o ZIP com segurança na raiz deste projeto, preservar referência do original, ler instruções, preparar ambiente, executar scripts existentes e inventariar dependências | Servidor inicia; relatório sem APIs é gerado; limitações e versões ficam documentadas |
| 1 — Português e posicionamento · 2–4 dias | Traduzir interface, relatório, erros, prompts de saída, datas e materiais; configurar marca; mapear valores de formulário | Jornada visível em pt-BR; formulários continuam alimentando as mesmas regras; nenhuma mudança de lógica por traduçêo |
| 2 — Confiabilidade do núcleo · 4–6 dias | Qualidade de captura, limites de upload, validaçêo de textos, fallback, revisêo do barbeiro e estados de falha | Foto inválida é rejeitada; indisponibilidade de IA mantém relatório útil; entrega exige aprovaçêo |
| 3 — Piloto assistido · 5–7 dias | Login, armazenamento privado, acesso ao relatório, exclusêo, custos e treinamento | Fluxo real com dados autorizados; acesso indevido bloqueado; retençêo executada mesmo sem novos atendimentos |
| 4 — Simulaçêo frontal · 3–5 dias | Adaptador de ediçêo, teste do flux2-klein, avaliaçêo humana e calibraçêo do filtro geométrico | Comparaçêo documentada em amostra variada; rejeições funcionam; relatório continua disponível sem imagem |
| 5 — Cobrança e operaçêo compartilhada · 5–8 dias | Gateway, webhooks, idempotência, assinaturas, isolamento por empresa e painel mínimo | Pagamento sandbox confirmado no servidor; duplicidade nêo gera cobrança/entrega extra; empresa A nêo acessa B |
| 6 — Preparaçêo para venda · 3–5 dias | Backup/restauraçêo, observabilidade, limites, onboarding, oferta comercial e documentaçêo | Ensaio de restauraçêo; custos conhecidos; suporte definido; piloto avaliado |

A etapa 5 depende de evidência de utilidade no piloto. A simulaçêo pode ser adicionada durante o piloto; nêo é condiçêo para testar o valor da consulta.

### Primeira semana — ordem prática

1. Importar o kit sem criar outro repositório e registrar o estado original.
2. Executar a demo sem APIs externas e os scripts `prueba_motor.py` e `prueba_informe.py`, verificando pré-requisitos e fixtures.
3. Mapear strings visíveis e valores internos acoplados a textos em espanhol, especialmente em `simulacion.py`.
4. Traduzir a jornada principal e gerar um relatório pt-BR de ponta a ponta.
5. Documentar falhas observadas e priorizar o mínimo necessário para atendimento assistido.

## 7. Validaçêo técnica e de qualidade

- **Motor:** entradas conhecidas, proporções finitas, rejeiçêo de geometria inválida e repetibilidade sob pequenas mudanças de captura. Documentar que medições 2D dependem de pose, iluminaçêo e câmera.
- **Captura:** JPEG, PNG, WEBP e HEIC; arquivo corrompido, extensêo falsa, foto grande, nenhuma face, mais de uma face e rosto inclinado. O detector atual configurado para uma face nêo basta para rejeitar imagens com várias pessoas.
- **Relatório:** regras sem IA; IA indisponível; saída incoerente; nome com caracteres especiais; recomendaçêo compatível com o questionário; revisêo e impressêo no celular.
- **Simulaçêo:** rosto nêo detectado, proporçêo alterada, imagem aceitável, timeout e saldo esgotado. Avaliar também reconhecimento visual pelo cliente e barbeiro; proporções semelhantes nêo garantem identidade preservada.
- **Dados e acesso:** URL de outro atendimento, tentativa entre empresas, sessêo expirada, exclusêo dos arquivos e histórico, dados pessoais fora dos logs.
- **Pagamentos:** webhook com assinatura inválida, evento duplicado, evento fora de ordem, cancelamento, reembolso e reprocessamento após falha.
- **Operaçêo:** reinício durante geraçêo, duas solicitações simultâneas, restauraçêo de backup e teto de custo.

Começar a avaliaçêo facial e das simulações com 20–30 participantes autorizados, com variedade de formatos de rosto, tons de pele, texturas de cabelo, barba e condições de captura. É uma amostra exploratória; nêo demonstra validade universal. Registrar divergências entre sistema e profissional para ajustar regras e linguagem.

## 8. Dados pessoais e operaçêo

Antes de usar fotos de clientes, revisar o tratamento no contexto brasileiro com responsável qualificado: finalidade, fundamento aplicável, comunicaçêo ao cliente, terceiros que recebem fotos, prazos, exclusêo e responsabilidades da barbearia e do fornecedor. Nêo transplantar as conclusões sobre RGPD do ZIP para a LGPD.

Separar fotos temporárias do histórico necessário ao acompanhamento. A retençêo de 24 horas do legado é uma configuraçêo inicial, nêo garantia de expurgo nem regra jurídica. Implementar exclusêo agendada e verificar também Supabase, armazenamento e backups.

No piloto, trabalhar com adultos e fotos autorizadas. Desabilitar compartilhamento público e evitar identificaçêo biométrica. Relatórios devem indicar quando uma imagem é simulaçêo gerada por IA.

O Supabase legado usa políticas de inserçêo anônima; revisar esse desenho antes do serviço compartilhado. Nêo expor credenciais administrativas e testar autorizaçêo no servidor e no banco.

## 9. Modelo comercial e custos

Separar duas receitas:

1. **Fornecedor → barbearia:** implantaçêo por configuraçêo e treinamento; mensalidade por operaçêo, suporte e franquia de uso; excedente por atendimento, se necessário.
2. **Barbearia → cliente:** consulta de estilo avulsa ou integrada ao corte e acompanhamento.

A recorrência precisa corresponder a entregas úteis: ajustar o plano, acompanhar crescimento, mudar estilo ou manter o corte. Nêo exigir uma nova análise completa a cada visita sem necessidade.

Definir preços depois de medir:

`custo por relatório entregue = (APIs + tentativas rejeitadas + infraestrutura alocada + suporte variável) / relatórios entregues`

`margem de contribuiçêo = receita − taxas − tributos variáveis − custos variáveis`

`barbearias para equilíbrio = custos fixos mensais / contribuiçêo média mensal por barbearia`

A contribuiçêo deve ser positiva para a última fórmula fazer sentido. Incluir o tempo adicional do barbeiro ao avaliar o ganho para a barbearia. Implantaçêo cobre trabalho inicial, nêo mascara uma mensalidade deficitária. Nêo prometer aumento de faturamento sem medir.

## 10. Piloto comercial e decisêo de continuidade

Entrevistar cinco barbeiros antes de fixar oferta e preço. Selecionar 1–3 parceiros e realizar inicialmente cerca de 30 consultas assistidas, registrando:

- Quantos clientes receberam a oferta, aceitaram e pagaram.
- Tempo total e tempo de intervençêo do barbeiro.
- Utilidade percebida e quantidade de recomendações corrigidas.
- Custo por atendimento e taxa de falha/rejeiçêo de simulaçêo.
- Retorno agendado e retorno efetivo em 30–60 dias.
- Interesse da barbearia em continuar pagando e motivos de desistência.

Metas provisórias para o piloto, a ajustar com os parceiros: pelo menos 90% das entradas válidas produzirem relatório sem intervençêo técnica; 100% dos relatórios entregues terem revisêo; pelo menos 80% das consultas avaliadas como úteis pelo cliente e pelo profissional; margem de contribuiçêo positiva; ao menos dois parceiros interessados em continuidade paga se houver três participantes.

Nêo atribuir causalidade ao retorno sem comparaçêo adequada. Separar intençêo de renovaçêo de renovaçêo realmente paga. Se o texto for útil e a simulaçêo falhar, manter o serviço sem imagem; se a consulta nêo gerar valor, ajustar a oferta antes de ampliar a plataforma.

## 11. Publicaçêo e continuidade

Código e guia futuro permanecem neste repositório `visagismo-b`. Quando solicitado, o guia público será `guia/index.html`; o aplicativo Python precisará de ambiente próprio de execuçêo — GitHub Pages serve somente conteúdo estático.

Publicaçêo será via Git, respeitando a conta de destino e autoria dos commits. Nêo operar o painel do Vercel nem depender do status dele para concluir um push. Nêo foi criado remoto nem feito deploy nesta etapa de planejamento.

**Próxima entrega recomendada:** demo local em português, funcionando sem APIs pagas, com relatório revisável pelo barbeiro e limitações explícitas. Depois disso, iniciar o piloto e medir se o serviço merece a camada de cobrança e multiempresa.
