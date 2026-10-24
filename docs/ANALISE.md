# Análise: o que dá para fazer na TV TCL (24/10/2026)

Documento para o Sr. Hao avaliar **antes** de qualquer modificação. Nada foi alterado na TV até agora.

- Dados reais da TV: [`dispositivo.md`](dispositivo.md)
- Pesquisa de otimizações (Gemini/Antigravity), com fontes: [`pesquisa/otimizacoes.md`](pesquisa/otimizacoes.md)
- Pesquisa de interface personalizada (Codex), com fontes: [`pesquisa/interface-personalizada.md`](pesquisa/interface-personalizada.md)

## 1. O ponto de partida

| Fato | Por que importa |
|---|---|
| **Google TV** (não Android TV clássico), Android 11 | O launcher do Google TV é pesado e tem anúncios. Algumas travas de launcher são específicas do Google TV. |
| **2 GB de RAM**, ~750 MB livres com a TV parada | Pouca folga. Cada 30 a 40 MB liberados fazem diferença. |
| **Realtek RTD2851A, 32 bits** | Apps próprios precisam sair em `armeabi-v7a`. Guias de root da TCL são para MediaTek e não servem aqui. |
| **Launcher do Google TV = ~280 MB** (244 + 36 do serviço dele) | É o maior alvo isolado. |
| Bootloader travado, OEM unlock permitido | Dá para fazer quase tudo por ADB, sem root. |

## 2. Opções por nível de risco

Todas usam ADB pela rede, sem root. **"Desativar"** = `pm disable-user --user 0`, que se reverte com `pm enable`. Não vamos usar `uninstall`.

### Nível A: baixo risco, reversível, ganho certo

| # | O que | Ganho esperado | Condição |
|---|---|---|---|
| A1 | Backup da lista de pacotes e da memória (linha de base) | Ponto de restauração | Obrigatório antes de tudo |
| A2 | Animações de 1.0 para 0.5 | Menus parecem mais rápidos | Nenhuma |
| A3 | Desativar telemetria TCL (`com.tcl.ttvs`) | ~33 MB | Nenhuma |
| A4 | Desativar Safety Guard (`com.tcl.guard`) | ~19 MB. Também evita que ele derrube o launcher alternativo depois de reiniciar a TV. | Perde só o "limpador" da TCL |
| A5 | Desativar apps de loja/demonstração/hotel (`esticker`, `factory.view`, `hotelmenu`, `notereminder`, `useragreement`) | Pouca RAM, menos processos | Nenhuma |
| A6 | Desativar extras Google não usados (`videos`, `play.games`, `dreamx`, `feedback`) | ~33 MB+ | `dreamx` é a proteção de tela. Sem ele, a tela não entra em descanso. |
| A7 | Limpar cache de todos os apps (`pm trim-caches`) | Espaço em disco | Não apaga logins |

### Nível B: médio risco ou perde alguma função (decisão sua)

| # | O que | Ganho | O que você perde |
|---|---|---|---|
| B1 | Desativar vitrines/anúncios TCL (`channelplus`, `messagebox`, `dashboard`, `tclhome_passive`, `partnercustomizer`) | Menos propaganda e processos | Depende do pacote. Aplicar **um por vez** e reiniciar. |
| B2 | `com.tcl.waterfall.overseas` | Remove conteúdo patrocinado | **Há relato de bootloop no logo** com ele desativado ([gist 0x4f53](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), conferido). Fica por último, e só se valer o risco. |
| B3 | Assistente/voz (`katniss`) | ~37 MB | Botão de microfone e busca por voz |
| B4 | Chromecast (`mediashell`) e T-Cast | ~31 MB | Espelhar e transmitir do celular |
| B5 | Limitar processos em cache (hoje 32) | Menos apps acumulados na RAM | A pesquisa sugere 2 a 4, o que **acho agressivo**: trocar de app passa a recarregar tudo. Sugiro testar 8 antes. |
| B6 | **Launcher leve via acessibilidade** (Projectivy "override") | Interface limpa, sem anúncios | Mantém o launcher do Google rodando, então quase não libera RAM. Há relato de conflito com a busca por voz na TCL ([issue #295](https://github.com/spocky/miproja1/issues/295)). |
| B7 | **Desativar o launcher do Google** (`launcherx` + `setupwraith`) | **~280 MB**, o maior ganho | Só depois de um launcher alternativo testado. Fora de ordem, a TV fica com tela preta no Home (recuperável por ADB). Pode quebrar o botão de Configurações ou o do YouTube do controle. |

### Nível C: não recomendo

| O que | Por quê |
|---|---|
| Desbloquear bootloader / root / TWRP | Não há método documentado para Realtek G07. Apaga tudo, o Widevine cai de L1 para L3 (Netflix/Prime em ~720p) e há risco de brick com recuperação só por UART ([XDA](https://xdaforums.com/t/guide-rooting-tcl-t653t01-models-eg-c855.4774098/), conferido). Nada disso traz ganho real sobre o ADB. |
| Desativar `com.android.location.fused` | Bootloop comprovado com a localização ligada ([gist](https://gist.github.com/0x4f53/27bee9a0069f7bd6a2aa9eb60bb10d1e), conferido). |
| Desativar `com.tcl.tv`, `com.tcl.tvinput` | Quebra entradas HDMI e CEC. |
| `pm uninstall` em vez de `disable-user` | Reversão mais difícil, sem ganho a mais. |

## 3. Interface personalizada

| Caminho | Esforço | Ensina programação? | Ganho de RAM |
|---|---|---|---|
| Projectivy (pronto, configurável) | 2 a 6 h | Não | Só se combinado com B7 |
| FLauncher (pronto, open source, Flutter) | 2 a 5 h | Não | Só se combinado com B7 |
| Fork do FLauncher | 20 a 120 h | Dart/Flutter, mas o toolchain do projeto é antigo (Flutter 3.7) | Só com B7 |
| **Launcher próprio em Kotlin + Compose for TV** | 40 a 80 h (MVP) | **Sim. É o caminho oficial do Android para TV.** | Só com B7 |
| Web/WebView | 12 a 100 h | HTML/JS | Pior (o WebView consome mais memória) |

As horas são estimativas de planejamento do Codex, não medições.

**Recomendação (Codex e eu concordamos):** dois trilhos.

1. **Uso diário:** Projectivy (ou FLauncher) com o Jellyfin em primeiro lugar. Resolve rápido e não deixa a TV da casa dependendo de código inacabado.
2. **Aprendizado:** launcher próprio em Kotlin + Compose for TV, feito por partes: primeiro favoritos e relógio, depois o atalho do Jellyfin, depois clima e conteúdo. Quando estiver estável, ele substitui o Projectivy. Isso atende ao seu objetivo de voltar a programar de verdade.

**Limites conhecidos de um launcher próprio:** ele não reproduz o feed de recomendações do Google TV. Abrir um filme específico do Jellyfin direto da home não tem deep link documentado. Mostrar o catálogo do Jellyfin na home é possível pela API do servidor, mas é trabalho à parte.

**O que precisa no PC para o trilho 2:** Android Studio (já traz o JDK), SDK Android e um emulador de Android TV. O Platform-Tools já está instalado.

## 4. Sequência que proponho

1. **Fase 1, segura (nível A):** backup → A2 a A7 → reiniciar → medir a RAM de novo e comparar com a linha de base. Tudo registrado em `docs/log/`.
2. **Fase 2, launcher:** instalar o Projectivy → testar uma semana pela acessibilidade (B6) → se aprovado, B7 para liberar ~280 MB → medir.
3. **Fase 3, nível B restante:** só os itens que você escolher, um por vez.
4. **Em paralelo:** projeto do launcher próprio em `launcher/` neste repo.

## 5. Decisões que preciso de você

1. Quais itens do **nível B** você topa perder: voz (B3)? Chromecast/T-Cast (B4)?
2. Launcher do dia a dia: **Projectivy** (mais completo, código fechado) ou **FLauncher** (open source, mais simples)?
3. Começamos o launcher próprio já ou depois da Fase 1?
4. Você usa o botão de microfone do controle? E o Chromecast?

## Como foi feito

- Coleta na TV: Claude Code via ADB, só com comandos de leitura.
- Pesquisa: Gemini (Antigravity, agente "Aurora") e Codex (agente "Ferrolho"), cada um no seu portal do Maestri.
- Revisão: Claude Code conferiu na fonte original duas das afirmações mais críticas (bootloop do `location.fused` e Widevine/UART no root) e corrigiu uma inconsistência do relatório de interface (o `setupwraith` está instalado nesta TV).
