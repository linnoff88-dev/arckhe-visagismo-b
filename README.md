ï»¿# Visagismo B êÂ· v1.1.0

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

Consultoria de estilo para barbearias, em portuguêÂªs, com anêÂ¡lise facial real, revisêÂ£o profissional e implantaêÂ§êÂ£o em **VPS**.

**[Guia pêÂºblico](https://linnoff88-dev.github.io/arckhe-/guia/)** êÂ· [InstalaêÂ§êÂ£o na VPS](VPS.md) êÂ· [Plano original](PLANO.md)

## O que estêÂ¡ implementado

- FastAPI, interface responsiva, pêÂ¡gina pêÂºblica e painel por barbearia.
- Login com senha protegida por scrypt, sessêÂµes revogêÂ¡veis e proteêÂ§êÂ£o CSRF.
- QuestionêÂ¡rio, upload JPEG/PNG/WEBP/HEIC e validaêÂ§êÂ£o de tamanho/resoluêÂ§êÂ£o.
- MediaPipe Tasks com 478 pontos detectados; cêÂ¡lculo e classificaêÂ§êÂ£o herdados do kit.
- RecomendaêÂ§êÂµes em portuguêÂªs, preferêÂªncias do cliente e decisêÂ£o final do barbeiro.
- Fila persistente em SQLite e worker separado, com retomada de processamento interrompido.
- RevisêÂ£o obrigatêÂ³ria, notas profissionais, retorno e relatêÂ³rio imprimêÂ­vel/PDF pelo navegador.
- HistêÂ³rico isolado por barbearia, exclusêÂ£o e retenêÂ§êÂ£o automêÂ¡tica.
- Links revogêÂ¡veis de 7 dias para relatêÂ³rios **sem fotos**.
- SimulaêÂ§êÂ£o frontal opcional com flux2-klein via fal.ai, ou ARCKHÉimg remoto; filtro geomêÂ©trico e fallback sem imagem.
- CobranêÂ§a no caixa e integraêÂ§êÂ£o opcional Mercado Pago Checkout Pro, com assinatura de webhook, consulta do pagamento no servidor e conferêÂªncia do valor.
- Docker Compose, Caddy/HTTPS, volume persistente, backup e workflow de testes.

A jornada pêÂºblica êÂ© **pêÂ¡gina da barbearia ê¢â â pagamento ê¢â â questionêÂ¡rio/foto ê¢â â anêÂ¡lise ê¢â â revisêÂ£o profissional ê¢â â relatêÂ³rio ê¢â â retorno**. O profissional tambêÂ©m pode iniciar a consulta pelo painel. Compartilhe a pêÂ¡gina disponêÂ­vel em ConfiguraêÂ§êÂµes. O cliente acompanha a consulta por link privado de 7 dias; fotos e rascunhos nêÂ£o aparecem no relatêÂ³rio pêÂºblico. Reenvio apêÂ³s falha pode ser liberado pelo barbeiro sem cobrar novamente.

Assinatura automêÂ¡tica B2B e agenda com lembretes nêÂ£o fazem parte desta versêÂ£o; a mensalidade da barbearia êÂ© administrada comercialmente fora do sistema.

## InstalaêÂ§êÂ£o

O destino êÂ© uma VPS Linux com Docker Compose, domêÂ­nio e portas 80/443 disponêÂ­veis. Recomenda-se comeêÂ§ar com 2 vCPUs, 4 GB RAM e 20 GB de disco, validando carga real. NêÂ£o precisa de GPU quando a simulaêÂ§êÂ£o usa fal.ai; sem provedor, a consulta funciona sem imagem gerada.

```bash
git clone https://github.com/linnoff88-dev/visagismo-b.git
cd visagismo-b
cp .env.example .env
# Edite DOMAIN e PUBLIC_URL; configure integraêÂ§êÂµes opcionais.
chmod 600 .env
docker compose up -d --build
docker compose exec app python -m app.manage create-user \
  --email voce@barbearia.com.br --barbearia 'Sua Barbearia' --profissional 'Seu nome'
```

A senha êÂ© solicitada sem aparecer no terminal. NêÂ£o hêÂ¡ login padrêÂ£o. Acesse o domêÂ­nio configurado e conclua os dados da barbearia. Detalhes, cobranêÂ§a, backup e atualizaêÂ§êÂ£o em [VPS.md](VPS.md).

## VerificaêÂ§êÂ£o

```bash
python3 -m venv .venv
.venv/bin/pip install --no-deps -r requirements.txt
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/python -m pytest -q tests
```

As dependêÂªncias estêÂ£o fixadas; `--no-deps` evita instalar um segundo pacote OpenCV pelo MediaPipe. O teste usa uma imagem pêÂºblica de teste do scikit-image, nunca fotos de clientes. NêÂ£o usar o `requisitos.txt` antigo para a aplicaêÂ§êÂ£o nova.

## Estrutura

- `app/`: aplicaêÂ§êÂ£o atual, interface pt-BR, autenticaêÂ§êÂ£o, fila, pagamentos e administraêÂ§êÂ£o.
- `03-motor/`: motor facial original reaproveitado; detector ajustado para rejeitar mais de um rosto.
- `tests/`: fluxo completo, isolamento, acesso, uploads, retenêÂ§êÂ£o, assinatura e conciliaêÂ§êÂ£o.
- `guia/` e `capa/`: pêÂ¡gina pêÂºblica e capa do projeto.
- `06-app/`, `01-estrategia/`, `configura.py`, `marca.json`: referêÂªncia do kit legado em espanhol, nêÂ£o usada pelo Compose atual.
- `docs/legado/`: instruêÂ§êÂµes originais preservadas para referêÂªncia, nêÂ£o sêÂ£o o caminho de implantaêÂ§êÂ£o atual.

## Limites e tratamento de dados

As proporêÂ§êÂµes sêÂ£o estimativas 2D influenciadas pela captura; nêÂ£o validam beleza, personalidade ou saêÂºde. Comparar proporêÂ§êÂµes da imagem gerada **nêÂ£o garante identidade preservada**. SimulaêÂ§êÂ£o lateral fica desabilitada. A pessoa revisa antes da entrega.

Fotos sêÂ£o removidas apêÂ³s 24 horas, relatêÂ³rios apêÂ³s 90 dias (configurêÂ¡vel); worker verifica a cada minuto. Backups precisam de retenêÂ§êÂ£o prêÂ³pria. O administrador deve completar os dados do responsêÂ¡vel e a polêÂ­tica de privacidade antes do atendimento comercial.

O Mercado Pago usa a conta configurada na instalaêÂ§êÂ£o: nêÂ£o existe divisêÂ£o automêÂ¡tica de recebêÂ­veis entre barbearias. Para operadores comerciais independentes, use uma instalaêÂ§êÂ£o e credenciais por empresa. Pagamento online exige credenciais e validaêÂ§êÂ£o sandbox na conta de destino; testes automatizados usam respostas controladas.

## Origem

Adaptado do ZIP `visagismo-barberias.zip` fornecido pelo usuêÂ¡rio. O kit nêÂ£o traz arquivo de licenêÂ§a; esta adaptaêÂ§êÂ£o nêÂ£o declara uma licenêÂ§a sobre o material original. Verifique os termos de origem antes de redistribuir ou comercializar o kit.

---

Projeto  - ARCKHÉe