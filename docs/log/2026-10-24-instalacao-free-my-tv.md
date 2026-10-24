# 24/10/2026: instalação do Free My TV

## Linha de base (antes, home do Google TV parada)

| Métrica | Valor |
|---|---|
| RAM livre (`dumpsys meminfo`, Free RAM) | 716 MB |
| RAM usada (Used RAM) | 1.266 MB |
| `launcherx` + `coreservices` | 202 MB + 35 MB |
| `dreamx` (proteção de tela) | 75 MB |
| Pacotes ativos | 131 |

## O que foi feito na TV

| # | Comando (resumido) | Efeito | Como reverter |
|---|---|---|---|
| 1 | `adb install -r FreeMyTV.apk` | Instala o app (build do fork, sem TMDB e sem cripto, voz desligada) | `adb uninstall io.freemytv` (antes, voltar para "Original" no app) |
| 2 | `appops set io.freemytv APP_AUTO_START allow` | Libera o app para rodar em segundo plano na TCL. Sem isso, a troca e o watchdog não funcionam. (`AUTO_START` não existe nesta TV.) | `appops set io.freemytv APP_AUTO_START default` |
| 3 | `cmd wifi forget-network <id>` + `cmd wifi connect-network <ssid> wpa2 <senha>`, rodando dentro da TV com `nohup` | Recria a rede Wi-Fi da casa, que tinha sido salva pelo `setupwraith` (configuração inicial). Sem isso, o Android apagaria a rede quando o free-my-tv desativasse o `setupwraith`. | Não precisa. A rede nova é equivalente. |

### Verificações após o passo 3

- A TV voltou à rede em ~5 s, **com o mesmo IP** (reservado por MAC no roteador).
- **O MAC continuou o de fábrica.** As redes salvas nesta TV usam `macRandomizationSetting: 0`, então a TV não sorteia MAC.
- Nenhuma rede ficou com `cname=com.google.android.tungsten.setupwraith`.

## Jellyfin: login verificado

- Servidor: Jellyfin 10.11.10 no homeserver, na porta **8097** (não na 8096 padrão). A descoberta UDP (7359) não responde.
- **Não precisa de chave de API.** `POST /Users/AuthenticateByName` com usuário e senha devolve `AccessToken` e `User.Id`.
- `GET /Users/{id}/Items/Resume` → 49 itens no usuário da TV, todos com imagem de fundo. `GET /Shows/NextUp` → 24.
- As credenciais ficam só no `config.properties` local (fora do git).

## Próximo passo

A troca de launcher é feita **na TV** (Free My TV → Tela inicial → Free My TV), seguindo a orientação do projeto.
