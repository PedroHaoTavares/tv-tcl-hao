# Decisões

## 24/10/2026: rumo do projeto

| Decisão | Detalhe |
|---|---|
| Linha de ação | **Nível B** da [análise](ANALISE.md): trocar o launcher do Google TV e desativar o que o Sr. Hao não usa. |
| Voz / Assistente | **Desativar.** O Sr. Hao não usa o botão de microfone. |
| Chromecast / T-Cast | **Desativar.** Não usa. |
| Launcher | **Próprio**, desde já, pulando o launcher pronto (Projectivy/FLauncher). |
| Quem desenvolve | Os agentes (Claude Code, Gemini, Codex). O Sr. Hao revisa e aprova. |
| Base de partida | **[free-my-tv](https://github.com/simbasjungles/free-my-tv)** (licença MIT), indicado pelo Sr. Hao. A interface dele ainda não agrada 100% e será personalizada. |

## Por que o free-my-tv substitui o "Kotlin + Compose do zero"

Fonte: README do projeto, lido em 24/10/2026 (MIT, 4,1 mil estrelas, 288 commits).

| Ponto | free-my-tv | Compose do zero |
|---|---|---|
| Testado em TCL Google TV | Sim. A home usou **96 MB contra 242 MB** do launcher original (medido pelo autor numa TCL com Android 14). | Não |
| Tecnologia | Java puro, desenhado à mão num Canvas, sem bibliotecas. APK de ~2 MB. | Kotlin + Compose (mais bibliotecas, mais memória) |
| Suporta Android 11 | Sim (Android 8+) | Sim |
| Rede de segurança | Botão "Original" restaura tudo. Watchdog volta ao launcher original após 3 travamentos em 5 min. Lista de pacotes protegidos. | Teria que ser escrita |
| Debloat TCL | Já embutido: TCL Channel, mensagens, demo de loja, menu de hotel, espelhamento e logs | Teria que ser escrito |
| Botões Netflix/YouTube/Prime do controle TCL | Continuam funcionando por um serviço de acessibilidade mínimo | Teria que ser escrito |
| Esforço até ter algo na TV | Horas | 40 a 80 h |

### Pontos de atenção levantados

1. **O visual de vidro fosco e o céu animado só aparecem no Android 13+.** No Android 8 a 12 o app usa cores chapadas e sombras mais leves. **A TV do Sr. Hao é Android 11**, então as capturas do README não mostram como vai ficar nela.
2. **O app usa o ADB da própria TV** (conecta em `127.0.0.1:5555`) para trocar a home e desativar pacotes. Por isso a depuração precisa continuar ligada.
3. **Na TCL**, é preciso liberar o início automático (`appops set io.freemytv AUTO_START allow`). Sem isso, a troca não acontece.
4. **Dependências externas:** TMDB (chave grátis, para as sugestões), Open-Meteo (clima) e Binance (cripto, opcional e desligável).
5. **Com o launcher do Google desligado**, pedir um filme ao Assistente pode falhar. Isso não afeta o Sr. Hao, porque a voz será desativada.

## Plano proposto (aguardando aprovação)

1. Fork `simbasjungles/free-my-tv` → `PedroHaoTavares/free-my-tv` (ou o código dentro deste repo, em `launcher/`).
2. Montar o ambiente de build no PC: JDK 17, Android command-line tools, build-tools 34 e platform android-34.
3. Gerar o APK sem mudanças e instalar na TV com `disable_assistant=true`, mais a desativação do Chromecast/T-Cast. Medir a RAM antes e depois.
4. O Sr. Hao usa alguns dias e lista o que não agrada.
5. Os agentes personalizam a interface e acrescentam o que for dele (exemplo: linha "Continuar assistindo" do Jellyfin).
