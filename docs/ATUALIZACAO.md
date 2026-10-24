# Atualizar o launcher com o projeto original (rotina mensal)

## Por quê

O nosso launcher é um **fork**: uma cópia do [free-my-tv](https://github.com/simbasjungles/free-my-tv) com as nossas mudanças (Jellyfin, abas, dólar, lembrete de água, prateleira). O autor original continua lançando melhorias e correções (em 24/10/2026 saiu a 1.2.0, com timer de desligamento e correções para Android 12). Nada disso chega sozinho ao fork: é preciso **trazer as mudanças dele e juntar com as nossas**.

Analogia: o autor é a editora de um livro; o fork é o seu exemplar, cheio de anotações. A rotina consiste em pegar a edição nova e passar as suas anotações para ela.

## Passo a passo (um agente faz; o Sr. Hao só aprova)

```bash
cd /c/Users/pphen/free-my-tv
git fetch upstream                      # baixa o que o autor publicou
git log --oneline main..upstream/main   # lista o que é novo
git merge upstream/main                 # junta com as nossas mudanças
# conflitos: os dois lados mexeram na mesma linha; resolver mantendo os dois
./build.sh                              # compila
adb install -r FreeMyTV.apk             # instala na TV, por cima, sem perder nada
# testar: abas, Jellyfin, Home, dólar, prateleira
git push origin main
```

## Quando

Uma vez por mês, ou quando o autor anunciar uma versão. Dá para virar uma tarefa agendada que só **avisa** quando há novidades. Instalar na TV continua pedindo aprovação.

## Cuidados aprendidos

- **Nunca instalar na TV uma versão com número de versão menor que a instalada.** O Android recusa (`INSTALL_FAILED_VERSION_DOWNGRADE`). Em 24/10, um teste da versão original deixou a TV com uma versão mais nova que a do fork, e foi preciso fazer o merge na hora para voltar.
- **A versão original desliga o YouTube Music** (está na lista de limpeza dela). Se ela for instalada, mesmo para teste, religue com `pm enable com.google.android.youtube.tvmusic`.
- Os conflitos mais comuns ficam nas traduções (`app/assets/i18n/*.json`) e no `build.sh`. Nas traduções, usar a versão do autor e acrescentar as nossas chaves.
