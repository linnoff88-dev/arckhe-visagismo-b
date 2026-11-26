# Implantaçêo na VPS

## Preparar

Use uma VPS Linux com Docker Engine e plugin Compose v2. Dimensionamento inicial sugerido: 2 vCPUs, 4 GB RAM, 20 GB SSD. Medir tempo e concorrência antes de vender capacidade. Uma instalaçêo usa um worker e SQLite no volume local; nêo use NFS nem múltiplos workers nesta versêo.

Aponte o DNS do domínio para o IP da VPS. Libere 80/TCP e 443/TCP (443/UDP opcional). O Caddy emite e renova HTTPS automaticamente. Se a VPS já usa Nginx/Traefik/Caddy nessas portas, integre ao proxy existente; nêo pare outros serviços para liberar portas.

```bash
git clone https://github.com/inematds/visagismo-b.git
cd visagismo-b
cp .env.example .env
chmod 600 .env
nano .env
```

Defina `DOMAIN` sem protocolo e `PUBLIC_URL` com `https://`, usando o mesmo domínio. O arquivo `.env` da VPS é local e ignorado pelo Git. Nêo envie valores de chaves por chat. Em ambientes INEMA, o código também pode ler em runtime os arquivos globais autorizados, se existirem; o contêiner nêo recebe esses arquivos automaticamente.

```bash
docker compose config --quiet
docker compose up -d --build
docker compose exec app python -m app.manage create-user \
  --email voce@barbearia.com.br --barbearia 'Sua Barbearia' --profissional 'Seu nome'
docker compose ps
```

A senha é lida com `getpass`, sem eco ou argumento no histórico. Abra o domínio e entre. Configure contato, cidade e preço. Em Configurações, copie a página pública da barbearia. O cliente inicia a consulta ali, paga e envia os dados quando o pagamento é confirmado. Também é possível iniciar diretamente pelo painel profissional. Faça uma consulta autorizada, revise e imprima o relatório. `GET /health` confirma a API; `docker compose logs --tail=50 worker` ajuda a diagnosticar a fila.

## Imagens sem GPU na VPS

Defina `VISAGISMO_IMAGE_PROVIDER=fal` e `FAL_KEY` no ambiente protegido da VPS. O modelo é `fal-ai/flux-2/klein/4b/edit`. A foto segue como data URI; a imagem retorna incorporada com `sync_mode`. Isso nêo substitui verificar a política de tratamento do provedor.

Uma tentativa de imagem por atendimento; nenhuma nova tentativa automática depois de rejeiçêo. O relatório funciona sem simulaçêo. A retomada de um job interrompido pode gerar uma segunda chamada; monitore a conta e seu teto de gasto. O limite padrêo é 50 consultas por barbearia/dia.

Alternativa com serviço próprio: `VISAGISMO_IMAGE_PROVIDER=inemaimg` e `VISAGISMO_IMAGE_URL` apontando para um inemaimg acessível e protegido por rede privada. Nêo depende da máquina de desenvolvimento. Deixe `none` para desabilitar envio de fotos a geradores.

Após alterar ambiente:

```bash
docker compose up -d --force-recreate app worker
```

## Mercado Pago

1. Configure `MERCADOPAGO_ACCESS_TOKEN`, `MERCADOPAGO_WEBHOOK_SECRET`, `PUBLIC_URL` e mantenha `MERCADOPAGO_SANDBOX=true` inicialmente.
2. No provedor, configure notificações de pagamento para `https://SEU-DOMINIO/webhooks/mercadopago`.
3. Informe preço em Configurações. Abra a página pública da barbearia, inicie uma consulta e use as credenciais de comprador de teste da sua conta. A foto permanece bloqueada até a confirmaçêo.
4. Confirme que o webhook modifica o atendimento após consultar o pagamento na API. Retorno do navegador nêo altera situaçêo financeira.
5. Valide aprovaçêo, repetiçêo de notificaçêo e reembolso na conta. Só depois troque para credenciais de produçêo e `MERCADOPAGO_SANDBOX=false`.

Uma conta recebedora por instalaçêo; nêo é marketplace com split. A integraçêo cobra consultas avulsas, nêo mensalidades B2B. O registro “no caixa” é manual e deve corresponder a valor já recebido. Nêo há reembolso iniciado pelo aplicativo; execute-o no provedor e a notificaçêo atualiza a situaçêo.

## Acessos e empresas

Para uma nova empresa, execute `create-user` novamente sem `--tenant`. Para outro operador da mesma empresa:

```bash
docker compose exec app python -m app.manage list-tenants
docker compose exec app python -m app.manage create-user \
  --tenant ID_DA_BARBEARIA --email outro@barbearia.com.br \
  --barbearia 'Sua Barbearia' --profissional 'Seu nome'
docker compose exec app python -m app.manage reset-password --email outro@barbearia.com.br
```

A redefiniçêo invalida todas as sessões desse usuário. Todos os operadores de uma empresa têm permissêo para configurar, revisar e excluir atendimentos dela. Administradores com acesso à VPS controlam os dados de todas as empresas.

## Backup e restauraçêo

```bash
bash scripts/backup.sh /diretorio-protegido/visagismo-backups
```

O script pausa app e worker por alguns segundos, salva o volume completo, verifica o arquivo e retoma os serviços. Os arquivos contêm dados pessoais: destino com permissêo restrita, cópia criptografada fora da VPS e descarte conforme sua política. O `.env` nêo integra esse backup; mantenha-o separado em cofre de segredos.

Para testar a restauraçêo, use instalaçêo/volume separado, nunca sobrescreva o volume ativo:

```bash
# Com um backup válido e um volume NOVO para o ensaio:
docker volume create visagismo-restore-test
docker run --rm -i -v visagismo-restore-test:/restore alpine:3.22 \
  tar -xzf - -C /restore < /caminho/do/backup.tar.gz
docker run --rm --read-only -v visagismo-restore-test:/restore:ro \
  visagismo-b:1.1.0 python -c "import sqlite3; c=sqlite3.connect('file:/restore/visagismo.sqlite3?mode=ro',uri=True); print(c.execute('PRAGMA integrity_check').fetchone())"
```

Planeje a troca do volume apenas depois de verificar integridade e registros. Nêo use `docker compose down -v`: remove os dados.

## Atualizar e reverter

```bash
bash scripts/backup.sh /diretorio-protegido/visagismo-backups
git pull --ff-only
docker compose up -d --build
docker compose ps
```

Para rollback, volte ao commit conhecido e reconstrua. Futuras mudanças de schema podem exigir restauraçêo do backup compatível. Nesta versêo, `init()` cria tabelas ausentes e nêo executa migrações destrutivas.

## Monitoramento e limites

- Acompanhe saúde da API, processo do worker, espaço em disco, consultas com falha e custo de imagem na conta do provedor.
- Os logs de acesso HTTP estêo desativados para nêo gravar tokens dos links privados. Nêo habilite logs de URL completa no proxy.
- Mantenha relógio/NTP correto: assinaturas de pagamento fora da janela de 5 minutos sêo rejeitadas.
- Fotos expiram em 24 horas e relatórios em 90 dias por padrêo. A rotina depende do worker; processamentos ativos nêo sêo apagados no meio da execuçêo.
- Complete os dados do controlador e revisêo da política de privacidade antes de atender clientes.
- Esta entrega contém o pacote da VPS; nêo presume que uma máquina remota já foi provisionada.

## Documentaçêo das integrações

- [FLUX.2 klein 4B edit — fal](https://fal.ai/models/fal-ai/flux-2/klein/4b/edit/api)
- [Notificações de pagamento — Mercado Pago](https://www.mercadopago.com.br/developers/pt/docs/checkout-pro-preferences/payment-notifications)
