# Interface personalizada — TV TCL do Sr. Hao

Pesquisa em 24/10/2026. Nota “TV TCL - Brief” lida no Maestri. Consulta web exclusivamente por `maestri portal navigate/text`, no **Portal Ferrolho**. Fase: análise; nenhum comando ADB foi executado, nenhuma alteração na TV, nenhum commit.

Dados reais, conforme nota interna “TV TCL - Brief” relida (origem: docs/dispositivo.md): TCL Google TV G07_4K_GB_NF/device G07; Android 11/API 30, patch 2024-12; Realtek RTD2851A, quatro núcleos ARMv7/32 bits/armeabi-v7a; 2 GB RAM, ~750 MB livres parada; armazenamento 8,5 GB, 3,9 GB livres; UI 1080p. Bootloader travado, OEM unlock permitido mas não feito. Launcher com.google.android.apps.tv.launcherx usando ~244 MB. Jellyfin e outros apps já instalados. Esses dados são evidência interna fornecida pelo brief, não especificações presumidas de uma página web. O homeserver Jellyfin é requisito informado pelo solicitante. As horas abaixo são **estimativas próprias de planejamento**, para uma pessoa retomando programação, não promessas dos projetos; incluem aprendizado básico e ajustes, mas não publicação na loja.

## 1. Caminhos, do mais simples ao mais ambicioso

| Caminho | Stack e personalização confirmadas | Esforço estimado | PC necessário |
|---|---|---|---|
| Projectivy pronto | App Android TV com categorias/canais, ícones personalizados, atalhos, estilos e wallpapers; declara interface sem anúncios. O repositório consultado hospeda configurações, releases e issues, não o código completo do launcher. [Fonte](https://github.com/spocky/miproja1) | 2–6 h para tema, favoritos e validação; programação mínima | Pela loja, nenhum ambiente de desenvolvimento. Para APK via ADB, somente Platform-Tools; Android Studio/JDK não são necessários para instalar um APK já pronto. [ADB](https://developer.android.com/tools/adb) |
| FLauncher pronto | Flutter; categorias em linha ou grade, ordenação manual, wallpaper, relógio, apps sideloaded, sem anúncios. Não achei clima nem linhas de filmes documentados no README. [Fonte](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md) | 2–5 h | Loja ou APK + Platform-Tools; sem compilador para APK pronto. [README](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [ADB](https://developer.android.com/tools/adb) |
| ATV Launcher pronto | App da DStudio; widgets de relógio/calendário, clima e Watch Next embutidos anunciados pelo fabricante; launcher vendido na Play Store e pacotes de widgets gratuitos. Não achei confirmação suficiente para garantir ausência de anúncios em todas as versões. [Site](https://atvlauncher.com/), [Widgets](https://atvlauncher.com/widgets/) | 3–8 h | Loja: sem PC. APK autorizado do fornecedor: Platform-Tools. Não confundir os APKs gratuitos dos widgets com o launcher pago. [Widgets](https://atvlauncher.com/widgets/), [ADB](https://developer.android.com/tools/adb) |
| Fork do FLauncher | Dart/Flutter, com projeto Android; GPLv3. Permite começar por tema e categorias existentes. O upstream encontrado é **GitLab**, não GitHub. [Projeto](https://gitlab.com/flauncher/flauncher), [README](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md) | 20–50 h para pequenas mudanças; 60–120 h com integração de conteúdo | Git, Flutter SDK, Android Studio recomendado, SDK Android/Build-Tools/Platform-Tools e JDK compatível com o Gradle do projeto. O plugin Flutter sozinho não instala o Flutter SDK. [Flutter](https://docs.flutter.dev/platform-integration/android/setup), [JDK](https://developer.android.com/build/jdks) |
| Fork Android nativo alternativo | Leanback on Fire, `tsynik/LeanbackLauncher`, é fork de `leanback-on-fire/LeanbackLauncher`; seu build declara Kotlin, AndroidX Leanback, Gradle, Glide e biblioteca de clima. Não é Compose. [Repo](https://github.com/tsynik/LeanbackLauncher), [Build](https://github.com/tsynik/LeanbackLauncher/blob/kotlinx/app/build.gradle) | 30–80 h para entender e adaptar; risco maior de herdar complexidade | Android Studio, JDK e SDK coerentes com o build; não usar o `jvmTarget` como se fosse a versão exigida para executar Gradle. [Build](https://github.com/tsynik/LeanbackLauncher/blob/kotlinx/app/build.gradle), [JDK](https://developer.android.com/build/jdks) |
| Launcher próprio | Kotlin + Jetpack Compose for TV; componentes `androidx.tv.material3` e artefato `androidx.tv:tv-material`. Compose é recomendado pelo Android para apps de TV. [Compose TV](https://developer.android.com/training/tv/playback/compose), [Início](https://developer.android.com/training/tv/start/start) | 40–80 h para MVP; 120–240 h para conteúdo, clima e robustez | Android Studio, SDK/Build-Tools/Platform-Tools, Gradle Wrapper, JDK compatível e emulador Android TV. O Android Studio inclui runtime Java; AGP 8.x, por exemplo, exige JDK 17, mas deve-se seguir a matriz do AGP escolhido. [JDK](https://developer.android.com/build/jdks), [Emulador TV](https://developer.android.com/training/tv/start/start) |
| Web/WebView | HTML/CSS/JS em página e, para APK, Activity Kotlin/Java com WebView; WebView suporta conteúdo local ou remoto e integração com código Android. [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview) | 12–24 h para maquete web; 40–100 h para shell utilizável com foco e integração nativa | Maquete: editor e navegador. APK: mesmo SDK/JDK/Gradle de um app Android, mais ferramentas web escolhidas. [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview), [JDK](https://developer.android.com/build/jdks) |

**Cuidado específico no fork:** o `pubspec.yaml` consultado do FLauncher declara Flutter 3.7.5 e Dart `>=2.12.0 <3.0.0`, além de Firebase Analytics/Crashlytics/Remote Config. Portanto, minha avaliação é que atualizar o toolchain pode consumir parte considerável do esforço; “sem anúncios” não deve ser interpretado como “sem telemetria”. Não compilei o projeto nem confirmei quais serviços ficam ativos em execução. [Pubspec](https://gitlab.com/flauncher/flauncher/-/raw/master/pubspec.yaml)

O build do Leanback on Fire também inclui Firebase Analytics e Crashlytics. Eu o usaria como referência de implementação, depois de avaliar licença, dependências e orientação a Fire TV; não encontrei nesta consulta evidência suficiente para recomendá-lo como substituto validado para esta TCL. [Build](https://github.com/tsynik/LeanbackLauncher/blob/kotlinx/app/build.gradle), [Repo](https://github.com/tsynik/LeanbackLauncher)

## 2. Instalação e escolha como padrão

### Instalar — exemplos documentais, para uma fase futura

ADB integra o Android SDK Platform-Tools, disponível também separadamente do Android Studio. Requer depuração habilitada e autorização do computador no dispositivo. A nota diz que depuração já está habilitada, mas isso não comprova autorização nem conectividade. [ADB](https://developer.android.com/tools/adb)

```powershell
# Exemplos; NÃO executados nesta pesquisa
adb -s SERIAL install launcher.apk
adb -s SERIAL install -r launcher.apk
```

`install` instala o APK e `-r` reinstala mantendo os dados do app; selecionar o serial evita ambiguidade com vários dispositivos. Para launcher próprio ou fork, primeiro gerar o APK com o build correspondente; instalar não implica torná-lo HOME. [ADB](https://developer.android.com/tools/adb), [HOME](https://developer.android.com/reference/android/content/Intent#CATEGORY_HOME)

### Padrão: ordem sugerida de avaliação

1. **Seleção nativa, quando oferecida:** usar a seleção de app inicial/HOME do sistema, escolhendo o launcher e a opção permanente quando existir. HOME é uma categoria do Android, mas não achei uma instrução oficial TCL universal que garanta um menu para todos os modelos. O README do FLauncher documenta a escolha pelo botão Home após desabilitar o original, especificamente no Chromecast. [HOME](https://developer.android.com/reference/android/content/Intent#CATEGORY_HOME), [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md)
2. **Projectivy: override do launcher atual:** o projeto anuncia override; há relato primário de TCL Android TV 12 com opção de padrão cinza e uso do serviço de acessibilidade + “override current launcher”. Isso é evidência de um caso, não prova de bloqueio em toda TCL. O relato também registra interferência na busca por voz; o mantenedor sugeriu testar versões 4.64/4.65, sem confirmação de resolução nessa página. [README](https://github.com/spocky/miproja1), [Issue TCL #295](https://github.com/spocky/miproja1/issues/295)
3. **Button Mapper:** o README do FLauncher recomenda remapear Home como alternativa mais simples. A descrição oficial do Button Mapper informa que usa serviço de acessibilidade, abre apps/atalhos e que algumas ações exigem ADB ou root. Remapear um botão não equivale a registrar o aplicativo como HOME real; minha recomendação é verificar também inicialização e retorno dos apps. [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [Button Mapper](https://play.google.com/store/apps/details?id=flar2.homebutton)
4. **Último recurso: desabilitar launcher original por usuário.** Existe procedimento publicado pelo FLauncher, mas testado somente no Chromecast com Google TV. O brief confirma com.google.android.apps.tv.launcherx nesta TV; com.google.android.tungsten.setupwraith também está instalado (confirmado em docs/coleta/2026-10-24-pacotes.txt — nota do Claude Code). Mesmo com um pacote coincidente, os comandos do Chromecast não são receita validada para esta TCL. [Procedimento e reversão](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md)

```powershell
# SOMENTE exemplo do README para Chromecast com Google TV
adb shell pm disable-user --user 0 com.google.android.apps.tv.launcherx
adb shell pm disable-user --user 0 com.google.android.tungsten.setupwraith
# Reversão documentada para esse exemplo
adb shell pm enable com.google.android.apps.tv.launcherx
adb shell pm enable com.google.android.tungsten.setupwraith
```

Nesse Chromecast, o README diz que `setupwraith` pode reativar o launcher e que o botão YouTube pode parar de funcionar; sugere Button Mapper para esse botão. `disable-user` e `enable` são comandos do gerenciador de pacotes. A reversão exige ainda conseguir acessar o dispositivo; minha condição para uma futura avaliação seria ter launcher alternativo funcional, pacote original confirmado e acesso de recuperação previamente testado. Não desinstalar pacotes como atalho. [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [ADB/pm](https://developer.android.com/tools/adb)

**Resposta ao bloqueio:** não há evidência para afirmar “Google TV/TCL sempre bloqueia terceiros”. Há limitações documentadas por dispositivo e um relato TCL concreto. Instalação, seleção HOME, override por acessibilidade e remapeamento são mecanismos diferentes; o comportamento desta unidade permanece não verificado. [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [Issue TCL](https://github.com/spocky/miproja1/issues/295), [HOME](https://developer.android.com/reference/android/content/Intent#CATEGORY_HOME)

## 3. Launcher próprio: escopo e manifesto

Sugestão de arquitetura: uma Activity Kotlin/Compose, lista de favoritos configurável, persistência local e acesso a apps pelo PackageManager. `getLeanbackLaunchIntentForPackage` busca a atividade principal LEANBACK e pode retornar `null`; tratar app ausente e não fixar nome de Activity interna. O Jellyfin tem cliente oficial Android TV. [PackageManager](https://developer.android.com/reference/android/content/pm/PackageManager#getLeanbackLaunchIntentForPackage(java.lang.String)), [Jellyfin TV](https://github.com/jellyfin/jellyfin-androidtv)

Modelo mínimo ilustrativo, montado a partir dos conceitos documentados; não é um projeto compilado:

```xml
<uses-feature android:name="android.software.leanback" android:required="true" />
<uses-feature android:name="android.hardware.touchscreen" android:required="false" />
<application android:banner="@drawable/banner" android:icon="@mipmap/ic_launcher">
    <activity android:name=".MainActivity" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.HOME" />
            <category android:name="android.intent.category.DEFAULT" />
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

HOME identifica a tela inicial; LEANBACK_LAUNCHER faz o app aparecer na interface de apps da TV, não concede a posição de HOME. Banner, ícone e declaração de touchscreen não obrigatório fazem parte da configuração de TV. A seleção efetiva depende do sistema. [Intent](https://developer.android.com/reference/android/content/Intent), [Configuração TV](https://developer.android.com/training/tv/start/start)

Usar os componentes TV Material, com foco próprio para controle remoto, e evitar misturar inadvertidamente Material móvel e TV: a documentação alerta para temas e comportamentos inconsistentes. [Compose TV](https://developer.android.com/training/tv/playback/compose)

## 4. Recursos úteis e limites

| Recurso desejado | Entrega inicial e limite |
|---|---|
| Jellyfin em destaque | Um favorito grande abrindo o cliente oficial já atende ao acesso ao homeserver. No app próprio, usar intent LEANBACK do pacote instalado e tratar ausência. Não encontrei nesta pesquisa um contrato de deep link estável para garantir abrir filme/episódio específico; não inventar URI. [Cliente](https://github.com/jellyfin/jellyfin-androidtv), [API de lançamento](https://developer.android.com/reference/android/content/pm/PackageManager#getLeanbackLaunchIntentForPackage(java.lang.String)) |
| Linhas | Projectivy documenta categorias/canais; FLauncher documenta linhas/grades de apps. Essas listas não comprovam catálogo de filmes Jellyfin na HOME. Para conteúdo de mídia, avaliar TvProvider ou integração separada depois do MVP. [Projectivy](https://github.com/spocky/miproja1), [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [Canais Android](https://developer.android.com/training/tv/discovery/recommendations-channel) |
| Relógio/clima | Relógio consta no FLauncher; ATV anuncia relógios e clima. Para app próprio, proponho relógio local primeiro e cidade configurada manualmente para clima depois. Não selecionei nem validei API de clima, licença, custo ou limite de requisições. [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [ATV widgets](https://atvlauncher.com/widgets/) |
| Sem anúncios | Projectivy e FLauncher declaram não exibir anúncios em sua interface. Isso não comprova ausência de coleta de dados nem remoção de anúncios nos demais apps. Para implementação própria, proponho não incluir SDK de publicidade. Não achei promessa equivalente suficiente para ATV nas páginas lidas. [Projectivy](https://github.com/spocky/miproja1), [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [Dependências FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/pubspec.yaml) |

### TvProvider, recomendações e Watch Next

Android TV usa canais e programas; desde Android 8/API 26, recomendações podem ocupar linhas separadas. O primeiro canal tem tratamento padrão, e os adicionais dependem de aprovação do usuário. Não confundir canais de recomendação com entradas HDMI ou canais de sintonia. [Recomendações](https://developer.android.com/training/tv/discovery/recommendations), [TvContract](https://developer.android.com/reference/android/media/tv/TvContract)

Para publicar canais, a documentação usa `androidx.tvprovider`, `TvContractCompat` e permissão `com.android.providers.tv.permission.WRITE_EPG_DATA`; informa que `READ_EPG_DATA` foi descontinuada no Android M. Isso documenta publicação dos dados do app, **não demonstra acesso irrestrito às recomendações privadas de outros apps por um launcher comum**. Não achei nesta pesquisa uma autorização genérica que assegure esse acesso na TCL. [Criar canais](https://developer.android.com/training/tv/discovery/recommendations-channel)

Watch Next é mantido pelo sistema e aceita, por exemplo, continuar assistindo, próximo episódio e watchlist. O app publica metadados e intent de destino; a documentação limita mover, ocultar ou remover a linha na HOME do sistema. Para uma HOME própria, desenhar uma linha visual não comprova que ela conseguirá consumir os dados de todos os provedores ou reproduzir o feed personalizado do Google TV. [Watch Next](https://developer.android.com/training/tv/discovery/watch-next-add-programs)

### Permissões e D-pad

Apps mirando Android 11+ têm regras de visibilidade: declarar `<queries>` para pacotes ou intents necessários; não assumir que listar todos os apps funcionará sem configuração. A documentação desaconselha `QUERY_ALL_PACKAGES` como solução indiscriminada. [Visibilidade](https://developer.android.com/training/package-visibility/declaring)

Para dados remotos em WebView, declarar `INTERNET`. Button Mapper usa acessibilidade; Projectivy também tem override por acessibilidade registrado na issue TCL. Não proponho conceder acessibilidade ou administração ao launcher próprio apenas para exibir cards. [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview), [Button Mapper](https://play.google.com/store/apps/details?id=flar2.homebutton), [Projectivy/TCL](https://github.com/spocky/miproja1/issues/295)

O Android recomenda foco inicial lógico, movimento bidimensional previsível, indicação clara, restauração de foco e rolagem que mantenha o item selecionado visível. Critérios propostos: percorrer todos os cards sem mouse, voltar do Jellyfin ao card anterior, atravessar linhas, abrir/fechar configurações e lidar com lista vazia sem perder foco. Validar em emulador primeiro; validação na TV fica para fase autorizada posterior. [Foco Compose](https://developer.android.com/develop/ui/compose/touch-input/focus), [Emulador TV](https://developer.android.com/training/tv/start/start)

## 5. Alternativas web e WebView

**Página web no homeserver:** recomendação como protótipo visual de layout, relógio e dados; não encontrei mecanismo documentado para uma página isolada registrar-se diretamente como HOME Android. HOME pertence ao contrato de Activity/Intent do sistema. Abrir uma página no navegador não demonstra substituição da HOME. [HOME](https://developer.android.com/reference/android/content/Intent#CATEGORY_HOME)

**Shell Android com WebView:** pode carregar HTML local ou remoto e interceptar URLs para acionar código Android; para abrir Jellyfin, propor uma ponte restrita a ações permitidas e lançar o cliente nativamente. O shell ainda precisa dos filtros HOME/LEANBACK e enfrenta as mesmas condições de seleção de launcher. [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview), [Configuração TV](https://developer.android.com/training/tv/start/start), [Intent](https://developer.android.com/reference/android/content/Intent)

JavaScript é desabilitado por padrão no WebView; habilitá-lo e usar `addJavascriptInterface` requer cuidado com conteúdo não confiável. A documentação também registra maior uso de memória do WebView em relação a Views padrão. Minha avaliação: bom para reaproveitar conhecimento web, mas foco por D-pad, navegação Voltar e ponte nativa reduzem a vantagem para um launcher pequeno. Não validei desempenho nesta TCL. [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview)

## 6. Recomendação para voltar a programar sem travar a entrega

**Minha recomendação: Projectivy para o uso diário, e launcher pequeno próprio em Kotlin + Compose for TV como trilha de aprendizado.** Projectivy já documenta personalização e ausência de anúncios; Compose TV é o caminho recomendado pelo Android para interface nova. Separar os dois objetivos permite que o aprendizado avance sem transformar a tela inicial da casa em dependência de um projeto inacabado. [Projectivy](https://github.com/spocky/miproja1), [Compose TV](https://developer.android.com/training/tv/playback/compose)

Plano proposto, ainda sem implementação:

1. Definir tema legível, poucos favoritos e Jellyfin como primeiro card; usar os dados já coletados e validar o caminho de HOME no firmware antes de escolher override.
2. Em emulador, criar Activity Compose TV com 4–8 cards, relógio e configurações; aprender Kotlin, estado, composição, foco e persistência em tarefas pequenas.
3. Implementar abertura de apps e restauração de foco. Só considerar trocar HOME depois de demonstrar fluxo de retorno, estabilidade e recuperação.
4. Acrescentar clima e, por último, conteúdo Jellyfin/Watch Next, cada qual como entrega independente.

A escolha de Kotlin ensina mais diretamente a plataforma Android e suas integrações; isso é avaliação pedagógica minha, não resultado de benchmark. Se o Sr. Hao preferir Dart/Flutter, o fork do FLauncher é alternativa legítima, mas eu começaria por uma mudança visual pequena e reservaria tempo para o toolchain antigo. Um fork amplo de Leanback on Fire parece menos previsível para a primeira entrega, pelas dependências e contexto de Fire TV documentados. [FLauncher/pubspec](https://gitlab.com/flauncher/flauncher/-/raw/master/pubspec.yaml), [Leanback on Fire](https://github.com/tsynik/LeanbackLauncher/blob/kotlinx/app/build.gradle)

### Lacunas registradas

Já confirmados pelo brief: G07/Google TV, Android 11, ARMv7, memória e pacote launcherx. Não foram confirmados: comportamento específico do firmware; seleção HOME nessa unidade; funcionamento persistente do override após reinicialização; impacto nos botões/voz/entradas; leitura de canais de terceiros; deep links Jellyfin por item; API de clima; ausência de anúncios em ATV; compatibilidade de compilação atual do FLauncher. Os URLs GitHub tentados para FLauncher não forneceram um repositório válido; a fonte upstream localizada e lida foi o GitLab. Nenhuma dessas lacunas foi preenchida por suposição.

## 7. Recomendação ajustada ao hardware real

**Prioridade: entregar uma HOME simples e validar consumo antes de prometer otimização.** Os 2 GB de RAM, ~750 MB livres em repouso e ~244 MB atribuídos ao launcher original vêm do brief interno. Não foi informado o método de medição (por exemplo, PSS/RSS), nem há medição equivalente de Projectivy, FLauncher ou ATV nesta TV. Portanto, não há base para estimar quantos MB serão economizados, ganho de velocidade ou redução de travamentos. Manter o launcher original e abrir outro também não comprova que o processo original deixará de consumir memória; isso precisa ser observado em uma fase futura.

Configuração que proponho para esta G07: Projectivy com **wallpaper estático**, sem GIF/vídeo, apenas uma linha principal de favoritos e uma segunda linha opcional. Primeiro card: Jellyfin; depois os apps de uso frequente escolhidos pelo Sr. Hao entre os já instalados. A documentação permite categorias, estilos e wallpapers, mas não fornece benchmark para este Realtek. Minha escolha é por escopo e facilidade de personalização, não por suposta superioridade de RAM. FLauncher fica como alternativa de interface simples com relógio e categorias; ATV merece teste somente se os widgets forem prioridade. [Projectivy](https://github.com/spocky/miproja1), [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [ATV](https://atvlauncher.com/widgets/)

**Aprendizado: continuo recomendando Kotlin + Compose TV em paralelo**, com MVP enxuto, interface pensada para a UI 1080p informada, sem player próprio, sem vídeos automáticos e sem catálogo na primeira versão. Compose para TV documenta suporte a Android 5.0/API 21 ou superior, portanto Android 11/API 30 atende ao piso documentado; ainda será preciso conferir o minSdk das versões concretas de todas as dependências. Não estou afirmando que Compose seja mais leve que Flutter: não há benchmark comparativo desta unidade. [Compatibilidade Compose TV](https://developer.android.com/training/tv/playback/compose)

**ARM de 32 bits é condição obrigatória de distribuição.** Para o fork Flutter, a documentação confirma geração de APK `app-armeabi-v7a-release.apk`; não escolher um artefato exclusivamente arm64. Validar também plugins e bibliotecas nativas do fork. Para Kotlin, conferir quaisquer dependências com código nativo antes de entregar o APK; não presumir que uma build que funciona em emulador x86/64 prova compatibilidade ARMv7. O formato armeabi-v7a/ARM 32 bits e os artefatos Flutter estão documentados na fonte abaixo. [Build Android Flutter](https://docs.flutter.dev/deployment/android)

**Google TV e padrão HOME:** agora sabemos que a TV usa launcherx, mas permanece desconhecido se este firmware permite selecionar HOME nativamente. Priorizar escolha nativa quando disponível; depois avaliar override do Projectivy ou Button Mapper. Adiar `pm disable-user` até que a recuperação e os efeitos em busca, botões e entradas estejam demonstrados. O README do FLauncher limita seus comandos ao Chromecast e a issue TCL registra conflito de voz com override. Nenhum desses mecanismos foi executado aqui. [FLauncher](https://gitlab.com/flauncher/flauncher/-/raw/master/README.md), [Relato TCL](https://github.com/spocky/miproja1/issues/295)

**ADB no Android 11 da TV:** não aplicar automaticamente o fluxo moderno de pareamento Wi-Fi de smartphones Android 11. A documentação atual distingue TVs, indicando Android 13+ para esse fluxo. O transporte de depuração efetivamente disponível nesta TCL deve ser confirmado pelo responsável pela coleta; esta frente não realizou conexão. Para instalar APK pronto, continuam suficientes Platform-Tools e conexão autorizada; bootloader desbloqueado não figura como requisito desse procedimento documentado. [ADB e pré-requisitos](https://developer.android.com/tools/adb)

**WebView cai na prioridade para a entrega final:** permanece útil para protótipo, mas o Android documenta custo de memória maior que Views padrão e há trabalho adicional de foco e ponte nativa. Com 2 GB, eu não acrescentaria essa camada ao MVP sem uma razão concreta e medição. Essa é uma avaliação de projeto; a comparação da documentação é WebView versus Views padrão, não WebView versus Compose ou Flutter. [WebView](https://developer.android.com/develop/ui/views/layout/webapps/webview)

Plano de validação proposto para fase futura, sem executar nesta análise: registrar a mesma métrica de memória no mesmo estado de repouso; repetir após uso e retorno do Jellyfin; verificar Home/Voltar, voz, entradas e reinicialização; comparar cada candidato separadamente. Caso a substituição não traga benefício observável, preservar a personalização por conveniência sem alegar economia de RAM. O foco de aprendizado continua no MVP próprio, sem condicionar o uso diário à conclusão das integrações avançadas.

