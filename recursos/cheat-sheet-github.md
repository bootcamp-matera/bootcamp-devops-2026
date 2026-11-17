# Cola do GitHub ☁️

Uma página para imprimir ou deixar aberta ao lado. Tudo aqui é o GitHub do bootcamp (Aulas 09 e 10), com as mensagens de erro como elas aparecem de verdade. Os comandos de Git do dia a dia estão na [cola do Git](cheat-sheet-git.md).

## 🧠 O mapa mental: dois repositórios, uma ponte

```text
        NA VM (local)                                NO GITHUB (remoto, apelido: origin)
 ┌──────────────────────────┐                      ┌──────────────────────────────────┐
 │ ~/meu-projeto-devops     │ ─── git push ──────▶ │ github.com/SEU-USUARIO/          │
 │ com o .git dentro        │ ◀── git pull ─────── │         meu-projeto-devops       │
 └──────────────────────────┘                      └──────────────────────────────────┘
   você edita e commita aqui                         o mundo vê, revisa e automatiza aqui
```

| Peça | O que é | Para olhar |
|------|---------|------------|
| **Git** | A ferramenta: guarda o histórico, na sua máquina | `git log --oneline` |
| **GitHub** | A plataforma: hospeda o repositório e soma PR, revisão e automação | A página do repositório |
| **Remoto** (`origin`) | O endereço do repositório no GitHub, anotado no repositório local | `git remote -v` |
| **`origin/main`** | A lembrança local de onde estava a `main` do GitHub no último `push`/`pull`/`fetch` | `git log --oneline --all --decorate` |
| **Token (PAT)** | A "senha" que o Git usa para falar com o GitHub — com prazo e alcance limitados | *Settings → Developer settings* |

> São **dois repositórios completos**. Commit grava só no local; nada chega ao GitHub sem `git push`, e nada chega à VM sem `git pull`.

## 🔑 O token (PAT), campo a campo

Caminho: avatar → **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**.

| Campo | Valor no bootcamp | Por quê |
|-------|-------------------|---------|
| Token name | `bootcamp-matera` | Para você reconhecer (e revogar) depois |
| Expiration | 30 dias | Segredo com prazo: se vazar, morre sozinho |
| Repository access | **Only select repositories** → `meu-projeto-devops` | O token só abre **uma** porta |
| Permissions → Contents | **Read and write** | Permite o `git push` |
| Permissions → Workflows | **Read and write** | Permite enviar a pipeline da Aula 10 |

O token aparece **uma única vez**. No `git push`: em *Username*, o seu usuário do GitHub; em *Password*, **cole o token** (nada aparece na tela — é normal) e dê Enter.

| Onde o token pode ficar | Comando | Risco |
|-------------------------|---------|-------|
| Só na sua cabeça/gerenciador de senhas | Nenhum: cole a cada push | Nenhum na máquina |
| Na memória, por 2 horas | `git config --global credential.helper 'cache --timeout=7200'` | Baixo — some sozinho |
| Em arquivo, em **texto puro** | `git config credential.helper store` (grava em `~/.git-credentials`) | Só aceitável em VM de estudo |

## 🚚 Comandos de remoto

| Comando | Faz |
|---------|-----|
| `git remote add origin URL` | Anota o endereço do GitHub no repositório local (uma vez só) |
| `git remote -v` | Mostra o endereço anotado (aparece duas vezes: fetch e push) |
| `git remote set-url origin URL` | Corrige o endereço (errou o usuário ou o nome do repositório) |
| `git push -u origin main` | **Primeiro** envio da `main`; o `-u` liga a branch local à do GitHub |
| `git push` | Envios seguintes da branch atual |
| `git push -u origin nome` | Envia uma branch nova — o primeiro passo de um Pull Request |
| `git pull` | Traz os commits novos do GitHub e integra na branch atual |
| `git fetch` | Só atualiza a lembrança (`origin/main`), sem mexer nos seus arquivos |
| `git clone URL` | Baixa um repositório inteiro, já com o `origin` configurado |
| `git branch -d nome` | Apaga a branch local depois que o PR foi integrado |

**Antes de qualquer push, três perguntas:**

```bash
git status                           # estou na branch certa? sobrou algo sem commit?
git log --oneline --all --decorate   # onde estão main, HEAD e origin/main?
git remote -v                        # para onde isto vai?
```

**Como ler o `git status` depois que existe um remoto:**

| Você vê | Significa | O que fazer |
|---------|-----------|-------------|
| `Your branch is up to date with 'origin/main'` | Local e GitHub iguais (até o último `fetch`) | Nada |
| `Your branch is ahead of 'origin/main' by 2 commits` | Você tem commits que o GitHub não tem | `git push` |
| `Your branch is behind 'origin/main' by 1 commit` | O GitHub tem commits que você não tem | `git pull` |
| `have diverged` | Os dois lados andaram | `git pull --no-rebase` e depois `git push` |

## 🔁 O ciclo do Pull Request

```text
 VM      git checkout -b docs-readme  ➜  edita  ➜  git add + git commit  ➜  git push -u origin docs-readme
                                                                                        │
 GitHub  Compare & pull request  ➜  título + o quê e por quê  ➜  Create pull request  ◀─┘
                      ➜  Files changed (a revisão)  ➜  Merge pull request  ➜  Confirm merge
                                                                                        │
 VM      git checkout main  ➜  git pull  ➜  git branch -d docs-readme                 ◀─┘
```

| Na página do PR | Para que serve |
|-----------------|----------------|
| **Conversation** | A discussão: descrição, comentários e o botão de merge |
| **Commits** | Os commits que o PR leva — confira se são **só os seus** |
| **Files changed** | O *diff*: vermelho saiu, verde entrou. É aqui que se revisa |
| **Checks** | O resultado da pipeline — ✅ ou ❌ (a partir da Aula 10) |

✍️ **PR bom:** título no imperativo (`Adiciona README do projeto`) e uma descrição que responde *o que mudou* e *por quê*. PR pequeno é revisado; PR gigante é aprovado sem ler.

> O merge do PR acontece **no GitHub**. A `main` da VM só fica sabendo depois do `git pull`.

## 🔐 Hábitos seguros (o repositório é público!)

| Hábito | Por quê | Como |
|--------|---------|------|
| Token **nunca** em arquivo do projeto, print ou chat | Robôs varrem o GitHub atrás de segredos em minutos | Cole só no prompt do `git push` |
| Vazou? **Revogue primeiro**, investigue depois | Apagar o arquivo não apaga o histórico | *Developer settings* → o token → **Delete** → gere outro |
| Olhe o `git status` antes do `git add .` | O `.` leva tudo, inclusive o que não devia | `.env`, chaves e senhas no `.gitignore` |
| Menor alcance, menor prazo | Token que abre um repositório por 30 dias causa pouco estrago | *Only select repositories* + expiração |
| Ative a verificação em duas etapas | A senha sozinha não basta para proteger o seu portfólio | *Settings → Password and authentication* |
| Fim do bootcamp: revogue o token e apague o arquivo | Segredo sem uso é só risco | `rm ~/.git-credentials` na VM |

## 🆘 Socorro

| Mensagem ou situação | O que fazer |
|----------------------|-------------|
| `Password authentication is not supported` · `Authentication failed` | Você digitou a senha da conta, ou o token está errado/vencido: cole o **token** |
| O Git nem pergunta e já falha na autenticação | Token velho guardado: apague a linha dele em `~/.git-credentials` com o `vi` e repita o push |
| `Permission ... denied` · erro `403` no push | O token não alcança este repositório ou falta *Contents: Read and write* — edite o token |
| `refusing to allow a Personal Access Token to create or update workflow` | Falta a permissão *Workflows: Read and write* no token (Aula 10) |
| `remote: Repository not found` | Erro de digitação na URL: `git remote -v` e corrija com `git remote set-url origin URL` |
| `error: remote origin already exists` | Já existe um endereço: `git remote set-url origin URL` (ou `git remote remove origin` e refaça) |
| `error: src refspec main does not match any` | Sua branch se chama `master`: `git branch -M main` e repita o push |
| `rejected ... (fetch first)` · `failed to push some refs` | O GitHub tem commits que você não tem: `git pull --no-rebase` e depois `git push` |
| Idem, logo no **primeiro** push | O repositório foi criado com README: `git pull origin main --no-rebase --allow-unrelated-histories --no-edit` e repita o push |
| `fatal: Need to specify how to reconcile divergent branches` | O `pull` quer saber como juntar: `git pull --no-rebase` |
| `fatal: The current branch xxx has no upstream branch` | Primeiro envio dessa branch: `git push -u origin xxx` |
| Dei push e o GitHub não mudou | Você está em outra branch, ou faltou o commit: `git status` e `git log --oneline --all --decorate` |
| No GitHub faltam arquivos (Dockerfile, compose...) | Os commits ficaram em outra branch: `git checkout main`, `git merge --no-edit nome-da-branch`, `git push` |
| O aviso amarelo **Compare & pull request** sumiu | Aba **Pull requests** → **New pull request** → em *compare*, escolha a sua branch |
| `There isn't anything to compare` ao abrir o PR | A branch não tem commit novo, ou não foi enviada: `git push -u origin nome` |
| O PR mostra commits que não são meus | A branch nasceu de outra branch, não da `main`. Antes de criar uma branch: `git checkout main` e `git pull` |
| Fiz o merge no GitHub e a VM não mudou | Falta sincronizar: `git checkout main` e `git pull` |
| Abriu um editor estranho no `pull`/`merge` | É o **nano** pedindo a mensagem: `Ctrl + X` sai. Para evitar, use `--no-edit` |
| Colei o token e nada apareceu | Normal: senhas não aparecem na tela. Dê Enter |
| Commitei um token ou senha 😱 | **Revogue o token já** — apagar o arquivo não o tira do histórico |

## 🏁 A Aula 09 inteira, em 10 linhas

```bash
cd ~/meu-projeto-devops && git status                                             # 1. na main, tudo commitado
git remote add origin https://github.com/SEU-USUARIO/meu-projeto-devops.git       # 2. anota o endereço
git push -u origin main                                                           # 3. primeiro envio (cole o token)
git checkout -b docs-readme                                                       # 4. branch para a melhoria
git mv README.txt README.md && vi README.md                                       # 5. renomeia e reescreve
git add README.md && git commit -m "adiciona README do projeto"                   # 6. grava
git push -u origin docs-readme                                                    # 7. envia a branch
# 8. no navegador: Compare & pull request ➜ Create pull request ➜ Merge pull request
git checkout main && git pull                                                     # 9. sincroniza a VM
git branch -d docs-readme                                                         # 10. apaga a branch ja integrada
```

Veja também: [cola do Git](cheat-sheet-git.md) · [cola do Docker e do Compose](cheat-sheet-docker.md) · [cola do terminal](cheat-sheet-terminal.md) · [glossário](../glossario.md).
