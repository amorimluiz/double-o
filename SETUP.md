# SETUP.md

Instruções para um agente configurar uma nova máquina a partir deste repositório.

## Objetivo

Este repositório é a fonte canônica da configuração de agente. Em cada máquina, os caminhos globais de cada harness devem ser **symlinks** apontando para cá. Não copie arquivos: cópias divergem, symlinks acompanham o `git pull`.

O repositório gerencia apenas o que é **dele**: instruções, rules, skills e config de harness. Artefatos gerados por libs externas (plugins, hooks, arquivos de instrução de lib) **não são versionados** — cada lib é instalada pelo seu próprio instalador, por harness.

## Pré-condições

1. O repositório está clonado em um caminho estável. Descubra-o e use como `REPO`.
   - Ex.: `C:\Users\<voce>\projects\double-o` (Windows) ou `~/projects/double-o` (Unix).
   - `REPO` é uma entrada do setup, **não** um dado versionado: o repo não guarda caminho absoluto. A fonte do caminho é o próprio clone — este `SETUP.md` fica na raiz dele.
2. Confirme que o caminho é estável. Symlinks quebram se o repositório for movido.

## Parte 1 — Artefatos do repositório (symlinks)

Crie cada link abaixo. A coluna "Harness" indica quando ele se aplica — pule harnesses que não estão instalados na máquina.

| Harness | Origem no repo | Destino (global da máquina) | Tipo |
| --- | --- | --- | --- |
| opencode | `instructions/AGENTS.md` | `~/.config/opencode/AGENTS.md` | arquivo |
| opencode | `rules/` | `~/.config/opencode/rules/` | diretório |
| opencode | `harnesses/opencode/opencode.jsonc` | `~/.config/opencode/opencode.jsonc` | arquivo |
| opencode | `harnesses/opencode/package.json` | `~/.config/opencode/package.json` | arquivo |
| agnóstico | `skills/` | `~/.agents/skills/` | diretório |

`~/.agents/skills/` é a convenção agnóstica de harness (lida por opencode, Codex e outros). Skills globais vão para lá, não para o diretório de um harness específico.

### Procedimento

Para cada linha do mapa cujo harness está instalado:

1. **Detecte o harness.** Se o diretório global do harness existe, ele está instalado.
   - opencode: `~/.config/opencode/` existe ou o comando `opencode` está no PATH.
2. **Faça backup do que já existe.** Se o destino existir e não for um symlink para `REPO`, mova-o para `<destino>.bak-<data>` antes de continuar.
3. **Crie o diretório pai** do destino, se necessário.
4. **Crie o link.**
5. **Verifique** (ver seção adiante).

O procedimento é idempotente: se o destino já for um symlink correto para `REPO`, não faça nada.

#### Windows (PowerShell)

Symlinks de diretório exigem privilégio de administrador ou Modo de Desenvolvedor. Sem isso, use **junction** para diretórios.

```powershell
# Arquivo (symlink)
New-Item -ItemType SymbolicLink -Path "<destino>" -Target "<REPO>\<origem>"

# Diretório (junction — não exige admin)
New-Item -ItemType Junction -Path "<destino>" -Target "<REPO>\<origem>"
```

#### Unix

```bash
ln -s "<REPO>/<origem>" "<destino>"
```

## Parte 2 — Ferramentas externas (instaladas pela própria ferramenta)

Estas ferramentas trazem binário e artefatos próprios. **Não** os crie nem os versione no repo: rode o instalador da ferramenta; o repo apenas **referencia** o resultado (ex.: `@RTK.md` em `instructions/AGENTS.md`). Esta tabela é o rastreio canônico das ferramentas dos agentes: cada linha diz **para que skill ou harness** a ferramenta é necessária — sem ela, a skill correspondente quebra.

| Ferramenta | Necessária para | Instalação | Verificação |
| --- | --- | --- | --- |
| RTK | integração de harness (opencode, Claude Code, Codex) | ver [RTK](#rtk) | `rtk --version` e `rtk init --show` |
| skills CLI (`npx skills`) | instalar e atualizar skills | sem install; use `npx skills` | `npx skills --version` |
| ctx7 | skill `context7` | `npm i -g ctx7@latest` | `ctx7 --version` |
| agent-browser | skills `agent-browser` e `qa-execution` | `npm i -g agent-browser && agent-browser install` | `agent-browser --version` |
| Python 3 | scripts auxiliares de `deep-review` e `qa-report` | `winget install Python.Python.3.13` (Windows) ou o gerenciador do sistema (Unix) | `python3 --version` (Windows: `python --version`) |

O lock do skills CLI é estado local da máquina e **não** é versionado. A procedência canônica e versionada das skills fica em `skills-lock.json` na raiz do repo.

### RTK

1. Instale o binário (repo `rtk-ai/rtk`) e confirme que é o Token Killer correto:
   ```bash
   rtk --version   # reporta versão
   rtk gain        # mostra o dashboard de economia
   ```
   Se `rtk gain` falhar mas `rtk --version` funcionar, você instalou o pacote errado (Rust Type Kit).
2. Rode o instalador do harness em uso, conforme a tabela acima. O plugin/hook se desativa sozinho se o binário não existir (sem erro).
3. No opencode, a integração é via plugin: **não há `RTK.md`** e o `@RTK.md` do `AGENTS.md` é inerte (opencode não expande referências `@arquivo`). Nos harnesses prompt-level (Claude Code, Codex), o instalador cria o `RTK.md` ao lado das instruções globais, e a referência `@RTK.md` resolve para ele.

### Ferramentas npm (ctx7, agent-browser)

1. Instale globalmente: `npm i -g ctx7@latest agent-browser`.
2. `agent-browser install` baixa o Chrome dedicado (~200 MB) para `~/.agent-browser/browsers/`. O npm pode bloquear o postinstall da lib (política `allow-scripts`); o `agent-browser install` explícito resolve — não é preciso liberar o script de instalação.
3. O `ctx7` funciona sem autenticação; `ctx7 login` (OAuth) ou `CONTEXT7_API_KEY` elevam o limite de uso.

### Python

`deep-review` e `qa-report` rodam scripts Python auxiliares; sem o interpretador, essas skills operam parcialmente. No Windows, `winget install Python.Python.3.13` instala por usuário e entra no PATH — abra um terminal novo depois.

### Dependências do plugin (opencode)

O diretório global do opencode precisa das dependências do plugin:

```bash
cd ~/.config/opencode && bun install
```

`package.json` é symlinkado do repo (`harnesses/opencode/package.json`) e declara `@opencode-ai/plugin`.

## Verificação

Para cada link criado:

1. O destino resolve para dentro de `REPO` (ex.: `Get-Item <destino> | Select-Object LinkType, Target` ou `ls -l`).
2. Um arquivo de teste escrito em `REPO` aparece no destino sem nova cópia.
3. O harness inicia sem erro e enxerga os artefatos:
   - opencode: `/init` não deve recriar `AGENTS.md`; skills aparecem na lista de skills disponíveis.
4. Para libs: `rtk init --show` lista a integração do harness como instalada.
5. Ferramentas externas: rode os comandos de verificação da tabela da Parte 2. Um comando que falha indica uma skill quebrada naquela máquina.

## Se algo der errado

- Restaure os `.bak-<data>` para os destinos originais.
- Remova apenas os links que você criou; nunca delete conteúdo dentro de `REPO`.
- Para desfazer uma integração de lib, use o desinstalador dela (ex.: `rtk init -g --uninstall`).
