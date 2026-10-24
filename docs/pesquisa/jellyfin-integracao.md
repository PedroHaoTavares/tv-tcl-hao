# Integração com o Jellyfin (verificada em 24/10/2026)

## 1. Ler o "Continuar assistindo" (API do servidor)

Testado contra o Jellyfin 10.11.10 do homeserver (porta 8097), usuário da TV.

| Passo | Chamada | Resultado |
|---|---|---|
| Login | `POST /Users/AuthenticateByName` com `{"Username": ..., "Pw": ...}` e o cabeçalho `Authorization: MediaBrowser Client="FreeMyTV", Device="...", DeviceId="...", Version="..."` | Devolve `AccessToken` e `User.Id`. **Não precisa de chave de API.** |
| Continuar assistindo | `GET /Users/{userId}/Items/Resume?MediaTypes=Video&Limit=N` com `Token="..."` no mesmo cabeçalho | 49 itens. Cada um tem `Id`, `Name`, `SeriesName`, `Type`, `UserData.PlayedPercentage` e tags de imagem (`BackdropImageTags` / `ParentBackdropImageTags`). |
| Próximos episódios | `GET /Shows/NextUp?UserId={userId}&Limit=N` | 24 itens |
| Imagens | `GET /Items/{id}/Images/Backdrop` (ou `Primary`, `Thumb`), com `?maxWidth=1280` | Imagem pronta para o fundo da tela |

## 2. Abrir o episódio direto no app oficial

O código do app `org.jellyfin.androidtv` (master) mostra:

- `app/src/main/AndroidManifest.xml`: `.ui.startup.StartupActivity` é **exportada** e trata `MAIN`, `SEARCH` e `VIEW`.
- `app/src/main/java/org/jellyfin/androidtv/ui/startup/StartupActivity.kt`:
  ```kotlin
  const val EXTRA_ITEM_ID = "ItemId"
  ...
  val itemId = when {
      intent.action == Intent.ACTION_VIEW && intent.data != null -> intent.data.toString()
      else -> intent.getStringExtra(EXTRA_ITEM_ID)
  }?.toUUIDOrNull()
  ...
  itemId != null -> Destinations.itemDetails(itemId)
  ```
  Fonte: https://github.com/jellyfin/jellyfin-androidtv/blob/master/app/src/main/java/org/jellyfin/androidtv/ui/startup/StartupActivity.kt

### Teste real na TV

```bash
adb shell am start -n org.jellyfin.androidtv/.ui.startup.StartupActivity --es ItemId <ItemId>
```

**Resultado:** o app abriu na **tela de detalhes do episódio** com **"Retomar de 2:25"** em foco. Um OK no controle continua de onde parou. Captura: [`img/2026-10-24-jellyfin-deeplink.png`](../img/2026-10-24-jellyfin-deeplink.png).

### Condições

- O app do Jellyfin usa o **último servidor e usuário logados nele**. O item precisa existir para esse usuário, então o launcher e o app devem usar o mesmo usuário.
- Ele abre os detalhes, não o player direto. Isso é bom: permite escolher entre "Retomar" e "Reproduzir".

## 3. Equivalente em Java (para o free-my-tv)

```java
Intent i = new Intent();
i.setClassName("org.jellyfin.androidtv", "org.jellyfin.androidtv.ui.startup.StartupActivity");
i.putExtra("ItemId", itemId);
i.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
context.startActivity(i);
```
