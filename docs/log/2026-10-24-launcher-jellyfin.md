# 24/10/2026: reboot, Chromecast/T-Cast e launcher com Jellyfin

## 1. Reboot controlado (risco do `waterfall.overseas`)

`adb reboot` às 16:23. A TV ligou normalmente, **sem travar no logo**, e abriu direto no Free My TV (`uptime` confirmou o reinício). O relato de bootloop com `com.tcl.waterfall.overseas` desativado **não se repete nesta TV**.

## 2. Chromecast e T-Cast desativados (decisão de 24/10)

| Pacote | Comando | Reverter |
|---|---|---|
| `com.google.android.apps.mediashell` (Chromecast embutido) | `pm disable-user --user 0` | `pm enable com.google.android.apps.mediashell` |
| `com.tcl.MultiScreenInteraction_TV` (T-Cast) | `pm disable-user --user 0` | `pm enable com.tcl.MultiScreenInteraction_TV` |

**Atenção:** esses dois foram desativados **por nós**, não pelo free-my-tv. O botão "Original" do app **não os reativa**. Para isso, use os comandos acima.

## 3. Launcher personalizado (fork `PedroHaoTavares/free-my-tv`, commit `9fba426`)

| Mudança | Como |
|---|---|
| "Continuar assistindo" do Jellyfin no lugar das sugestões do TMDB | Nova classe `Jellyfin.java`. Faz login com usuário e senha, lê `/Users/{id}/Items/Resume`, mostra uma entrada por série e baixa a imagem de fundo do servidor. Atualiza a cada 1 min. |
| OK no cartão abre o episódio | Intent explícita para `org.jellyfin.androidtv/.ui.startup.StartupActivity` com o extra `ItemId`. O genérico `ACTION_VIEW + BROWSABLE` do projeto não serve para o Jellyfin. |
| Jellyfin em 1º lugar na prateleira | Entrada no `SHELF` (`HomeApps.java`). Com o limite de 8 apps, o Spotify saiu da prateleira (continua em "Todos os apps"). |
| Título "Continuar assistindo" e aba "Tudo" | `HomeView.java` + traduções pt/en |
| Dólar em vez de cripto | `coins=USDT` + `currency=BRL`. O símbolo do Tether (₮) foi trocado por "$". **É a cotação USDT/BRL da Binance**, que acompanha o dólar de perto, mas não é o dólar comercial oficial. |
| HTTP só para o Jellyfin | O app bloqueia HTTP sem criptografia. O `build.sh` gera um `network_security_config.xml` que libera **só o host do Jellyfin** e fica fora do git. |

### Verificado na TV

- Tela inicial: [`../img/2026-10-24-home-jellyfin.png`](../img/2026-10-24-home-jellyfin.png)
- OK no cartão de The Walking Dead → Jellyfin abre no T7:E1 com "Retomar de 2:25" em foco: [`../img/2026-10-24-ok-abre-episodio.png`](../img/2026-10-24-ok-abre-episodio.png)

### RAM

| Situação | RAM livre | RAM usada | Tela inicial |
|---|---|---|---|
| Google TV original | 716 MB | 1.266 MB | 237 MB (`launcherx` + `coreservices`) |
| Free My TV sem imagens | 869 MB | 1.160 MB | 119 MB |
| **Free My TV + Jellyfin (atual)** | **790 MB** | **1.127 MB** | **190 MB** |

As imagens de fundo em 1920×1080 custam ~70 MB na home. Possível otimização futura: baixar as imagens dos cartões em resolução menor.
