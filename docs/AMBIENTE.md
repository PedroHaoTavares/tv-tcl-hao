# Ambiente de desenvolvimento (PC Windows)

Montado em 24/10/2026.

## Ferramentas instaladas

| Ferramenta | Como | Onde |
|---|---|---|
| Android Platform-Tools 37.0.1 (`adb`) | `winget install Google.PlatformTools` | `%LOCALAPPDATA%\Microsoft\WinGet\Packages\Google.PlatformTools_*\platform-tools\` |
| Eclipse Temurin JDK 17.0.20 | `winget install EclipseAdoptium.Temurin.17.JDK` | `C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot` |
| Android CLI (Google) | `winget install Google.AndroidCLI` | `%LOCALAPPDATA%\Microsoft\WinGet\Packages\Google.AndroidCLI_*\android.exe` |
| Android SDK: build-tools 34.0.0 + platform android-34 | `android --no-metrics --sdk=<SDK> sdk install "build-tools;34.0.0" "platforms;android-34"` | `%LOCALAPPDATA%\Android\Sdk` |

Variáveis de ambiente do usuário:

```
JAVA_HOME    = C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot
ANDROID_HOME = C:\Users\pphen\AppData\Local\Android\Sdk
```

## Código do launcher

- Fork: [`PedroHaoTavares/free-my-tv`](https://github.com/PedroHaoTavares/free-my-tv) (upstream `simbasjungles/free-my-tv`, MIT)
- Cópia local: `C:\Users\pphen\free-my-tv`
- Por que fork e não pasta neste repo: dá para puxar as correções do autor (`git fetch upstream`). Este repo (`tv-tcl-hao`) guarda a documentação e o histórico do que foi feito na TV.

## Build

No Git Bash:

```bash
cd /c/Users/pphen/free-my-tv
export JAVA_HOME="C:/Program Files/Eclipse Adoptium/jdk-17.0.20.101-hotspot"
export ANDROID_HOME="C:/Users/pphen/AppData/Local/Android/Sdk"
./build.sh        # gera FreeMyTV.apk (~1,3 MB)
```

**Pegadinha resolvida:** no Git Bash, o `ANDROID_HOME` precisa estar no formato `C:/...`, e não `/c/...`. Com `/c/...`, o `javac` recebe o bootclasspath sem conversão e falha com `Unable to find method metafactory` (os stubs de lambda não são encontrados).

## Arquivos que NÃO vão para o git (já no `.gitignore` do fork)

| Arquivo | Por quê |
|---|---|
| `config.properties` | Configuração local e chaves (Jellyfin). |
| `freemytv.keystore` | **Chave de assinatura. Sem ela, não dá para atualizar o app instalado na TV**: seria preciso desinstalar e reinstalar. Guardar um backup. |
| `FreeMyTV.apk`, `out/` | Saída do build. |

## Configuração usada (`config.properties`)

| Chave | Valor | Motivo |
|---|---|---|
| `latitude` / `longitude` / `city` | -7.9408 / -34.8728 / Paulista | Clima e pôr do sol (coordenadas aproximadas do centro de Paulista-PE) |
| `language` | pt | |
| `tmdb_region` | BR | Irrelevante: o TMDB não será usado |
| `tmdb_key` | vazio | Decisão: sem TMDB, entra o Jellyfin |
| `temperature` / `currency` | C / BRL | |
| `disable_assistant` | true | Decisão: voz desativada |
| `show_crypto` | true (padrão) | Pendente: confirmar com o Sr. Hao |

## Pré-requisitos verificados na TV (só leitura, 24/10/2026)

| Item | Resultado | Consequência |
|---|---|---|
| Conexão | Wi-Fi (sem cabo) | |
| Redes Wi-Fi salvas pelo `setupwraith` | **Sim** (4 ocorrências em `dumpsys wifi`) | Na troca, o free-my-tv vai pedir para **esquecer a rede e conectar de novo** em Configurações. Se não fizer isso, o Android apaga a rede ao desativar o `setupwraith`. |
| Operação de autostart da TCL | `APP_AUTO_START` existe (`AUTO_START` não) | Rodar `appops set io.freemytv APP_AUTO_START allow` após instalar |
