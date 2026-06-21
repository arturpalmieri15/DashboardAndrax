# Transformar o Painel Andrax em "site" (PWA) — guia de deploy

Objetivo: hospedar o mesmo HTML num endereco HTTPS para abrir no celular e no
computador, com opcao de "instalar" na tela inicial (cara de app, abre em tela
cheia, funciona offline depois do primeiro acesso). A planilha Excel continua
sendo lida localmente no aparelho — nada de dado vai para servidor.

--------------------------------------------------------------------------------
ANTES DE TUDO: 2 testes que decidem se vale a pena
--------------------------------------------------------------------------------

1. FIREWALL (o mais importante). O motivo de tudo ser inline hoje e o firewall
   da Andrax bloquear CDNs. Um host externo pode cair na mesma regra DENTRO do
   escritorio. Antes de migrar o time:
   - Suba um deploy de teste (passos abaixo).
   - Abra o link de uma maquina do escritorio, na rede corporativa.
   - Se carregar, segue o jogo. Se travar/bloquear, o caminho "site" so vai
     servir no celular (4G/wifi de casa) e fora da rede corporativa.
   Obs.: celular em 4G ou wifi de casa NAO passa pelo firewall, entao para
   mobile costuma funcionar mesmo que o desktop no escritorio bloqueie.

2. ACESSO. Ao virar site, a URL fica alcancavel por quem tiver o link. O DADO
   continua local, mas a TELA do dashboard fica exposta. Por isso o passo 4
   coloca um login (Cloudflare Access) na frente. Nao pule.

--------------------------------------------------------------------------------
ARQUIVOS QUE VAO PARA O HOST (todos na MESMA pasta)
--------------------------------------------------------------------------------

  index.html                 <- seu HTML compilado, renomeado para index.html
  manifest.webmanifest
  sw.js
  icon-192.png
  icon-512.png
  icon-maskable-512.png

Antes de subir: abra o index.html e cole o conteudo de _INSERIR_NO_HEAD.html
dentro de <head>...</head>. Sem isso, vira site comum (nao instala, nao fica
offline).

--------------------------------------------------------------------------------
PASSO 1 — Conta no host (recomendado: Cloudflare Pages)
--------------------------------------------------------------------------------

Por que Cloudflare Pages: gratis, HTTPS automatico, deploy por arrastar pasta
(sem precisar de Git), e tem o Cloudflare Access (login gratis ate 50 usuarios)
para travar o acesso. Alternativas equivalentes: Netlify, GitHub Pages (este
ultimo nao tem login gratis facil).

  1. Crie conta em dash.cloudflare.com
  2. Menu lateral: "Workers & Pages" > "Create" > aba "Pages"
  3. "Upload assets" (deploy direto, sem Git)

--------------------------------------------------------------------------------
PASSO 2 — Subir os arquivos
--------------------------------------------------------------------------------

  1. De um nome ao projeto, ex: painel-andrax
  2. Arraste a pasta inteira com os 6 arquivos
  3. Deploy. Voce recebe uma URL tipo:  https://painel-andrax.pages.dev

--------------------------------------------------------------------------------
PASSO 3 — Testar (faca os 2 testes do topo)
--------------------------------------------------------------------------------

  Desktop no escritorio: abre? carrega a planilha? (teste do firewall)
  Celular: abre a URL > seleciona o Excel > navega normal?
  Offline: depois de abrir 1x, ative modo aviao e recarregue — deve abrir.

--------------------------------------------------------------------------------
PASSO 4 — Colocar login na frente (Cloudflare Access) — NAO PULAR
--------------------------------------------------------------------------------

  1. Em Cloudflare: "Zero Trust" > "Access" > "Applications" > "Add application"
  2. Tipo: "Self-hosted". Aponte para o dominio do seu projeto Pages.
  3. Em "Policies", crie uma regra de allow por e-mail:
     - "Emails ending in" @andrax.com.br   (ou liste os e-mails do time)
  4. Salve. Agora abrir a URL pede um codigo enviado ao e-mail corporativo.
     So quem esta na lista entra.

--------------------------------------------------------------------------------
PASSO 5 — Instalar como app
--------------------------------------------------------------------------------

  Android/Chrome: menu (3 pontos) > "Instalar app" / "Adicionar a tela inicial"
  iPhone/Safari: botao compartilhar > "Adicionar a Tela de Inicio"
  Desktop/Edge ou Chrome: icone de instalar na barra de endereco

  Vira icone proprio, abre em tela cheia, funciona offline.

--------------------------------------------------------------------------------
QUANDO PUBLICAR UMA VERSAO NOVA DO DASHBOARD
--------------------------------------------------------------------------------

  1. Gere o novo HTML, renomeie para index.html, cole de novo o bloco do
     _INSERIR_NO_HEAD.html no <head>.
  2. Em sw.js, troque o numero da versao do cache:
        const CACHE = "andrax-v1";   ->   "andrax-v2"
     (isso forca o aparelho do usuario a baixar a versao nova)
  3. Suba a pasta de novo no mesmo projeto Pages. Os usuarios pegam a
     atualizacao sozinhos no proximo acesso online. Acabou o "rebaixar do
     OneDrive toda vez".

--------------------------------------------------------------------------------
RESUMO DO TRADE-OFF
--------------------------------------------------------------------------------

  Ganha: abre no celular por link/QR, instala como app, offline, auto-update.
  Cuida: testar o firewall no escritorio; manter o login do Access ativo.
  Nao muda: planilha continua lida localmente, nada sobe pra servidor.
