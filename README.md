# Visagismo B Â· v1.1.0

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

Consultoria de estilo para barbearias, em portuguÃªs, com anÃ¡lise facial real, revisÃ£o profissional e implantaÃ§Ã£o em **VPS**.

**[Guia pÃºblico](https://inematds.github.io/visagismo-b/guia/)** Â· [InstalaÃ§Ã£o na VPS](VPS.md) Â· [Plano original](PLANO.md)

## O que estÃ¡ implementado

- FastAPI, interface responsiva, pÃ¡gina pÃºblica e painel por barbearia.
- Login com senha protegida por scrypt, sessÃµes revogÃ¡veis e proteÃ§Ã£o CSRF.
- QuestionÃ¡rio, upload JPEG/PNG/WEBP/HEIC e validaÃ§Ã£o de tamanho/resoluÃ§Ã£o.
- MediaPipe Tasks com 478 pontos detectados; cÃ¡lculo e classificaÃ§Ã£o herdados do kit.
- RecomendaÃ§Ãµes em portuguÃªs, preferÃªncias do cliente e decisÃ£o final do barbeiro.
- Fila persistente em SQLite e worker separado, com retomada de processamento interrompido.
- RevisÃ£o obrigatÃ³ria, notas profissionais, retorno e relatÃ³rio imprimÃ­vel/PDF pelo navegador.
- HistÃ³rico isolado por barbearia, exclusÃ£o e retenÃ§Ã£o automÃ¡tica.
- Links revogÃ¡veis de 7 dias para relatÃ³rios **sem fotos**.
- SimulaÃ§Ã£o frontal opcional com flux2-klein via fal.ai, ou inemaimg remoto; filtro geomÃ©trico e fallback sem imagem.
- CobranÃ§a no caixa e integraÃ§Ã£o opcional Mercado Pago Checkout Pro, com assinatura de webhook, consulta do pagamento no servidor e conferÃªncia do valor.
- Docker Compose, Caddy/HTTPS, volume persistente, backup e workflow de testes.

A jornada pÃºblica Ã© **pÃ¡gina da barbearia â†’ pagamento â†’ questionÃ¡rio/foto â†’ anÃ¡lise â†’ revisÃ£o profissional â†’ relatÃ³rio â†’ retorno**. O profissional tambÃ©m pode iniciar a consulta pelo painel. Compartilhe a pÃ¡gina disponÃ­vel em ConfiguraÃ§Ãµes. O cliente acompanha a consulta por link privado de 7 dias; fotos e rascunhos nÃ£o aparecem no relatÃ³rio pÃºblico. Reenvio apÃ³s falha pode ser liberado pelo barbeiro sem cobrar novamente.

Assinatura automÃ¡tica B2B e agenda com lembretes nÃ£o fazem parte desta versÃ£o; a mensalidade da barbearia Ã© administrada comercialmente fora do sistema.

## InstalaÃ§Ã£o

O destino Ã© uma VPS Linux com Docker Compose, domÃ­nio e portas 80/443 disponÃ­veis. Recomenda-se comeÃ§ar com 2 vCPUs, 4 GB RAM e 20 GB de disco, validando carga real. NÃ£o precisa de GPU quando a simulaÃ§Ã£o usa fal.ai; sem provedor, a consulta funciona sem imagem gerada.

```bash
git clone https://github.com/inematds/visagismo-b.git
cd visagismo-b
cp .env.example .env
# Edite DOMAIN e PUBLIC_URL; configure integraÃ§Ãµes opcionais.
chmod 600 .env
docker compose up -d --build
docker compose exec app python -m app.manage create-user \
  --email voce@barbearia.com.br --barbearia 'Sua Barbearia' --profissional 'Seu nome'
```

A senha Ã© solicitada sem aparecer no terminal. NÃ£o hÃ¡ login padrÃ£o. Acesse o domÃ­nio configurado e conclua os dados da barbearia. Detalhes, cobranÃ§a, backup e atualizaÃ§Ã£o em [VPS.md](VPS.md).

## VerificaÃ§Ã£o

```bash
python3 -m venv .venv
.venv/bin/pip install --no-deps -r requirements.txt
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/python -m pytest -q tests
```

As dependÃªncias estÃ£o fixadas; `--no-deps` evita instalar um segundo pacote OpenCV pelo MediaPipe. O teste usa uma imagem pÃºblica de teste do scikit-image, nunca fotos de clientes. NÃ£o usar o `requisitos.txt` antigo para a aplicaÃ§Ã£o nova.

## Estrutura

- `app/`: aplicaÃ§Ã£o atual, interface pt-BR, autenticaÃ§Ã£o, fila, pagamentos e administraÃ§Ã£o.
- `03-motor/`: motor facial original reaproveitado; detector ajustado para rejeitar mais de um rosto.
- `tests/`: fluxo completo, isolamento, acesso, uploads, retenÃ§Ã£o, assinatura e conciliaÃ§Ã£o.
- `guia/` e `capa/`: pÃ¡gina pÃºblica e capa do projeto.
- `06-app/`, `01-estrategia/`, `configura.py`, `marca.json`: referÃªncia do kit legado em espanhol, nÃ£o usada pelo Compose atual.
- `docs/legado/`: instruÃ§Ãµes originais preservadas para referÃªncia, nÃ£o sÃ£o o caminho de implantaÃ§Ã£o atual.

## Limites e tratamento de dados

As proporÃ§Ãµes sÃ£o estimativas 2D influenciadas pela captura; nÃ£o validam beleza, personalidade ou saÃºde. Comparar proporÃ§Ãµes da imagem gerada **nÃ£o garante identidade preservada**. SimulaÃ§Ã£o lateral fica desabilitada. A pessoa revisa antes da entrega.

Fotos sÃ£o removidas apÃ³s 24 horas, relatÃ³rios apÃ³s 90 dias (configurÃ¡vel); worker verifica a cada minuto. Backups precisam de retenÃ§Ã£o prÃ³pria. O administrador deve completar os dados do responsÃ¡vel e a polÃ­tica de privacidade antes do atendimento comercial.

O Mercado Pago usa a conta configurada na instalaÃ§Ã£o: nÃ£o existe divisÃ£o automÃ¡tica de recebÃ­veis entre barbearias. Para operadores comerciais independentes, use uma instalaÃ§Ã£o e credenciais por empresa. Pagamento online exige credenciais e validaÃ§Ã£o sandbox na conta de destino; testes automatizados usam respostas controladas.

## Origem

Adaptado do ZIP `visagismo-barberias.zip` fornecido pelo usuÃ¡rio. O kit nÃ£o traz arquivo de licenÃ§a; esta adaptaÃ§Ã£o nÃ£o declara uma licenÃ§a sobre o material original. Verifique os termos de origem antes de redistribuir ou comercializar o kit.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe