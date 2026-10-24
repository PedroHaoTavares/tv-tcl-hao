# tv-tcl-hao

Otimização e interface personalizada para uma TV TCL com Google TV (codinome `G07_4K_GB_NF`, Android 11, Realtek RTD2851A, 2 GB de RAM), feitas via ADB e sem root.

## Status

**Ambiente de build pronto.** Launcher: fork do [free-my-tv](https://github.com/PedroHaoTavares/free-my-tv). Nada foi alterado na TV ainda.

## Documentação

| Arquivo | Conteúdo |
|---|---|
| [`docs/ANALISE.md`](docs/ANALISE.md) | **Comece aqui.** O que dá para fazer, riscos, condições e decisões pendentes. |
| [`docs/DECISOES.md`](docs/DECISOES.md) | Decisões tomadas e plano do launcher (base: free-my-tv). |
| [`docs/AMBIENTE.md`](docs/AMBIENTE.md) | Ferramentas no PC, build do launcher e pré-requisitos da TV. |
| [`docs/dispositivo.md`](docs/dispositivo.md) | Perfil da TV coletado via ADB. |
| [`docs/pesquisa/otimizacoes.md`](docs/pesquisa/otimizacoes.md) | Debloat, desempenho, launcher, root: pesquisa completa com fontes. |
| [`docs/pesquisa/interface-personalizada.md`](docs/pesquisa/interface-personalizada.md) | Caminhos para uma interface própria, com fontes. |
| [`docs/coleta/`](docs/coleta/) | Listas brutas de pacotes. |

## Regras do repositório

- Repositório público: nada de IP, número de série ou nome do aparelho.
- Toda alteração na TV é registrada em `docs/log/` com o comando exato e como reverter.
- Desativar (`pm disable-user`) em vez de desinstalar.
