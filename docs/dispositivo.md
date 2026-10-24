# Perfil da TV (coletado via ADB em 24/10/2026)

Coleta feita **só com comandos de leitura** (`getprop`, `pm list packages`, `dumpsys meminfo`, `settings get`). Nada foi alterado.
IP, número de série e nome do aparelho ficam fora do repositório, porque ele é público.

## Identificação

| Item | Valor | Origem |
|---|---|---|
| Marca / fabricante | TCL | `ro.product.brand` |
| Codinome do produto | `G07_4K_GB_NF` (device `G07`) | `ro.product.name` |
| Modelo comercial | Não exposto (`ro.product.model` = "Smart TV"). Confirmar na etiqueta. | `ro.product.model` |
| Sistema | **Google TV** (GTVS, launcher `launcherx`) | `ro.tcl.product_type` = GTV, `ro.com.google.gmsversion` |
| Android | **11** (API 30), build `RP1A.200622.001` | `ro.build.version.*` |
| Patch de segurança | 05/12/2024 | `ro.build.version.security_patch` |
| Chip (SoC) | **Realtek RTD2851A**, 4 núcleos ARMv7 | `ro.board.platform`, `/proc/cpuinfo` |
| Arquitetura | **32 bits** (`armeabi-v7a`) | `ro.product.cpu.abi` |
| RAM | **2 GB** (~750 MB disponíveis com a TV parada na home) | `/proc/meminfo` |
| Armazenamento interno | 8,5 GB, 54% usado (3,9 GB livres) | `df -h` |
| Resolução da interface | 1920x1080, densidade 320 (o painel é 4K; a UI roda em 1080p) | `wm size`, `wm density` |
| Bootloader | **Travado** (`flash.locked=1`, verified boot `green`). O desbloqueio OEM é *permitido* (`oem_unlock_allowed=1`), mas não foi feito. | `getprop` |

## Estado atual

| Item | Valor |
|---|---|
| Launcher padrão | `com.google.android.apps.tv.launcherx` (Google TV) |
| Escalas de animação | janela 1.0 / transição 1.0 / animator padrão |
| Limite de processos em cache | 32 |
| Pacotes instalados | 134 no total, 13 instalados pelo usuário |
| Já desativados | `com.tcl.smartalexa`, `au.com.stan.and`, `tv.wuaki.apptv` |

## Quem mais consome RAM (TV parada na home)

| Processo | RAM (PSS) | Observação |
|---|---|---|
| `com.google.android.apps.tv.launcherx` | **~244 MB** | Launcher do Google TV. Maior consumidor, ~12% da RAM total. |
| `system` | ~123 MB | Sistema, intocável. |
| `com.google.android.gms` | ~76 MB | Google Play Services. Necessário para Play Store e login. |
| `com.android.tv.settings` | ~71 MB | Estava aberto durante a coleta. |
| `com.android.vending` (+ background) | ~87 MB | Play Store. |
| `com.android.systemui` | ~57 MB | Sistema. |
| `com.tcl.tv` | ~44 MB | Entradas/sintonizador TCL. |
| `com.google.android.katniss` | ~37 MB | Assistente / busca por voz. |
| `com.google.android.apps.tv.launcherx:coreservices` | ~36 MB | Serviço do launcher. |
| `com.tcl.ttvs` | ~33 MB | Serviço TCL. |
| `com.google.android.videos` | ~33 MB | Google TV / Play Filmes. |
| `com.google.android.apps.mediashell` | ~31 MB | Chromecast embutido. |
| `com.amazon.amazonvideo.livingroom` | ~20 MB | Prime Video rodando em segundo plano. |
| `com.tcl.guard` | ~19 MB | "Limpador"/guarda TCL. |

## Leitura inicial

1. **Hardware fraco:** 2 GB de RAM e CPU de 32 bits. É a classe de TV em que debloat e launcher leve fazem diferença perceptível.
2. **O launcher do Google TV é o maior alvo.** Trocar por um launcher leve (pronto ou próprio) tende a liberar mais memória que qualquer outra medida isolada.
3. **Apps de 32 bits:** qualquer app próprio precisa gerar APK com `armeabi-v7a` (ou ser só Kotlin/Java, sem código nativo).
4. **Root não é caminho realista:** bootloader travado, Realtek e sem TWRP conhecido para este codinome (ver `docs/pesquisa/otimizacoes.md`).

Listas brutas em `docs/coleta/`.
