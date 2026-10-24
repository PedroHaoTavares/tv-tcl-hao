# Otimizações em TV Android / Google TV TCL via ADB (Sem Root)

> **Escopo e Regras da Pesquisa:** Levantamento realizado em 24/10/2026 exclusivamente por meio do **Portal Aurora** (`maestri portal navigate/text/snapshot`), seguindo a nota `"TV TCL - Brief"`. Esta etapa é **estritamente analítica**: nenhum comando de modificação foi executado na TV do Sr. Hao e nenhum commit foi realizado. Todas as afirmações possuem fonte documentada; pontos sem evidência pública encontrada estão explicitamente declarados como tal.

---

## Resumo Geral — Tabela de Otimizações e Condições

| Otimização | Benefício | Requisito | Risco | Reversível? | Fonte |
|---|---|---|---|---|---|
| **Inventário prévio de pacotes e RAM (`pm list packages -e/-d/-3/-f -u` e `dumpsys meminfo`)** | Cria ponto de restauração exato e linha de base de consumo (`Used PSS`) antes de qualquer alteração | ADB conectado (USB ou Wi-Fi na porta 5555) | Nenhum (somente leitura) | N/A (leitura) | [ohernandezdev/android-tv-tune (`inventory.sh`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/scripts/inventory.sh) |
| **Desativar telemetria e analíticos TCL (`com.tcl.ttvs`, `com.tcl.logkit`, `com.tcl.bi`)** | Libera ~33 MB a ~43 MB de RAM residente (`TTVSCore` consome ~33 MB na TV do Sr. Hao) e interrompe monitoramento de boot/instalação de pacotes | `adb shell pm disable-user --user 0 <pacote>` | Baixo (serviços de telemetria/diagnóstico sem impacto na reprodução) | **Sim** (`pm enable <pacote>`) | [ohernandezdev/android-tv-tune (`debloat.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/debloat.md), [vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [docs/dispositivo.md](../dispositivo.md) |
| **Desativar guarda/antivírus TCL (`com.tcl.guard`, `com.tcl.tvweishi`)** | Libera ~19 MB de RAM na TV do Sr. Hao, reduz uso de CPU em background e evita que o Safety Guard encerre serviços de acessibilidade de launchers/remapeadores | `adb shell pm disable-user --user 0 com.tcl.guard` | Baixo (perde apenas a interface "Safety Guard" da TCL) | **Sim** (`pm enable` ou `pm install-existing`) | [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [tvQuickActions Docs](https://tvdevinfo.com/tvquickactions/troubleshooting/) |
| **Desativar motor de anúncios/canais e vitrines TCL (`com.tcl.waterfall.overseas`, `com.tcl.channelplus`, `com.tcl.messagebox`, `com.tcl.partnercustomizer`, `com.tcl.dashboard`, `com.tcl.tv.tclhome_passive`)** | Remove vitrines patrocinadas, notificações promocionais e camadas OEM em segundo plano | `adb shell pm disable-user --user 0 <pacote>` (aplicar em lotes pequenos) | **Baixo a Médio:** seguro na maioria dos modelos (`G10`, `C645K`, `65C7K`), mas há relato isolado de bootloop ao desativar `com.tcl.waterfall.overseas` no modelo europeu `55P635` (`R51AT01`). Testar com ADB de rede ativo | **Sim** (`pm enable <pacote>` via ADB ou Factory Reset) | [ohernandezdev/android-tv-tune (`vendor-tcl.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [Gist 0x4f53 (comentário BassMyAss)](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [Gist llaumgui](https://gist.github.com/llaumgui/e39fdbafc920edacf42fdff6d8ab3a0a) |
| **Desativar modos de loja, fábrica e apps residuais TCL (`com.tcl.esticker`, `com.tcl.factory.view`, `com.tcl.hotelmenu`, `com.tcl.t_solo`, `com.tcl.usercenter`, `com.tcl.useragreement`, `com.tcl.notereminder`, `com.tcl.gallery`)** | Elimina processos de demonstração de loja (e-Sticker), menus de hotel e aplicativos utilitários redundantes | `adb shell pm disable-user --user 0 <pacote>` | Baixo (não afetam o core do Android TV/Google TV) | **Sim** (`pm enable <pacote>`) | [imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [ohernandezdev/android-tv-tune (`vendor-tcl.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list) |
| **Desativar extras Google e sistema ociosos (`com.google.android.videos`, `com.google.android.play.games`, `com.google.android.youtube.tvmusic`, `com.google.android.apps.tv.dreamx`, `com.google.android.feedback`, `com.android.printspooler`, `com.android.providers.calendar`, `com.android.providers.contacts`)** | Libera RAM imediata (`com.google.android.videos` consome ~33 MB na TV do Sr. Hao; `dreamx` ~40 MB em referência) e reduz sincronizações em background | `adb shell pm disable-user --user 0 <pacote>` | Baixo (se usar o protetor de tela Ambient Mode, mantenha `dreamx`; se alugar filmes na Google TV Store, mantenha `videos`) | **Sim** (`pm enable <pacote>`) | [ohernandezdev/android-tv-tune (`google-tv.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md), [docs/dispositivo.md](../dispositivo.md) |
| **Desativar Google Assistente / busca por voz (`com.google.android.katniss`, `com.google.android.tts`)** | Libera ~37 MB (na TV do Sr. Hao) a ~90 MB de RAM | `adb shell pm disable-user --user 0 <pacote>` | **Médio (Condicional):** quebra a busca por voz, o botão de microfone do controle e o leitor de tela TalkBack | **Sim** (`pm enable <pacote>`) | [ohernandezdev/android-tv-tune (`google-tv.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [docs/dispositivo.md](../dispositivo.md) |
| **Desativar Chromecast embutido (`com.google.android.apps.mediashell`) e T-Cast (`com.tcl.MultiScreenInteraction_TV`, `com.tcl.miracast`)** | Libera ~31 MB de RAM (`mediashell` na TV do Sr. Hao) e remove serviços de recepção de mídia | `adb shell pm disable-user --user 0 <pacote>` | **Médio (Condicional):** quebra envio de vídeo do celular/PC via Google Cast (`mediashell`), app TCL T-Cast (`MultiScreenInteraction_TV`) e espelhamento Miracast (`com.tcl.miracast`) | **Sim** (`pm enable <pacote>`) | [ohernandezdev/android-tv-tune (`google-tv.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [robinlennox/tcl-debloat](https://github.com/robinlennox/tcl-debloat/blob/main/tcl.sh), [docs/dispositivo.md](../dispositivo.md) |
| **Reduzir escalas de animação (`window_animation_scale`, `transition_animation_scale`, `animator_duration_scale` para `0.5` ou `0`)** | Melhora imediata na sensação de fluidez ao navegar pelos menus com D-pad; persiste após reinicialização | `adb shell settings put global <escala> 0.5` | Muito baixo (usar `0` em `animator_duration_scale` pode causar *flickering* na seleção de cards ou esconder spinners de carregamento; `0.5` evita o problema) | **Sim** (voltar valor para `1.0`) | [ohernandezdev/android-tv-tune (`tune-animations.sh`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/scripts/tune-animations.sh), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md), [XDA Animations Guide](https://xdaforums.com/t/animations-set-scale-set-to-25.4365695/) |
| **Limitar processos em segundo plano / cache (`background_process_limit` ou `device_config max_cached_processes`)** | Reduz o limite padrão (atualmente `32` na TV do Sr. Hao) para `2` ou `4`, evitando acúmulo de apps de streaming em cache na RAM de 2 GB | Via UI (Opções do Desenvolvedor) ou ADB (`settings put global` / `device_config put activity_manager`) | Baixo (apps fecham ao alternar entre mais de 2–4 aplicativos; alteração pela UI não persiste no reboot sem `device_config set_sync_disabled_for_tests persistent`) | **Sim** (restaurar padrão `32` e `sync_disabled` para `none`) | [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md), [XDA Limit Background Processes](https://xdaforums.com/t/closed-limit-background-processes-via-adb.4607175/) |
| **Limpeza global de cache (`adb shell pm trim-caches 512G`) e parada de apps (`am force-stop` / `am kill-all`)** | Libera espaço de armazenamento interno e memória ocupada por apps de streaming em segundo plano sem apagar logins ou dados do usuário | Conexão ADB autorizada | Muito baixo (o próximo início de cada app pode levar alguns segundos a mais para reconstruir cache) | **Sim** (automático conforme os apps voltam a rodar) | [ByteScope (`pm trim-caches`)](https://bytescope.dev/en/web-adb/commands/pm-trim-caches/), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md) |
| **Trocar launcher via Acessibilidade ("Override current launcher" no Projectivy + `APP_AUTO_START allow`)** | Substitui a tela inicial pesada e com anúncios por interface limpa mantendo o botão de Configurações e recursos do sistema | Instalar Projectivy na memória interna, ativar Acessibilidade e permitir início automático (`appops set ... APP_AUTO_START allow`) | Baixo a Médio: `launcherx` ainda pode residir em memória; acessibilidade na TCL pode causar bug de volume pulando de 2 em 2 ou conflito na busca por voz (issue #295) | **Sim** (desativar serviço de acessibilidade ou desinstalar launcher) | [XDA Projectivy Launcher](https://xdaforums.com/t/app-android-tv-projectivy-launcher.4436549/), [Projectivy Issue #295](https://github.com/spocky/miproja1/issues/295), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [XDA TCL Guide](https://xdaforums.com/t/guides-tcl-32s6200-t221t01-platform-update-firmware-optimize-debloat.4688670/) |
| **Travar launcher alternativo desativando `launcherx` + `setupwraith` via ADB** | Libera ~280 MB de RAM na TV do Sr. Hao (`launcherx` ~244 MB + `coreservices` ~36 MB) e impede que o Google TV volte sozinho | Ter launcher alternativo já instalado/testado na memória interna; desativar **ambos** (`com.google.android.apps.tv.launcherx` e `com.google.android.tungsten.setupwraith`) | **Alto se feito fora de ordem:** desativar antes de ter outro launcher causa tela preta; desativar só `launcherx` faz `setupwraith` roubar o botão HOME; em TVs TCL Google TV pode desativar o botão físico de "Configurações" do controle e afetar login do YouTube/voz | **Sim** (`pm enable` nos dois pacotes via ADB ou Factory Reset) | [ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md), [XDA TCL 2023 (post #127)](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-7), [XDA Projectivy Launcher](https://xdaforums.com/t/app-android-tv-projectivy-launcher.4436549/) |
| **Desbloqueio de Bootloader (`fastboot flashing unlock`) / Root (Magisk) / TWRP** | Permitiria modificar partições de sistema, mas **não traz ganho prático** sobre o debloat ADB para uso em TV | Apenas documentado em chassis MediaTek `T653T01` (ex.: C855) ou via firmware de teste `userdebug`; **sem TWRP ou método documentado para o Realtek `RTD2851A` (`G07`)** | **Crítico:** apaga todos os dados, **rebaixa Widevine L1 para L3** (Netflix/Prime limitados a SD/720p ou bloqueados), invalida garantia e tem alto risco de *brick* (exigindo adaptador serial UART 3.3V no U-Boot) | **Parcial / Não garantido** (chaves Widevine L1 podem não voltar mesmo reflasheando `.pkg`) | [XDA Rooting TCL T653T01](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/), [XDA Unbricking TCL TVs via UART](https://xdaforums.com/t/guide-unbricking-tcl-tvs.4776759/), [XDA About Root Rights on TCL TVs](https://xdaforums.com/t/about-root-rights-on-tcl-tvs.4690567/) |

---

## 1. Como identificar Modelo, Plataforma, SoC e Versão do Android (e por que isso muda o que é possível)

### 1.1. Comandos e caminhos de identificação
Existem duas formas complementares de identificar a plataforma exata de uma TV TCL:

1. **Pela interface da TV (Menu Sobre / Informações do Produto):**
   - Caminho: `Configurações > Sistema (ou Preferências do Dispositivo) > Sobre > Informações do Produto`.
   - O campo de versão de software segue o padrão `V8-<PLATAFORMA>-LF1V<VERSÃO>` (ex.: `V8-R51MT02-LF1V392`, `V8-R51MT05-LF1V701`, `V8-R51AT01-LF1V107`, `V8-T653T01-LF1V135`). ([Fonte: XDA TCL 2023 p. 55](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-55), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide))
   - A segunda parte do código identifica o **SoC (processador) + variante de chassi (`TXX`)**:
     - `R851` = Realtek RTD2851 (ex.: `R851T02` = Android TV 9/11).
     - `R51M` = Realtek RTD2851M (`R51MT02` = Android TV 11; `R51MT05` = Google TV 11).
     - `R51A` = Realtek RTD2851A (ex.: `R51AT01` = Google TV 11).
     - `T615` = MediaTek MT9615; `T632` = MediaTek MT9632; `T653` = MediaTek MT9653 / Pentonic 700 (ex.: `T653T01`). ([Fonte: XDA TCL 2023 p. 55](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-55), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide))
   - Mesmo dentro de uma mesma linha comercial da TCL, diagonais diferentes (ex.: 43"/50" vs 55"/65"/75") frequentemente utilizam placas e processadores distintos. ([Fonte: XDA TCL 2023 p. 55](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-55))

2. **Via ADB (somente leitura):**
   - SoC e plataforma: `adb shell getprop ro.board.platform` e `adb shell cat /proc/cpuinfo`
   - Arquitetura (ABI suportada): `adb shell getprop ro.product.cpu.abi` e `adb shell getprop ro.product.cpu.abilist` ([Fonte: ohernandezdev/android-tv-tune (`sideload.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/sideload.md))
   - Versão do Android e API: `adb shell getprop ro.build.version.release` e `adb shell getprop ro.build.version.sdk`
   - Tipo de sistema (Android TV vs Google TV): `adb shell getprop ro.tcl.product_type` e verificação do pacote de launcher ativo (`com.google.android.tvlauncher` no Android TV clássico vs `com.google.android.apps.tv.launcherx` no Google TV). ([Fonte: docs/dispositivo.md](../dispositivo.md), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide))

### 1.2. Perfil confirmado da TV do Sr. Hao e por que isso muda o que é possível
Conforme o levantamento de leitura já salvo em [`docs/dispositivo.md`](../dispositivo.md), a TV do Sr. Hao possui:
- **Codinome:** `G07_4K_GB_NF` (device `G07`)
- **Sistema:** **Google TV** (`ro.tcl.product_type = GTV`), rodando **Android 11** (API 30, patch 05/12/2024)
- **SoC e Arquitetura:** **Realtek RTD2851A** (4 núcleos ARMv7) em modo **32 bits (`armeabi-v7a`)**
- **Memória e Armazenamento:** **2 GB de RAM** (~750 MB livres em repouso) e 8,5 GB de armazenamento interno (3,9 GB livres)

**Impacto direto nas otimizações possíveis:**
1. **Google TV (`launcherx`) vs Android TV (`tvlauncher`):**
   - Em **Android TV**, o launcher padrão é `com.google.android.tvlauncher` acompanhado de `com.google.android.tvrecommendations` e `com.google.android.leanbacklauncher.recommendations`. ([Fonte: Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide))
   - Em **Google TV** (caso da TV do Sr. Hao), o launcher é `com.google.android.apps.tv.launcherx` (consumindo sozinho **~244 MB + ~36 MB do `launcherx:coreservices` = ~280 MB de PSS** na TV do Sr. Hao). Além disso, no Google TV existe o pacote `com.google.android.tungsten.setupwraith`: se apenas o `launcherx` for desativado, a atividade `setupwraith/.RecoveryActivity` assume o botão `HOME` no lugar do launcher alternativo. ([Fonte: ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md), [docs/dispositivo.md](../dispositivo.md))
2. **Arquitetura 32 bits (`armeabi-v7a`):**
   - Qualquer APK instalado via *sideload* (`adb install`) precisa ser compatível com `armeabi-v7a` (32-bit ARM) ou universal. Instalar um APK compilado apenas para `arm64-v8a` resulta em falha na instalação ou fechamento imediato (*crash*) ao abrir o aplicativo. ([Fonte: ohernandezdev/android-tv-tune (`sideload.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/sideload.md))
3. **SoC Realtek (`RTD2851A`) vs MediaTek (`MT9653`):**
   - Os serviços de hardware e drivers do SoC (`com.realtek.*`, `com.tvos`) jamais devem ser removidos indiscriminadamente. ([Fonte: ohernandezdev/android-tv-tune (`never-disable.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/never-disable.list))
   - Em caso de atualização/downgrade manual de firmware via USB, um firmware de plataforma MediaTek (`T653T01`) ou mesmo de outra variante Realtek incompatível nunca deve ser aplicado sobre o chassi incorreto. ([Fonte: XDA TCL 2023 p. 55](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-55))

---

## 2. Debloat via ADB: Pacotes Seguros, Pacotes que Quebram Funções e Reversão

### 2.1. `pm disable-user` vs `pm uninstall --user 0` e como reverter
Sem root, a partição `/system` (bem como `/product` e `/vendor`) é somente leitura. O gerenciamento de pacotes para o usuário principal (`--user 0`) oferece dois comandos:

- **Desativar (Recomendado pela comunidade técnica):**
  ```bash
  adb shell pm disable-user --user 0 <nome.do.pacote>
  ```
  Esconde o pacote do sistema, impede que receba *broadcasts* (como `BOOT_COMPLETED`) e que inicie no boot, produzindo **exatamente o mesmo ganho de RAM e CPU** que a desinstalação, porém com reversão simples e imediata:
  ```bash
  adb shell pm enable <nome.do.pacote>
  ```
  ([Fonte: ohernandezdev/android-tv-tune (`debloat.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/debloat.md))

- **Desinstalar para o usuário 0:**
  ```bash
  adb shell pm uninstall -k --user 0 <nome.do.pacote>
  ```
  Remove o pacote para o usuário `0` (mantendo o APK original na partição de sistema). Para reverter sem reset de fábrica, utiliza-se:
  ```bash
  adb shell cmd package install-existing <nome.do.pacote>
  # ou equivalente:
  adb shell pm install-existing --user 0 <nome.do.pacote>
  ```
  *Nota:* Alguns modelos TCL também criam um perfil secundário (`--user 10`); scripts como `imaginator/TCL-TV-Debloat` iteram nos usuários `0` e `10`, mas o guia `ohernandezdev/android-tv-tune` recomenda fortemente preferir `pm disable-user --user 0` por ser mais fácil e previsível de reverter. ([Fonte: imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [Gist llaumgui](https://gist.github.com/llaumgui/e39fdbafc920edacf42fdff6d8ab3a0a), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [ohernandezdev/android-tv-tune (`debloat.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/debloat.md))

---

### 2.2. Pacotes SEGUROS para desativar (cruzados com a coleta da TV do Sr. Hao)

Os pacotes abaixo **estão instalados e ativos na TV do Sr. Hao** (conforme [`docs/coleta/2026-10-24-pacotes.txt`](../coleta/2026-10-24-pacotes.txt)) e são apontados pelas fontes da comunidade como seguros para desativar:

#### A) Bloatware, Telemetria e Demonstração da TCL (Presentes na TV do Sr. Hao)
- `com.tcl.ttvs` (**~33 MB de RAM na TV do Sr. Hao**): "TTVSCore" — serviço de telemetria/analytics da TCL que escuta `BOOT_COMPLETED`, `PACKAGE_ADDED`, `PACKAGE_REMOVED` e `DATE_CHANGED`. ([Fonte: ohernandezdev/android-tv-tune (`debloat.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/debloat.md), [vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [docs/dispositivo.md](../dispositivo.md))
- `com.tcl.guard` (**~19 MB de RAM na TV do Sr. Hao**): "Safety Guard" — suposto otimizador/antivírus da TCL que consome CPU/RAM em background e derruba serviços de acessibilidade de terceiros. ([Fonte: mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [docs/dispositivo.md](../dispositivo.md))
- `com.tcl.waterfall.overseas`: motor de conteúdo patrocinado / "TCL Channel". É apontado como um dos maiores ganhos em TVs TCL Google TV, **porém exige cautela**: enquanto funciona normalmente na maioria dos modelos (`TCL G10`, `C645K`, `65C7K`, `BeyondTV`), um usuário (`BassMyAss`) relatou *bootloop* na tela da Google TV no modelo europeu `55P635` (`V8-R51AT01-LF1V107`). Portanto, se for desativá-lo, faça individualmente com `pm disable-user` mantendo o ADB de rede ativo. ([Fonte: ohernandezdev/android-tv-tune (`vendor-tcl.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e))
- `com.tcl.channelplus`: vitrine de canais gratuitos/parceiros da TCL. ([Fonte: imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list))
- `com.tcl.partnercustomizer`: camada de customização que força conteúdo de parceiros comerciais da TCL. ([Fonte: robinlennox/tcl-debloat](https://github.com/robinlennox/tcl-debloat/blob/main/tcl.sh), [imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat))
- `com.tcl.dashboard` e `com.tcl.tv.tclhome_passive`: painel/camada passiva de Smart Home da TCL. ([Fonte: mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [Gist llaumgui](https://gist.github.com/llaumgui/e39fdbafc920edacf42fdff6d8ab3a0a))
- `com.tcl.messagebox`: notificações push e mensagens promocionais da TCL. ([Fonte: vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat))
- `com.tcl.esticker`, `com.tcl.factory.view`, `com.tcl.hotelmenu`: adesivos de demonstração de loja (e-Sticker), visão de teste de fábrica e modo hotel. ([Fonte: imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list))
- `com.tcl.usercenter`, `com.tcl.useragreement`, `com.tcl.t_solo`, `com.tcl.suspension`, `com.tcl.notereminder`, `com.tcl.gallery`, `com.tcl.keyhelp`: conta TCL, termos de uso pós-setup, utilitários e galeria OEM. ([Fonte: vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [seun-novodev/android-tv-debloat-toolkit](https://github.com/seun-novodev/android-tv-debloat-toolkit/blob/main/run_toolkit.py))
- `com.tcl.ui_mediaCenter`: reprodutor USB básico da TCL (seguro desativar se já usa VLC ou Jellyfin). ([Fonte: imaginator/TCL-TV-Debloat](https://github.com/imaginator/TCL-TV-Debloat), [robinlennox/tcl-debloat](https://github.com/robinlennox/tcl-debloat/blob/main/tcl.sh))
- *Outros pacotes TCL citados nas listas da comunidade (não listados na coleta da G07 do Sr. Hao, mas relevantes para referência):* `com.tcl.bootadservice`, `com.tcl.logkit`, `com.tcl.bi`, `com.tcl.exhibit`, `com.tcl.overseasappshow`, `com.tcl.appmarket2`, `com.tcl.tvweishi`, `com.tcl.magiconnectfree`, `com.tcl.ocean.instructions`, `com.tcl.repairguide`, `com.tcl.gamebar`, `com.tcl.hearaid`, `com.tcl.interactive`. ([Fonte: vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e))

#### B) Extras Google e Serviços de Sistema Ociosos (Presentes na TV do Sr. Hao)
- `com.google.android.videos` (**~33 MB de RAM na TV do Sr. Hao**): loja de aluguel de filmes Google Play Filmes / Google TV Store. ([Fonte: ohernandezdev/android-tv-tune (`google-tv.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [docs/dispositivo.md](../dispositivo.md))
- `com.google.android.play.games` e `com.google.android.youtube.tvmusic`: Google Play Games e YouTube Music. ([Fonte: google-tv.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list))
- `com.google.android.apps.tv.dreamx` e `com.android.dreams.basic`: protetores de tela ("Ambient Mode" consome ~40 MB em referência; manter se desejar protetor de tela com fotos). ([Fonte: google-tv.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md))
- `com.google.android.feedback`, `com.google.android.partnersetup`, `com.google.android.onetimeinitializer`, `android.autoinstalls.config.google.gtvpai`, `com.google.android.apps.nbu.smartconnect.tv`, `com.google.android.sss.authbridge`: telemetria de feedback Google, configuração de parceiros e instalador automático de apps promovidos. ([Fonte: google-tv.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e))
- `com.google.android.syncadapters.calendar`, `com.android.providers.calendar`, `com.android.providers.contacts`, `com.android.providers.userdictionary`, `com.android.printspooler`, `com.android.statementservice`, `com.android.wallpaperbackup`, `com.android.sharedstoragebackup`, `com.android.backupconfirm`: sincronizadores de agenda, contatos e serviço de impressão (inúteis em uma TV). ([Fonte: devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md), [seun-novodev/android-tv-debloat-toolkit](https://github.com/seun-novodev/android-tv-debloat-toolkit/blob/main/run_toolkit.py))
- *Apps de streaming pré-instalados/opcionais não utilizados:* na TV do Sr. Hao, `com.amazon.amazonvideo.livingroom` está consumindo **~20 MB de RAM em segundo plano** mesmo com a TV parada na Home (se o Sr. Hao não usar Prime Video, pode ser desativado; o mesmo vale para `com.cbs.ca`, `tv.pluto.android`, etc.). `com.tcl.smartalexa`, `au.com.stan.and` e `tv.wuaki.apptv` já constam como desativados na TV. ([Fonte: docs/dispositivo.md](../dispositivo.md), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e))

---

### 2.3. Pacotes que QUEBRAM funções específicas ou causam Bootloop

| Função / Subsistema | Pacotes Envolvidos (Presentes na TV do Sr. Hao) | O que quebra se desativar / remover | Fonte |
|---|---|---|---|
| **Launcher / Botão HOME / Botão Configurações** | `com.google.android.apps.tv.launcherx` (~244 MB) + `:coreservices` (~36 MB), `com.google.android.tungsten.setupwraith` | Desativar `launcherx` sem outro launcher pronto resulta em **tela preta**. Desativar `launcherx` sem desativar `setupwraith` faz o `RecoveryActivity` roubar o botão HOME. Em TVs TCL Google TV (relato C845) e Chromecast com Google TV, desativar `launcherx` **inutiliza o botão físico de Configurações (engrenagem)** do controle remoto, afeta a busca por voz embutida e impede troca de conta no YouTube oficial. | [ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md), [XDA TCL 2023 p. 7 (#127)](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-7), [XDA Projectivy Launcher](https://xdaforums.com/t/app-android-tv-projectivy-launcher.4436549/) |
| **T-Cast / Espelhamento / Chromecast** | `com.tcl.MultiScreenInteraction_TV`, `com.tcl.miracast`, `com.google.android.apps.mediashell` (~31 MB) | `MultiScreenInteraction_TV` quebra o controle/espelhamento pelo app mobile T-Cast (MagiConnect). `com.tcl.miracast` quebra espelhamento Miracast. `mediashell` quebra o **Chromecast Built-in** (Google Cast). | [robinlennox/tcl-debloat](https://github.com/robinlennox/tcl-debloat/blob/main/tcl.sh), [ohernandezdev/android-tv-tune (`google-tv.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [seun-novodev (`run_toolkit.py`)](https://github.com/seun-novodev/android-tv-debloat-toolkit/blob/main/run_toolkit.py) |
| **Voz / Microfone do Controle / Acessibilidade** | `com.tcl.micmanager`, `com.google.android.katniss` (~37 MB), `com.google.android.tv.assistant`, `com.tcl.assistant`, `com.google.android.tts`, `com.google.android.marvin.talkback` | `com.tcl.micmanager` desativa o microfone físico do controle remoto TCL. `katniss` e `tv.assistant` quebram o Google Assistente e comandos de voz. `tts` e `talkback` quebram síntese de voz e leitura de tela. | [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [ohernandezdev/android-tv-tune (`google-tv.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list) |
| **Entradas HDMI/AV, Sintonizador e HDMI-CEC** | `com.tcl.tv` (~44 MB), `com.tcl.tvinput`, `com.android.providers.tv`, `com.android.tv.settings`, `com.tcl.settings` | `com.tcl.tv` e `com.tcl.tvinput` controlam a comutação de entradas HDMI/AV, sintonizador de TV aberta e integração de hardware. No Projectivy Launcher, a opção de substituir a view interna de TV não suporta **HDMI-CEC**, pois apenas os apps de sistema originais detêm permissão de CEC. | [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [never-disable.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/never-disable.list), [seun-novodev (`run_toolkit.py`)](https://github.com/seun-novodev/android-tv-debloat-toolkit/blob/main/run_toolkit.py), [XDA Projectivy Launcher](https://xdaforums.com/t/app-android-tv-projectivy-launcher.4436549/) |
| **Atualizações (Sistema e Controle Remoto)** | `com.android.dynsystem` (presente na G07); em outros modelos: `com.tcl.versionUpdateApp`, `com.tcl.rc.ota`, `com.tcl.externaldevice.update`, `com.realsil.ota` | Impedem atualizações OTA da TV (`versionUpdateApp` / `realsil.ota`) ou atualizações de firmware do controle remoto Bluetooth (`com.tcl.rc.ota`). | [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [robinlennox/tcl-debloat](https://github.com/robinlennox/tcl-debloat/blob/main/tcl.sh), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md) |
| **Localização Fused (BOOTLOOP COMPROVADO)** | `com.android.location.fused` | **NUNCA DESATIVAR:** causa *bootloop* travado na animação de inicialização da TV se a opção de Localização estiver ativada nas configurações do Android. | [Gist 0x4f53 (confirmado pelo autor e por BassMyAss)](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e) |
| **Controle Remoto, Teclado, UI e Google Core (PROIBIDOS)** | `com.google.android.tv.remote.service`, `com.tcl.autopair`, `com.google.android.inputmethod.latin`, `com.tcl.inputmethod.international`, `com.android.systemui`, `com.google.android.gms`, `com.google.android.gsf`, `com.android.vending`, `com.google.android.webview`, `com.tcl.android.webview`, `com.tvos`, `com.tcl.system.server`, `com.tcl.providers.config` | **NUNCA DESATIVAR:** perda total do controle remoto Bluetooth/IR, perda do teclado virtual, sumiço da barra de volume/interface (`systemui`), quebra de login/DRM em todos os apps de streaming (`gms`/`vending`) ou falha de drivers do SoC (`com.tvos` / `com.tcl.system.server`). | [ohernandezdev/android-tv-tune (`never-disable.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/never-disable.list), [seun-novodev (`run_toolkit.py`)](https://github.com/seun-novodev/android-tv-debloat-toolkit/blob/main/run_toolkit.py) |

---

## 3. Desempenho: Escalas de Animação, Limite de Processos, Cache e Telemetria/Anúncios

### 3.1. Escalas de animação
Na TV do Sr. Hao, as três escalas estão atualmente em `1.0` ([`docs/dispositivo.md`](../dispositivo.md)). Reduzi-las para `0.5` é apontado como o ganho de velocidade percebida mais barato e seguro, e **persiste após reiniciar a TV**:
```bash
adb shell settings put global window_animation_scale 0.5
adb shell settings put global transition_animation_scale 0.5
adb shell settings put global animator_duration_scale 0.5
```
- **Por que `0.5` em vez de `0`?** O guia `devcelio/clean-android-tv` alerta que zerar `animator_duration_scale` (`0`) pode gerar *flickering* (piscadas) ao mover o foco entre cards de apps na TV; manter em `0.5` elimina a lentidão sem causar artefatos visuais. ([Fonte: ohernandezdev/android-tv-tune (`tune-animations.sh`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/scripts/tune-animations.sh), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md))

### 3.2. Limite de processos em segundo plano / em cache
Na coleta da TV do Sr. Hao ([`docs/dispositivo.md`](../dispositivo.md)), o limite atual de processos em cache é **32** — valor alto para apenas 2 GB de RAM, permitindo que apps pesados (como Prime Video, Netflix, Play Store) permaneçam residentes.
- **Via Menu do Desenvolvedor ou `settings put global`:**
  - `Configurações > Sistema > Opções do desenvolvedor > Limite de processos em segundo plano` (definir para `2` ou `4`, ou `1` conforme `mojothemonkey2/TCL_Guide`), ou via ADB:
    ```bash
    adb shell settings put global background_process_limit 2
    ```
  - **Condição importante:** a alteração pelo Menu do Desenvolvedor **não persiste após reiniciar a TV**. ([Fonte: mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md), [XDA Limit Background Processes](https://xdaforums.com/t/closed-limit-background-processes-via-adb.4607175/))
- **Via `device_config` no Android 11+ (com persistência opcional):**
  - Discussão técnica no XDA para Android TV 11 documenta o ajuste direto no `activity_manager`:
    ```bash
    adb shell /system/bin/device_config put activity_manager max_cached_processes 4
    adb shell /system/bin/device_config put activity_manager max_phantom_processes 8
    # Para impedir que o Android resete o device_config no reboot:
    adb shell "/system/bin/device_config set_sync_disabled_for_tests persistent"
    ```
  - Para reverter ao comportamento original: `adb shell "/system/bin/device_config set_sync_disabled_for_tests none"` e `max_cached_processes 32`. ([Fonte: XDA Limit Background Processes](https://xdaforums.com/t/closed-limit-background-processes-via-adb.4607175/))

### 3.3. Limpeza de Cache e Memória via ADB
- **Limpar cache de todos os aplicativos de uma vez (`pm trim-caches`):**
  ```bash
  adb shell pm trim-caches 512G
  ```
  Solicita ao `StorageManager` do Android que apague arquivos temporários de cache de todos os apps até atingir o alvo (como `512G` é maior que o armazenamento da TV, ele limpa todo o cache descartável).
  - **Diferença crítica:** `pm trim-caches` **não apaga logins, contas nem configurações** dos aplicativos (ao contrário de `pm clear <pacote>`, que reseta o app para o estado de fábrica no Android 11).
  - **Como ler o retorno:** sucesso no `pm trim-caches` não imprime nada no terminal; para medir quantos MB foram liberados, deve-se rodar `adb shell df -h` ou `adb shell dumpsys diskstats` antes e depois. ([Fonte: ByteScope (`pm trim-caches`)](https://bytescope.dev/en/web-adb/commands/pm-trim-caches/))
- **Encerrar apps de streaming em background sob demanda:**
  ```bash
  adb shell am force-stop <pacote>
  adb shell am kill-all
  ```
  ([Fonte: devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md))
- **Como medir o ganho real de RAM após otimizar:**
  O projeto `ohernandezdev/android-tv-tune` alerta que olhar apenas para "Free RAM" engana, pois o kernel Android usa a RAM liberada como cache de disco (o que acelera a abertura de apps). As métricas corretas no `adb shell dumpsys meminfo` após reiniciar a TV são:
  1. Queda no **Used PSS** total;
  2. Aumento no **Cached Kernel**;
  3. Desaparecimento dos processos desativados (como `launcherx`, `com.tcl.ttvs`, `com.google.android.videos`, `com.tcl.guard`) do topo da lista `Total PSS by process`. ([Fonte: ohernandezdev/android-tv-tune (`README.md` e `debloat.md`)](https://github.com/ohernandezdev/android-tv-tune))

### 3.4. Desativar Telemetria, Anúncios e ACR da TCL
1. **Desativação via ADB dos pacotes de telemetria/anúncios presentes na TV do Sr. Hao:**
   - `com.tcl.ttvs` (telemetria `TTVSCore`, ~33 MB RAM), `com.tcl.waterfall.overseas` (motor de anúncios/canais TCL), `com.tcl.messagebox` (push promocional), `com.tcl.channelplus`, `com.tcl.partnercustomizer`, `com.tcl.esticker`, `com.tcl.factory.view`, `com.google.android.feedback`, `com.google.android.partnersetup`, `android.autoinstalls.config.google.gtvpai`. ([Fonte: vendor-tcl.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/vendor-tcl.list), [google-tv.list](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/google-tv.list), [docs/coleta/2026-10-24-pacotes.txt](../coleta/2026-10-24-pacotes.txt))
2. **Pacotes de ACR (Automatic Content Recognition — caso apareçam em atualizações):**
   - `tv.samba.ssm.ui`, `com.samba.tv`, `com.inscape.tv`, `com.alphonso.service` (capturam o que está na tela, inclusive entradas HDMI, para perfil publicitário; nenhum deles consta na coleta atual da TV do Sr. Hao). ([Fonte: ohernandezdev/android-tv-tune (`telemetry.list`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/data/telemetry.list), [docs/coleta/2026-10-24-pacotes.txt](../coleta/2026-10-24-pacotes.txt))
3. **DNS privado / bloqueador de anúncios e rastreadores:**
   - O guia de otimização TCL no XDA recomenda configurar um DNS com bloqueio de anúncios/telemetria (como NextDNS) nas configurações de rede da TV para bloquear requisições de anúncios na origem. ([Fonte: XDA TCL Guide](https://xdaforums.com/t/guides-tcl-32s6200-t221t01-platform-update-firmware-optimize-debloat.4688670/))

---

## 4. Trocar o Launcher Padrão (Projectivy, FLauncher, ATV Launcher) e Como Travar na TCL

### 4.1. Opções de Launchers na Comunidade
- **Projectivy Launcher (`com.spocky.projengmenu`):** o mais completo para TVs TCL; suporta atalhos diretos para entradas HDMI 1–3/AV, lista apps de celular (que não declaram `LEANBACK_LAUNCHER`), não exibe anúncios e possui função própria de **"Override current launcher"** via Serviço de Acessibilidade. ([Fonte: XDA Projectivy Launcher](https://xdaforums.com/t/app-android-tv-projectivy-launcher.4436549/), [ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md))
- **FLauncher (`me.efesser.flauncher`):** código aberto (Flutter), grade minimalista e muito leve, mas **não possui serviço de acessibilidade próprio** para interceptar o botão Home caso o launcher original continue ativo. ([Fonte: ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md), [devcelio/clean-android-tv](https://github.com/devcelio/clean-android-tv/blob/main/tv-optimization-pt.md))
- **ATV Launcher (`ca.dstudio.atvlauncher.free` / `.pro`):** focado em grade customizável e suporte a widgets (exige `adb shell appwidget grantbind --package ca.dstudio.atvlauncher.free --user 0` para widgets sem root), mas também exige desativar o `launcherx` ou usar remapeador de botões para assumir o botão Home. ([Fonte: XDA ATV Launcher Widget Binding](https://xdaforums.com/t/widget-binding-failure-atv-launcher.4210161/), [XDA TCL 2023 p. 7 (#127)](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-7))

### 4.2. Por que em Google TV / TCL o launcher padrão às vezes volta (ou dá tela preta)?
A documentação técnica levantada identifica **quatro causas exatas** para o launcher voltar ao Google TV ou falhar em TVs TCL:

1. **`cmd package set-home-activity` é uma armadilha silenciosa no Google TV:**
   Rodar `adb shell cmd package set-home-activity "com.spocky.projengmenu/.ui.home.MainActivity"` retorna `Success` no terminal, mas **não muda nada**: o Google TV continua resolvendo o botão HOME para o `com.google.android.apps.tv.launcherx`. ([Fonte: ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md))
2. **O assistente de configuração (`setupwraith`) rouba o HOME ou reativa o `launcherx`:**
   Quando você desativa apenas `com.google.android.apps.tv.launcherx`, o sistema reavalia quem responde ao intent `android.intent.category.HOME`. Na lista de prioridade do Google TV, `com.google.android.tungsten.setupwraith/.RecoveryActivity` tem prioridade **acima** do Projectivy/FLauncher (`com.spocky.projengmenu/.ui.home.MainActivity`), abrindo uma tela de recuperação/setup ou tela preta, ou reativando o launcher original. ([Fonte: ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md), [interface-personalizada.md](interface-personalizada.md))
3. **O TCL Safety Guard (`com.tcl.guard`) e o gerenciamento de energia matam o Serviço de Acessibilidade no reboot:**
   Quando o usuário mantém o `launcherx` ativo e usa o modo "Override current launcher" do Projectivy (ou apps como `tvQuickActions`) via Acessibilidade, o sistema da TCL desativa o serviço em segundo plano após reiniciar a TV porque o app **Safety Guard** bloqueia a permissão `Auto Launch` / `Startup` por padrão. ([Fonte: mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [tvQuickActions Docs](https://tvdevinfo.com/tvquickactions/troubleshooting/))
4. **Instalação em armazenamento externo:**
   Se o launcher ou remapeador for movido para um pendrive USB formatado como armazenamento interno (*adopted storage*), o Android TV desativa seu serviço de acessibilidade a cada reinicialização. O app precisa ficar no armazenamento interno da TV. ([Fonte: tvQuickActions Docs](https://tvdevinfo.com/tvquickactions/troubleshooting/))

### 4.3. Como travar o novo launcher na TCL (Dois métodos documentados e suas condições)

#### Método 1: Via Acessibilidade + Permissão de Auto-Start (Sem desativar o `launcherx`)
Recomendado se o Sr. Hao quiser **preservar 100% do botão físico de Configurações (engrenagem), busca por voz e troca de contas do YouTube**:
1. Instalar o Projectivy Launcher (`com.spocky.projengmenu`) no armazenamento interno.
2. Nas configurações do Projectivy, ativar **"Override current launcher"** e habilitar o **Serviço de Acessibilidade** (ou via ADB):
   ```bash
   adb shell settings put secure enabled_accessibility_services "com.spocky.projengmenu/com.spocky.projengmenu.services.ProjectivyAccessibilityService"
   adb shell settings put secure accessibility_enabled 1
   ```
   *Atenção ao bug de volume na TCL:* no guia do XDA (`Retrial`) e no `mojothemonkey2/TCL_Guide`, documenta-se que serviços de acessibilidade em TVs TCL podem fazer o volume pular de 2 em 2 ou travar ao segurar o botão; recomenda-se desativar as opções de interceptação de volume dentro das configurações de acessibilidade do Projectivy. ([Fonte: XDA TCL Guide](https://xdaforums.com/t/guides-tcl-32s6200-t221t01-platform-update-firmware-optimize-debloat.4688670/), [mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide))
3. **Travar para não perder no reboot:**
   - Conceder permissão de auto-inicialização via ADB:
     ```bash
     adb shell appops set com.spocky.projengmenu APP_AUTO_START allow
     ```
   - Desativar o `com.tcl.guard` (`adb shell pm disable-user --user 0 com.tcl.guard`) **ou** abrir o app Safety Guard e liberar manualmente `Auto Launch Permissions` e `Startup Permissions` para o launcher;
   - Ativar `Configurações > Sistema > Energia > Screenless Service` (se presente no menu da TCL). ([Fonte: mojothemonkey2/TCL_Guide](https://github.com/mojothemonkey2/TCL_Guide), [tvQuickActions Docs](https://tvdevinfo.com/tvquickactions/troubleshooting/))
- **Condição/Limitação do Método 1:** como o `com.google.android.apps.tv.launcherx` não foi desativado no gerenciador de pacotes, ele pode continuar carregado em segundo plano consumindo RAM, e há relato de conflito com a tela de resultados da busca por voz em versões anteriores ao Projectivy 4.64 ([Issue #295](https://github.com/spocky/miproja1/issues/295)).

#### Método 2: Desativar `launcherx` e `setupwraith` via ADB (Trava definitiva no nível do sistema e libera ~280 MB de RAM)
Recomendado se o objetivo principal for **liberar os ~280 MB de RAM (`launcherx` + `coreservices`)**:
1. **Ordem obrigatória (para nunca ficar em tela preta):**
   - Instalar o Projectivy (`com.spocky.projengmenu`) ou FLauncher (`me.efesser.flauncher`);
   - Abrir o novo launcher, concluir o tutorial inicial e conceder as permissões (caso o diálogo de permissão trave em loop no Google TV, conceder via ADB):
     ```bash
     adb shell pm grant com.spocky.projengmenu android.permission.READ_TV_LISTINGS
     adb shell pm grant com.spocky.projengmenu android.permission.READ_EXTERNAL_STORAGE
     ```
2. **Desativar o launcher original E o assistente `setupwraith`:**
   ```bash
   adb shell pm disable-user --user 0 com.google.android.apps.tv.launcherx
   adb shell pm disable-user --user 0 com.google.android.tungsten.setupwraith
   ```
3. **Verificar quem assumiu o botão HOME:**
   ```bash
   adb shell "cmd package query-activities -a android.intent.action.MAIN -c android.intent.category.HOME" | grep -E '^\s+name='
   ```
   O primeiro item listado deve ser a `MainActivity` do novo launcher. ([Fonte: ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md))
- **Condição/Limitação do Método 2:** em TVs TCL com Google TV (relato no modelo C845 no XDA), desativar `launcherx` + `setupwraith` faz o botão físico de **Configurações (engrenagem)** do controle remoto parar de responder (sendo necessário abrir as Configurações pelo ícone dentro do próprio Projectivy/FLauncher ou remapear o botão), além de desativar a busca por voz embutida no `launcherx`. Caso queira reverter:
  ```bash
  adb shell pm enable com.google.android.apps.tv.launcherx
  adb shell pm enable com.google.android.tungsten.setupwraith
  adb shell input keyevent KEYCODE_HOME
  ```
  ([Fonte: ohernandezdev/android-tv-tune (`launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/launcher.md), [XDA TCL 2023 p. 7 (#127)](https://xdaforums.com/t/tcl-2023-google-tvs-p745-p755-c645-c745-c755-c805-c845-c855-c955-x955.4576005/page-7))

---

## 5. O que o Desbloqueio OEM Realmente Permite em TCL (Bootloader, Root, TWRP e Riscos)

### 5.1. O que a chave "Desbloqueio OEM" ativada faz sozinha?
Na TV do Sr. Hao ([`docs/dispositivo.md`](../dispositivo.md)), a opção "Desbloqueio OEM" nas Opções do Desenvolvedor está ativada (`oem_unlock_allowed=1`), mas **o bootloader continua travado** (`flash.locked=1`, `verifiedbootstate=green`).
- Enquanto o comando `fastboot flashing unlock` não for executado via cabo USB no modo fastboot, **a chave "Desbloqueio OEM" sozinha não altera nada no sistema**: não dá permissão de root ao ADB, não apaga dados e **não altera o Widevine L1**. ([Fonte: docs/dispositivo.md](../dispositivo.md), [XDA Rooting TCL T653T01](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/))

### 5.2. Bootloader, Root e TWRP são viáveis em TVs TCL?
1. **Bootloader e Root via Magisk (`fastboot flashing unlock`):**
   - Na comunidade XDA, existe um guia recente (2026, por `whew-inc`) demonstrando desbloqueio de bootloader (`adb reboot bootloader` -> `fastboot flashing unlock`) e flash de `boot.img` patcheado com Magisk **especificamente para a plataforma MediaTek `T653T01`** (ex.: TCL C855). ([Fonte: XDA Rooting TCL T653T01](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/))
   - Mesmo nesse tópico, outros usuários relatam tela preta sem detecção USB fastboot pelo PC ao tentar entrar no bootloader em versões/modelos diferentes. ([Fonte: XDA Rooting TCL T653T01 posts #7 e #8](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/), [XDA Unlocking Bootloader TCL 43P7K](https://xdaforums.com/t/unlocking-the-bootloader-on-my-tcl-tv-tcl-43p7k.4785388/))
   - Outra forma documentada no XDA para obter root em TCL é instalar firmwares internos de teste/engenharia da TCL (`build type: userdebug / userroot / eng`, como `V8-R51MT05-LF1V996_Test_3`), nos quais o comando `adb root` já vem habilitado de fábrica para remontar `/product` e `/vendor` como leitura/escrita. ([Fonte: XDA About Root Rights on TCL TVs](https://xdaforums.com/t/about-root-rights-on-tcl-tvs.4690567/))
   - **Para a TV do Sr. Hao (Realtek `RTD2851A` / `G07_4K_GB_NF`):** **não foi encontrado** na pesquisa nenhum guia de root validado, arquivo `boot.img` público para patch nem firmware `userdebug` específico para o codinome `G07`.
2. **TWRP (Custom Recovery) e Custom ROMs:**
   - **Não encontrado / Inviável para TVs TCL atuais.** As únicas referências a TWRP para TVs TCL no XDA dizem respeito a aparelhos antigos com Android 8 Oreo (chassi `MT85` / TCL P60, oriundos do fórum russo 4pda e com links mortos). Não existe TWRP oficial ou não-oficial nem Custom ROMs para as plataformas TCL Realtek/MediaTek com Google TV. ([Fonte: XDA Root Old TCL TV Android 8 MT85](https://xdaforums.com/t/q-root-an-old-tcl-tv-android-8-mt85.4607559/), [XDA Is there any TCL Custom Firmware out there?](https://xdaforums.com/t/is-there-any-tcl-custom-firmware-out-there.4671697/))

### 5.3. Riscos reais de desbloquear o Bootloader / Root em TV TCL
O próprio autor do guia de root para TCL no XDA (`whew-inc`) documenta consequências severas que tornam o desbloqueio de bootloader **fortemente contraindicado** para uma TV de sala:

1. **Queda de Widevine L1 para L3 e perda de 4K/Full HD em Streaming (Netflix em SD/720p ou bloqueada):**
   > *"You will lose Netflix, Amazon, other streaming service high quality options. Meaning you will be limited to probably 720p, if they allow you to watch at all. I don't know if it can be restored by flashing a .pkg again. Widevine will be L3."*
   Ao desbloquear o bootloader (`fastboot flashing unlock`), a certificação criptográfica Widevine cai de **L1 para L3**, limitando Netflix, Prime Video e demais serviços pagos de streaming a resolução SD/720p (ou impedindo a reprodução), sem garantia de que reflashear o firmware `.pkg` original restaure o Widevine L1. ([Fonte: XDA Rooting TCL T653T01](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/))
2. **Perda total de dados (Wipe obrigatório):**
   O comando `fastboot flashing unlock` apaga integralmente os dados do usuário na TV. ([Fonte: XDA Rooting TCL T653T01](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/))
3. **Risco elevado de Brick exigindo adaptador UART 3.3V via hardware:**
   Se a imagem `boot.img` falhar ou corromper a inicialização, a TV pode não exibir vídeo nem entrar no menu de recovery pelo botão físico. O guia do XDA exige ter em mãos um **adaptador serial UART de 3.3V** conectado com placa *breakout* à porta HDMI/USB de serviço da TV (pinos RX=2, TX=14, GND=11 a 115200 baud) para interromper o **U-Boot** e rodar `mtkupgrade usb` ou `usb_partial_upgrade_to_emmc`. ([Fonte: XDA Guide Unbricking TCL TVs](https://xdaforums.com/t/guide-unbricking-tcl-tvs.4776759/))
4. **Garantia e segurança:**
   Enquanto o debloat via ADB (`pm disable-user`) não altera partições nem viola a garantia (*"Nothing is flashed, no bootloader is unlocked, no warranty is voided"*), desbloquear o bootloader exibe aviso de dispositivo desbloqueado no boot e invalida as garantias de integridade do software. ([Fonte: ohernandezdev/android-tv-tune (`README.md`)](https://github.com/ohernandezdev/android-tv-tune), [XDA Rooting TCL T653T01](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/))

---

## 6. Riscos Gerais (Sem Root) e Como Fazer Backup Antes de Otimizar

### 6.1. Riscos Gerais do Processo via ADB e Cuidados Operacionais
1. **Desativar pacotes em massa de uma só vez:**
   Desativar 40 ou 50 pacotes de uma vez dificulta identificar o culpado caso algum comportamento quebre. A recomendação documentada é desativar em pequenos lotes, verificando se a interface e o controle remoto continuam respondendo entre cada lote. ([Fonte: ohernandezdev/android-tv-tune (`debloat.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/debloat.md))
2. **Desligar o ADB antes de reiniciar e testar:**
   O efeito real da desativação de pacotes só ocorre após o `adb reboot` (pois pacotes desativados continuam em memória até reiniciar, e erros como o bootloop do `com.android.location.fused` ou a perda do HOME pelo `setupwraith` só se manifestam no boot). **Mantenha a depuração ADB ativa até reiniciar a TV e validar que o botão HOME, o controle remoto e os aplicativos estão funcionando.** ([Fonte: ohernandezdev/android-tv-tune (`debloat.md` e `launcher.md`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/docs/debloat.md))
3. **Conflito das Opções do Desenvolvedor / Depuração USB com pendrives nas portas USB:**
   Tanto o guia de TCL no XDA (`Retrial`) quanto relatos de usuários no tópico de FAQ da TCL apontam que manter a **Depuração USB** e as **Opções do Desenvolvedor** ativadas permanentemente pode impedir que a TV reconheça pendrives/HDs externos na porta USB (pois o controlador USB muda para modo *device/debug*). Além disso, a porta 5555 do ADB de rede não exige senha adicional para dispositivos na mesma LAN. Portanto, **após concluir todas as otimizações e validar a estabilidade após o reboot**, recomenda-se desativar a Depuração USB/Rede (reativando apenas quando precisar fazer manutenção). ([Fonte: XDA TCL Guide](https://xdaforums.com/t/guides-tcl-32s6200-t221t01-platform-update-firmware-optimize-debloat.4688670/), [XDA FAQ TCL TVs post #4](https://xdaforums.com/t/f-a-q-and-useful-guides-for-tcl-tvs-flashing-firmwares-etc.4481901/), [ohernandezdev/android-tv-tune (`README.md`)](https://github.com/ohernandezdev/android-tv-tune))
4. **Ter o arquivo de firmware (`Update.img` / `.pkg` / OTA `.zip`) do chassi salvo no PC:**
   Usuários experientes na comunidade TCL recomendam anotar a versão exata do firmware em `Sobre > Informações do Produto` antes de qualquer modificação, para que, na pior hipótese de um *soft-brick* em que não se consiga fazer Factory Reset pelo controle remoto, seja possível restaurar o firmware via pendrive FAT32 pressionando o botão físico de Power da TV por 5 segundos ao ligar na tomada. ([Fonte: Gist 0x4f53 (comentário BassMyAss)](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), [XDA TCL Guide](https://xdaforums.com/t/guides-tcl-32s6200-t221t01-platform-update-firmware-optimize-debloat.4688670/))

---

### 6.2. Como fazer Backup da Lista de Pacotes e Estado da TV Antes de Começar
Seguindo o protocolo do script [`inventory.sh`](https://github.com/ohernandezdev/android-tv-tune/blob/main/scripts/inventory.sh) do repositório `ohernandezdev/android-tv-tune` (que já inspirou os arquivos salvos em [`docs/coleta/`](../coleta/)), o backup completo de estado antes de qualquer alteração consiste em salvar no computador:

```bash
# 1. Lista de todos os pacotes atualmente ATIVOS (seu ponto de restauração principal)
adb shell pm list packages -e | sed 's/^package://' | tr -d '\r' | sort > original-enabled.txt

# 2. Lista de pacotes que já vieram DESATIVADOS de fábrica/anteriormente
adb shell pm list packages -d | sed 's/^package://' | tr -d '\r' | sort > already-disabled.txt

# 3. Lista de pacotes de TERCEIROS (instalados pelo usuário)
adb shell pm list packages -3 | sed 's/^package://' | tr -d '\r' | sort > third-party.txt

# 4. Todos os pacotes com caminho do APK no sistema (incluindo desinstalados com -u)
adb shell pm list packages -f -u | sed 's/^package://' | tr -d '\r' | sort > all-with-paths.txt

# 5. Linha de base de memória (RAM total/livre e Top 20 processos por PSS)
adb shell dumpsys meminfo > baseline-memory.txt
```
([Fonte: ohernandezdev/android-tv-tune (`inventory.sh`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/scripts/inventory.sh))

**Como restaurar integralmente o estado original a partir do backup (`restore.sh`):**
Se qualquer pacote foi desativado com `pm disable-user`, basta percorrer o arquivo `original-enabled.txt` (na TV do Sr. Hao, já salvo em [`docs/coleta/2026-10-24-pacotes.txt`](../coleta/2026-10-24-pacotes.txt)) reativando cada item:
```bash
while IFS= read -r pkg; do
  [ -n "$pkg" ] && adb shell pm enable "$pkg"
done < original-enabled.txt
adb reboot
```
E, caso algum pacote tenha sido removido com `pm uninstall --user 0`, substituindo `pm enable "$pkg"` por `adb shell cmd package install-existing "$pkg"`. ([Fonte: ohernandezdev/android-tv-tune (`restore.sh`)](https://github.com/ohernandezdev/android-tv-tune/blob/main/scripts/restore.sh), [Gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e))
