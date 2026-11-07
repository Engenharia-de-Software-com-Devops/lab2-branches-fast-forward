# Laboratório 2 — Branches e fast-forward

Prática de criação de branches, troca de contexto com `git switch`, integração por **fast-forward** (`git merge --ff-only`), exclusão de branches e uso do `.gitignore`. Duas partes:

1. **Branches e fast-forward**: alteração isolada no `README.md` numa branch `feature`.
2. **Posts do blog com branches**: um post por branch (`perfil-profissional`, `hobbies`) integrado à `main`, e regra `*.tmp` no `.gitignore`.

> Saída real de todos os comandos: [`docs/execucao-completa.log`](docs/execucao-completa.log)

## Estrutura

```
lab2-branches/
├── README.md                     # este relatório (a linha "Nova funcionalidade" da parte 1 está no fim)
├── .gitignore                    # inclui a regra *.tmp (parte 2, passo 6)
├── hugo.toml, layouts/, archetypes/   # blog Hugo herdado da aula 1
├── content/posts/
│   ├── hello-world.md            # post da aula 1
│   ├── perfil-profissional.md    # criado na branch perfil-profissional
│   └── hobbies.md                # criado na branch hobbies
└── docs/execucao-completa.log
```

## Parte 1 — Branches e fast-forward

### 1. Preparar
`main` ativa e área de trabalho limpa (`nothing to commit, working tree clean`).

### 2. Criar e alternar de branch
```bash
$ git branch feature      # cria a branch, mas continua em main
$ git branch
  feature
* main
$ git switch feature      # agora HEAD aponta para feature
```

### 3. Registrar uma alteração
Linha `Nova funcionalidade` adicionada ao fim do `README.md`:
```bash
$ git commit -m "Adiciona nova funcionalidade"
[feature 8b7e11b] Adiciona nova funcionalidade
$ git log --all --oneline --graph --decorate
* 8b7e11b (HEAD -> feature) Adiciona nova funcionalidade
* 84c5154 (main) Commit inicial: README e blog Hugo da aula 1
```
**Resultado:** `feature` avançou; `main` ficou no commit anterior.

### 4. Comparar com main
```bash
$ git switch main
$ grep -c "Nova funcionalidade" README.md
0
```
**Resultado:** na `main` a linha não existe. Ela está só no commit de `feature`.

### 5. Integrar por fast-forward
```bash
$ git merge --ff-only feature
Updating 84c5154..8b7e11b
Fast-forward
$ git log --all --oneline --graph --decorate
* 8b7e11b (HEAD -> main, feature) Adiciona nova funcionalidade
```
**O que aconteceu:** como `main` não tinha commits novos, o Git só **moveu o ponteiro** de `main` para `8b7e11b`. Nenhum commit de merge foi criado. `--ff-only` garante isso: se o fast-forward não fosse possível, o merge seria recusado.

### 6. Excluir a branch
```bash
$ git branch -d feature
Deleted branch feature (was 8b7e11b).
```
`-d` funcionou sem forçar porque o commit já estava na `main`.

## Parte 2 — Posts do blog com branches

### Post de perfil profissional
```bash
$ git switch -c perfil-profissional           # cria e já troca
$ git commit -m "Adiciona post de perfil profissional"
[perfil-profissional ba93eb4]
$ git switch main && ls content/posts
hello-world.md                                 # o post não existe na main
$ git merge --ff-only perfil-profissional
Updating 8b7e11b..ba93eb4
Fast-forward
$ git branch -d perfil-profissional
$ ls content/posts
hello-world.md  perfil-profissional.md          # o arquivo apareceu
```

### Post de hobbies
Branch criada a partir da `main` **já atualizada**, por isso o segundo merge também foi fast-forward:
```bash
$ git switch -c hobbies
$ git commit -m "Adiciona post de hobbies"
[hobbies 88608a7]
$ git switch main && git merge --ff-only hobbies
Updating ba93eb4..88608a7
Fast-forward
$ git branch -d hobbies
$ ls content/posts
hello-world.md  hobbies.md  perfil-profissional.md
```

### Ignorar arquivo temporário
```bash
$ echo "arquivo temporario" > teste-lab2.tmp
$ git status --short --untracked-files=all
?? teste-lab2.tmp                     # não rastreado
# regra *.tmp adicionada ao .gitignore
$ git check-ignore -v teste-lab2.tmp
.gitignore:4:*.tmp	teste-lab2.tmp     # arquivo:linha:regra  caminho
$ git status --ignored
Arquivos ignorados:
	teste-lab2.tmp
$ git commit -m "Ignora arquivos temporarios"
[main c13f122]
```
**Resultado:** `teste-lab2.tmp` existe localmente, mas o Git o ignora. Só o `.gitignore` foi versionado.

## Validação final

```bash
$ git branch
* main
$ git status
nothing to commit, working tree clean
$ git log --all --oneline --graph --decorate
* c13f122 (HEAD -> main) Ignora arquivos temporarios
* 88608a7 Adiciona post de hobbies
* ba93eb4 Adiciona post de perfil profissional
* 8b7e11b Adiciona nova funcionalidade
* 84c5154 Commit inicial: README e blog Hugo da aula 1
```

Histórico **linear**: nenhum commit de merge, só fast-forwards.

## Critérios de conclusão

- [x] `main` ativa e contém a alteração da parte 1 e os dois posts
- [x] Os três merges foram fast-forward, sem commits de merge
- [x] `feature`, `perfil-profissional` e `hobbies` foram removidas
- [x] O commit da parte 1 continua no histórico de `main`
- [x] `git check-ignore -v teste-lab2.tmp` identifica a regra `.gitignore:4:*.tmp`
- [x] `git status` limpo

## Conceitos

| Comando | O que faz |
|---|---|
| `git branch <nome>` | Cria uma branch (um ponteiro para o commit atual) sem trocar de branch |
| `git switch <nome>` | Troca a branch ativa (move o `HEAD`) |
| `git switch -c <nome>` | Cria e troca numa só operação |
| `git merge --ff-only <b>` | Integra só se der para avançar o ponteiro; caso contrário, recusa |
| `git branch -d <b>` | Exclui uma branch já integrada |
| `git check-ignore -v <arq>` | Mostra qual regra do `.gitignore` ignora o arquivo |
| `git status --ignored` | Lista também os arquivos ignorados |

## Observações
- O repositório base reproduz o da aula 1 (README + blog Hugo do [git-workshop](https://github.com/Engenharia-de-Software-com-Devops/git-workshop)).
- Depois da validação, este README foi reescrito como relatório num commit posterior. A linha da parte 1 está preservada no fim.

## Referências
- [git-branch](https://git-scm.com/docs/git-branch) · [git-merge](https://git-scm.com/docs/git-merge) · [gitignore](https://git-scm.com/docs/gitignore)

---

Nova funcionalidade
