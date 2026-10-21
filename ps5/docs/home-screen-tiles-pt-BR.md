# Atalhos na Home do PS5 (AI-assisted)

Investigação de 20/10/2026, a partir do commit
`bea3c2e8420934aad24d5665eb75b8a6e462d452` e da versão oficial
[PS5SX2 2.15, build 177](https://github.com/Swordpdf/PS5SX2/releases/tag/vk-285-177).

## Criar um atalho para um jogo em arquivo

A versão 2.15 já oferece essa função, desativada por padrão:

1. Use o aplicativo 2.15 e o helper 171g compatível com seu ShadowMount+.
2. Crie o arquivo vazio `/data/PCSX2/flags/gametiles`, sem extensão.
3. Reinicie o console. Carregue ShadowMount+ e depois apenas um helper.
4. Abra PS5SX2, selecione o jogo em arquivo na biblioteca e pressione Quadrado.
5. Na opção **Home screen tile**, pressione Cross para adicionar o atalho.
6. Feche PS5SX2 completamente. Segundo as notas da versão, o atalho aparece
   aproximadamente 30 segundos depois, quando nenhum jogo está rodando.

O jogo precisa continuar no caminho em que estava ao criar o atalho. O início
por esse atalho usa o arquivo e não depende da presença de um disco. Para
removê-lo, use novamente a opção pelo aplicativo principal PS5SX2.

O helper `PS5SXHelper.elf` monta `/data` e é indicado nas notas da versão para
ShadowMount+ 1.7beta3. Com ShadowMount+ 1.7-beta5fix1 ou posterior, as notas
indicam `PS5SXHelper-no-data-mount.elf`. Reinicie antes de trocar de helper.

Os marcadores `nogametiles` (aplicativo) e `gametile_off` (helper) desativam
a função mesmo quando `gametiles` existe. A implementação reserva dez IDs,
`PPSA99251` a `PPSA99260`, para esses atalhos.

## Como os dois tipos de atalho funcionam

O atalho de disco carrega `disc-tile.txt`, que identifica o leitor. Em
`ps5/coreorbis/main-boot.cpp`, o aplicativo espera o leitor, identifica o serial
do disco e procura uma cópia completa com `OrbisFindDiscCopy`. Se a encontrar,
usa o arquivo; caso contrário, inicia pelo leitor. Essa seleção ainda depende
do disco para identificar o jogo. O helper fornecido contém mensagens de
desinstalação do atalho quando a mídia é removida.

O atalho de arquivo carrega `game-tile.txt`, que identifica diretamente o
caminho do jogo. O menu (`ps5/frontend/fe_app.cpp`) e a interface web
(`/api/tile`) escrevem pedidos em `/data/PCSX2/tiles`. O helper cria a cópia
do aplicativo com o ID próprio e o marcador. O aplicativo lê o marcador antes
da elevação, valida o arquivo e inicia o jogo; se o arquivo faltar, abre a
biblioteca e informa o problema.

Esses atalhos permanentes não convertem automaticamente o mesmo ID do atalho
temporário do disco. Implementar essa conversão exige coordenar a identidade
do jogo, sua cópia local e o ciclo de criação/remoção controlado pelo helper.

## Helper examinado e limite da investigação

O ELF fornecido com o nome `PS5SX2 Disc Helper.elf` tem 2.406.744 bytes e SHA-256
`f7960126abe5ad30869fbe01235fecf2bfac71bd90ee3b9c9eb22810b699e736`.
É idêntico ao `PS5SXHelper.elf` oficial 171g da versão 2.15. Foi examinado como
dado; não foi executado nem alterado.

`ps5/frontend/fe_gametile.h` referencia `tile6/gametile.c` e
`plans/gametiles_contract.md`, mas esses arquivos não constam desta árvore
pública. `ps5/disclaunch/disc_launch.c` trata a inicialização por disco e seus
sinais; não contém o gerenciador completo de atalhos e sua remoção no helper.
O ELF compilado não substitui esse código-fonte para uma alteração verificável.

O teste existente
`node ps5/frontend/host/gametile_page_test.js ps5/frontend/assets/web/index.html`
passou. Ele verifica a interface web e seus pedidos com um console simulado;
não verifica a instalação de atalhos nem sua persistência em um PS5 real.
Não houve teste no console nem alteração no comportamento do emulador.
