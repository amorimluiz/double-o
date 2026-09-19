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

## Parte 2 — Libs externas (instaladas pela própria lib)

Estas libs trazem binário + artefatos de integração próprios. **Não** os crie nem os versione no repo: rode o instalador da lib para o harness em uso. O repo apenas **referencia** o artefato resultante (ex.: `@RTK.md` em `instructions/AGENTS.md`), de modo que atualizar a lib atualiza o artefato automaticamente.

| Lib | Harness | Instalação | Artefato gerado |
| --- | --- | --- | --- |
| RTK | opencode | `rtk init -g --opencode` | `~/.config/opencode/plugins/rtk.ts` |
| RTK | Claude Code | `rtk init -g` | `~/.claude/RTK.md` + `@RTK.md` no `CLAUDE.md` |
| RTK | Codex | `rtk init -g --codex` | `~/.codex/RTK.md` + `@RTK.md` no `AGENTS.md` |

### RTK

1. Instale o binário (repo `rtk-ai/rtk`) e confirme que é o Token Killer correto:
   ```bash
   rtk --version   # reporta versão
   rtk gain        # mostra o dashboard de economia
   ```
   Se `rtk gain` falhar mas `rtk --version` funcionar, você instalou o pacote errado (Rust Type Kit).
2. Rode o instalador do harness em uso, conforme a tabela acima. O plugin/hook se desativa sozinho se o binário não existir (sem erro).
3. No opencode, a integração é via plugin: **não há `RTK.md`** e o `@RTK.md` do `AGENTS.md` é inerte (opencode não expande referências `@arquivo`). Nos harnesses prompt-level (Claude Code, Codex), o instalador cria o `RTK.md` ao lado das instruções globais, e a referência `@RTK.md` resolve para ele.

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

## Se algo der errado

- Restaure os `.bak-<data>` para os destinos originais.
- Remova apenas os links que você criou; nunca delete conteúdo dentro de `REPO`.
- Para desfazer uma integração de lib, use o desinstalador dela (ex.: `rtk init -g --uninstall`).
