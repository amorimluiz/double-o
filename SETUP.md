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
3. Node.js e Python 3 são pré-requisitos para as skills funcionarem: Node.js (npm/npx) para as CLIs instaladas via npm e Python 3 para os scripts auxiliares de `deep-review`, `qa-report` e `ui-ux-pro-max`. Confirme com `node --version` e `python3 --version` (Windows: `python --version`; instale com `winget install Python.Python.3.13`).

## Parte 1 — Artefatos do repositório (symlinks)

Crie cada link abaixo. A coluna "Harness" indica quando ele se aplica — pule harnesses que não estão instalados na máquina.

| Harness | Origem no repo | Destino (global da máquina) | Tipo |
| --- | --- | --- | --- |
| opencode | `instructions/AGENTS.md` | `~/.config/opencode/AGENTS.md` | arquivo |
| opencode | `rules/` | `~/.config/opencode/rules/` | diretório |
| opencode | `harnesses/opencode/opencode.jsonc` | `~/.config/opencode/opencode.jsonc` | arquivo |
| opencode | `harnesses/opencode/package.json` | `~/.config/opencode/package.json` | arquivo |
| claude | `instructions/AGENTS.md` | `~/.claude/CLAUDE.md` | arquivo |
| claude | `rules/` | `~/.claude/rules/` | diretório |
| claude | `skills/` | `~/.claude/skills/` | diretório |
| agnóstico | `skills/` | `~/.agents/skills/` | diretório |

`~/.agents/skills/` é a convenção agnóstica de harness (lida por opencode, Codex e outros). Skills globais vão para lá, não para o diretório de um harness específico. Claude Code, porém, lê apenas `~/.claude/skills/` — por isso a linha extra apontando o mesmo `skills/` para ele.

**MCPs são locais à máquina — não entram no repo.** O launch spec de um MCP contém caminho absoluto do app e, às vezes, um endpoint efêmero. Versionar isso faria o `git pull` sobrescrever o spec correto de uma máquina com o de outra. Por isso o MCP não é versionado nem symlinkado: cada ferramenta gera o próprio spec no arquivo global do harness (ver [OpenDesign](#opendesign)). No opencode, `opencode.json` e `opencode.jsonc` do diretório global são mesclados — a config portável continua vindo do repo (`harnesses/opencode/opencode.jsonc` → `~/.config/opencode/opencode.jsonc`) e só o MCP fica no arquivo local.

### Procedimento

Para cada linha do mapa cujo harness está instalado:

1. **Detecte o harness.** Se o diretório global do harness existe, ele está instalado.
   - opencode: `~/.config/opencode/` existe ou o comando `opencode` está no PATH.
   - claude: `~/.claude/` existe ou o comando `claude` está no PATH.
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
| herdr | skills `herdr` e `herdr-orchestration` | ver [herdr](#herdr) | `herdr --version` e `herdr integration status` |
| OpenDesign | skill `00-design` e MCP `open-design` (opencode e Claude Code) | ver [OpenDesign](#opendesign) | daemon de pé (`curl http://127.0.0.1:7456/api/health`) e MCP `connected` no harness |

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

### OpenDesign

App local-first que expõe um servidor **MCP stdio** com os projetos/arquivos de design. O MCP é sempre stdio e faz proxy para a API HTTP do daemon; o host precisa de um jeito de executar o CLI `od` (ou o `cli.js` equivalente) **e** do daemon de pé.

**macOS / Windows — app desktop**

1. **Instale o app desktop** em [open-design.ai](https://open-design.ai/) ou [GitHub Releases](https://github.com/nexu-io/open-design/releases). Abra-o uma vez.
2. **Gere o spec do MCP.** Em **Settings → MCP server**, selecione o harness (**OpenCode** / **Claude Code**) e copie o snippet. Ele já traz os caminhos absolutos e o endpoint do sidecar desta máquina. O app empacotado no Windows **não** põe o comando `od` no PATH ([nexu-io/open-design#4852](https://github.com/nexu-io/open-design/issues/4852)) — se o snippet citar `"command": "od"`, use o que aponta para o executável do app.
3. Salve no arquivo global do harness (OpenCode: `~/.config/opencode/opencode.json`, chave `mcp`, `type: "local"`; Claude Code: `claude mcp add-json … --scope user`). Specs de MCP são **locais à máquina** — não versione nem symlinke.
4. Reinicie o harness; o servidor aparece como `open-design`.
5. O app precisa estar aberto (parado, o MCP tenta subir uma instância headless). Ao reinstalar/atualizar, o endpoint do sidecar muda: **regenere o snippet**.

**Linux — daemon via Docker (não há artefato pré-buildado)**

O release oficial só publica macOS/Windows ([#4368](https://github.com/nexu-io/open-design/issues/4368)); o build do fonte exige Node 24 e um monorepo de ~3 GB. O caminho leve é rodar o daemon da imagem oficial e registrar o MCP via `docker exec`.

1. Docker + Compose instalados. Crie `~/.config/open-design/` com:
   - `.env` (local, não versionado): `OD_API_TOKEN=$(openssl rand -hex 32)` e `OPEN_DESIGN_PORT=7456`.
   - `docker-compose.yml`: serviço único com a imagem `ghcr.io/nexu-io/od:latest`, `ports: ["127.0.0.1:7456:7456"]`, volume `open_design_data:/app/.od`, `restart: always`. Base: [`deploy/docker-compose.yml`](https://github.com/nexu-io/open-design/blob/main/deploy/docker-compose.yml) — remova o `build:` e suba com `--no-build`.
2. Suba e valide: `docker compose up -d --no-build` → `curl -fsS http://127.0.0.1:7456/api/health`.
3. Registre o MCP. Dentro da imagem o CLI é `/app/apps/daemon/dist/cli.js` (o `od` do PATH da imagem é o BusyBox `od`), então o harness executa o MCP via `docker exec`:
   - OpenCode (`~/.config/opencode/opencode.json`):
     ```json
     "open-design": {
       "type": "local",
       "command": ["docker","exec","-i","open-design","node","/app/apps/daemon/dist/cli.js","mcp","--daemon-url","http://127.0.0.1:7456"],
       "enabled": true
     }
     ```
   - Claude Code:
     ```bash
     claude mcp add-json open-design '{"type":"stdio","command":"docker","args":["exec","-i","open-design","node","/app/apps/daemon/dist/cli.js","mcp","--daemon-url","http://127.0.0.1:7456"]}' --scope user
     ```
4. O `--daemon-url` é obrigatório: a descoberta automática via sidecar não existe no formato Docker. O daemon precisa estar de pé (o MCP faz proxy para `127.0.0.1:7456`); `restart: always` o mantém após reboot. Ao atualizar a imagem (`docker compose pull`), o MCP segue válido.

Ferramentas expostas: `create_project`, `start_run`, `get_run`, `get_artifact`, `list_skills`, `list_plugins` e os resources `od://design-systems/<id>/DESIGN.md` / `od://skills/<id>/SKILL.md`.

### herdr

Gerenciador de workspace de terminal para agentes, com servidor persistente e CLI sobre socket API. É o substrato que as skills `herdr` e `herdr-orchestration` controlam.

1. **Instale o binário.** Linux/macOS: `curl -fsSL https://herdr.dev/install.sh | sh`. Windows (PowerShell): `powershell -ExecutionPolicy Bypass -c "irm https://herdr.dev/install.ps1 | iex"`. Confirme com `herdr --version`.
2. **Instale a integração do harness em uso** para que o herdr reconheça o estado dos agentes: `herdr integration install opencode` (há alvos para claude, codex, cursor, entre outros). Confirme com `herdr integration status` — o harness em uso deve aparecer como `current`.
3. **As skills vêm do repo**, não do instalador: `herdr` e `herdr-orchestration` já chegam via symlink de `skills/`. Use `npx skills update` apenas para atualizá-las.

A integração escreve um plugin gerado (ex.: `~/.config/opencode/plugins/herdr-agent-state.js`) — artefato local à máquina, não versionado. A skill canônica também sai de `herdr --skill`; a cópia versionada em `skills/herdr/` é a mesma.

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
5. Ferramentas externas: rode os comandos de verificação da tabela da Parte 2 e os pré-requisitos da seção anterior. Um comando que falha indica uma skill quebrada naquela máquina.
6. MCP (OpenDesign): com o app aberto, o servidor `open-design` aparece no opencode (ex.: `opencode mcp list`).

## Se algo der errado

- Restaure os `.bak-<data>` para os destinos originais.
- Remova apenas os links que você criou; nunca delete conteúdo dentro de `REPO`.
- Para desfazer uma integração de lib, use o desinstalador dela (ex.: `rtk init -g --uninstall`).
