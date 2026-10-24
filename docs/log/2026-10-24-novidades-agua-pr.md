# 24/10/2026: PR da correção do Home, Novidades, lembrete de água, TTS e atualização 1.2.0

## Contribuição ao projeto original

PR **[simbasjungles/free-my-tv#259](https://github.com/simbasjungles/free-my-tv/pull/259)**: a correção do botão Home na TCL (Android 11). A branch `fix/tcl-home-translucent` foi criada a partir da versão atual do autor, **só com essa mudança**, e testada na TV antes do envio: depois de Home, o YouTube ficou `STOPPED`. Autor do commit: Pedro Hao Tavares, sem coautoria.

## Itens 2, 3 e 4 da lista: o que foi feito e o que não foi

| Pacote | Decisão | Motivo (verificado na TV com `dumpsys package`) |
|---|---|---|
| `com.tcl.ttvs` | **Mantido** | Não é só telemetria: é o serviço central da TCL (uid de sistema), que também roda o servidor do T-Cast, o teclado virtual, o vídeo pela rede e o atualizador da loja. O autor do free-my-tv o protege desde a 1.0. |
| `com.tcl.guard` | **Mantido** | É o gerenciador de memória da TCL (`TGuardService`, `HyperHold`), marcado `PERSISTENT`. Protegido pelo autor. |
| `com.google.android.tts` | **Desligado** (~45 MB) | Entrou no grupo `ASSISTANT` (voz) do app: com `disable_assistant=true` ele sai junto, e o botão Original o religa. |

A pesquisa inicial (`pesquisa/otimizacoes.md`) classificava os dois primeiros como seguros, com base em listas genéricas de debloat. A verificação no aparelho mostrou o contrário.

## Aba "Novidades"

Segunda aba, depois de Início: `/Users/{id}/Items/Latest` de todas as bibliotecas (séries aparecem como série, não episódio por episódio). [Captura](../img/2026-10-24-aba-novidades.png)

## Lembrete de água

- `Water.java`: alarme repetido a cada `water_every` minutos (configurado: **60**), só entre `water_from` (8h) e `water_to` (23h), e só com a tela ligada.
- Mostra um cartão "💧 Hora de beber água" no canto superior direito por 8 segundos, **por cima de qualquer app**, sem roubar o foco do controle.
- Precisa de `appops set io.freemytv SYSTEM_ALERT_WINDOW allow` (feito). Para reverter: `appops set io.freemytv SYSTEM_ALERT_WINDOW default`; para desligar o lembrete: `water_every=0` e rebuild.
- Testado com intervalo de 1 minuto, sobre o Jellyfin: [captura](../img/2026-10-24-lembrete-agua.png)

## Atualização com o upstream (1.2.0)

Feita no meio da sessão, porque um teste do PR instalou na TV a versão original 1.2.0, mais nova que o fork. Procedimento e cuidados em [ATUALIZACAO.md](../ATUALIZACAO.md). Conflitos: traduções e `build.sh`, resolvidos mantendo os dois lados.

## Estado final

- Abas: Início, Novidades, Filmes, Séries, Desenhos, Animes. [Captura](../img/2026-10-24-home-v2.png)
- Botão Home: correção mantida após o merge (Jellyfin `STOPPED` depois de Home).
- RAM livre: 715 MB. A home subiu para ~234 MB, porque guarda as imagens de 6 abas. Próxima otimização sugerida: imagens dos cartões em resolução menor.

Código: fork `PedroHaoTavares/free-my-tv`, commit `d3be4c4`.
