# 24/10/2026: aba Início, pôsteres, YouTube Music e apps removidos

## Apps removidos da TV (pedido do Sr. Hao)

| App | Pacote | Ação | Como reverter |
|---|---|---|---|
| Apple TV | `com.apple.atve.androidtv.appletv` | `adb uninstall` | Instalar pela Play Store |
| Max | `com.wbd.stream` | `adb uninstall` | Play Store |
| Disney+ | `com.disney.disneyplus` | `adb uninstall` | Play Store |
| Star+ | `com.disney.starplus` | `adb uninstall` | Play Store |
| Paramount+ | `com.cbs.ca` | `adb uninstall` | Play Store |
| Netflix | `com.netflix.ninja` | `pm disable-user --user 0` (é app de **sistema**, não sai) | `pm enable com.netflix.ninja` |

Espaço livre: **3,9 GB → 4,5 GB**.

## Prateleira

`shelf=` com Jellyfin, Globoplay, Prime Video, YouTube e **YouTube Music** (no lugar do Spotify, que continua instalado).

### YouTube Music era desativado de novo (corrigido)

1. O free-my-tv tinha o YouTube Music na lista `DEBLOAT`. Retirado.
2. Mesmo assim, após a atualização o app avisou "Desliguei o que faltava: com.google.android.youtube.tvmusic". **Causa:** a verificação periódica (`HomeMode`) desliga de novo **tudo o que ficou salvo na lista do momento da troca**, e não só o que está na `DEBLOAT` atual.
3. **Correção:** a lista salva só vale para pacotes que ainda são alvo (`targets()`) ou que já estavam desligados antes do Free My TV (`keptOff`).
4. **Verificado:** o YouTube Music continuou ligado após a atualização **e após reiniciar a TV**.

## Aba "Início"

Primeira aba, antes de Filmes, Séries, Desenhos e Animes. Mostra o que o Sr. Hao estava assistindo por último, **em qualquer biblioteca** (`/Items/Resume` sem `ParentId`, do mais recente para o mais antigo), completado com os próximos episódios.

## Pôsteres nos cartões em pé

- Cada item ganhou um pôster: o **da série** para episódios e o **do próprio item** para filmes (`/Items/{id}/Images/Primary?maxHeight=928`).
- O cartão largo continua com a imagem de fundo; os dois cartões em pé usam o pôster.
- **Atenção técnica:** o parâmetro `EnableImageTypes` precisa incluir `Primary`, senão o servidor omite as tags de pôster.
- O caso do Norbit (fundo branco no meio) foi resolvido: agora aparece a capa.
- `CACHE_VERSION` subiu para 5, porque o formato salvo mudou. Sem isso, depois da atualização os cartões ficaram vazios por até 1 minuto (o app lia o cache da versão anterior).

Capturas: [Início](../img/2026-10-24-inicio-posteres.png), [Filmes](../img/2026-10-24-aba-filmes-posteres.png)

## RAM

795 MB livres. Home: 204 MB.

Código: fork `PedroHaoTavares/free-my-tv`, commit `10ba725`.
