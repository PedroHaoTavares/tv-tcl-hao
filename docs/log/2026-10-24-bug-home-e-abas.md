# 24/10/2026: bug do botão Home, abas por biblioteca e prateleira

## Bug: Home não saía da tela do app

**Sintoma (relatado pelo Sr. Hao):** depois de abrir um episódio e apertar Home no controle, a tela do Jellyfin continuava visível. As setas faziam o som de navegação do launcher, ou seja, a home estava na frente, mas não aparecia. [Captura](../img/2026-10-24-bug-home-preso.png)

**Investigação (só leitura, via ADB):**

1. `dumpsys activity`: a home do Free My TV estava `RESUMED` e com o foco, então recebia as teclas. O app deixado para trás continuava `PAUSED` e visível (`mVisible=true`), **nunca `STOPPED`**.
2. A tarefa da home aparecia com `translucent=true` e `occludesParent=false`. Para o Android, uma tela translúcida não cobre a de trás, então o app de trás não é parado e continua desenhado.
3. **Acontecia com qualquer app** (Netflix e Jellyfin pela prateleira, Jellyfin pelo cartão). Não era culpa da integração com o Jellyfin.
4. O tema `FreeMyTV.Home` é opaco no APK (`aapt2 dump resources`). Declarar `windowIsTranslucent/Floating/SwipeToDismiss=false` **não mudou nada** (hipótese descartada; mudança revertida).
5. Não há nada no código que deixe a janela translúcida. A marcação vem do sistema da TCL, cujo código não é público.

**Correção:** `HomeActivity.onCreate` chama `setTranslucent(false)` (API pública do Android 11+), que informa ao sistema que a tela é opaca.

**Verificação:** home com `translucent=false`. Depois de Home, o Jellyfin fica `STOPPED` e a captura mostra a home. Testado com Netflix e com o cartão do Jellyfin.

## Abas = bibliotecas do Jellyfin

Nova opção `jellyfin_tabs=Filmes,Séries,Desenhos,Animes`. Cada aba mostra a biblioteca de mesmo nome (comparação sem acento e sem maiúsculas):

1. o que está **em andamento** (`/Items/Resume?ParentId=`);
2. depois os **próximos episódios** (`/Shows/NextUp?ParentId=`), marcados "Próximo episódio";
3. depois as **novidades** (`/Items/Latest?ParentId=`), marcadas "Novo".

Assim, nenhuma aba fica vazia. O `HomeView` tinha 3 abas fixas no código; o número virou a constante `TABS`. Os 3 cartões continuam 3.

Capturas: [Desenhos](../img/2026-10-24-aba-desenhos.png), [Animes](../img/2026-10-24-aba-animes.png)

## "Dólar hoje: $ 5,01"

Com `currency=BRL`, a cotação USDT aparece como texto completo, sem o ícone separado.

## Prateleira

Nova opção `shelf=` (pacotes em ordem). Configurada com: Jellyfin, Globoplay, Prime Video, YouTube e Spotify. Netflix, Max, Apple TV e Disney+ saíram da prateleira, **mas continuam instalados** na TV (Apple TV, Max, Disney+ e Star+ são apps instalados; a Netflix é app de sistema da TCL).

## Observação: cartão do Norbit em branco

Não é bug. A imagem de fundo desse filme no servidor é branca no centro, e o cartão vertical recorta o centro. Melhoria possível: usar o pôster nos cartões verticais.

## RAM (home parada, 4 abas pré-carregadas)

790 MB livres, 1.138 MB usados. Home: 207 MB (eram 190 MB com uma lista só, e 237 MB no launcher do Google).

Código: fork `PedroHaoTavares/free-my-tv`, commit `2f0110a`.
