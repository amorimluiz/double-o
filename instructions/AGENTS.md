# AGENTS.md

## Quem eu sou

Sou fundador e CTO. Eu dirijo, eu arquiteto, eu tomo decisões. Eu não escrevo código. Você escreve o código.

## Como eu trabalho

- Eu descrevo o que precisa ser construído em detalhe. Você executa.
- Eu tomo decisões técnicas. Você implementa.
- Não me peça pra confirmar decisões óbvias. Use julgamento. Escolha a melhor opção e siga.
- Toda sugestão minha se apoia em uma solução existente para um problema parecido: pesquise como o mercado, a comunidade ou as big techs resolveram antes de propor, sem esperar eu pedir. Cite a fonte (projeto, empresa, padrão) e explique por que ela se aplica aqui.

## Escopo

Estas instruções são globais e valem para **qualquer projeto**, salvo quando o próprio projeto declarar convenções mais específicas — nesse caso, a do projeto vence. Regras específicas de stack, domínio ou comando pertencem ao `AGENTS.md`/`CLAUDE.md` daquele projeto, nunca aqui.

## Padrões de engenharia

`rules/code-standards.md` (ao lado deste arquivo) são as regras duras de código para qualquer stack. Leia antes de escrever, alterar ou revisar código.

`rules/testing.md` (ao lado deste arquivo) são as regras duras de teste: **TDD obrigatório para nova funcionalidade** e **cobertura unitária ≥80% para código novo ou alterado**. Leia antes de escrever ou alterar qualquer teste ou funcionalidade.

`rules/git-workflow.md` (ao lado deste arquivo) são as regras duras de git: Conventional Commits, commits atômicos, GitHub Flow, merge e o que nunca entra no histórico. Leia antes de commitar, criar branch, mesclar ou abrir PR.

`rules/security.md` (ao lado deste arquivo) são as regras duras de segurança (baseline universal: authz, validação, cripto, secrets, supply chain, guardrails de IA). Leia antes de lidar com entrada externa, autenticação, autorização, criptografia, secrets ou dependências. Controles específicos de setor/compliance ficam no projeto.

`rules/documentation.md` (ao lado deste arquivo) são as regras duras de documentação: o teste de quando atualizar doc (sinal externo vs. churn), README obrigatório, tipos de doc e guardrails de doc por IA. Leia antes de escrever ou alterar documentação, e ao mudar contrato, comportamento público, config ou operação.

`rules/api-design.md` (ao lado deste arquivo) são as regras duras de design de API: contrato primeiro, evolução aditiva, erros RFC 9457, idempotência, paginação por cursor, versionamento `/v1` e interfaces para agentes. Leia antes de criar ou alterar qualquer interface pública (HTTP, RPC, GraphQL, biblioteca ou ferramenta de agente).

`rules/frontend.md` (ao lado deste arquivo) são as regras duras de frontend/UI: acessibilidade (WCAG 2.2 AA), design system e tokens antes de inventar, componentes puros, Core Web Vitals como orçamento (LCP/INP/CLS), segurança de frontend (XSS/CSP), i18n e guardrails de UI por IA. Leia antes de escrever ou alterar qualquer interface de usuário.

Novas regras entram em `rules/`, uma por tópico (`rules/observability.md`, …), sem inflar este arquivo.

## Fonte de verdade global (repo `double-o`)

O repositório `double-o` é a **single source of truth** da minha configuração de desenvolvimento assistido por agentes, em qualquer máquina. Ele é agnóstico de harness: cada máquina cria symlinks dos caminhos globais do seu harness apontando para dentro do repo.

Quando eu pedir para instalar, criar ou editar algo global de agente (skill, rule, instrução, config de harness), **sempre opere primeiro dentro do repo `double-o`**:

- **Skill:** instale/edite em `skills/<nome>/` no repo. Nunca instale direto na pasta global de skills da máquina. Registre/atualize `skills-lock.json`, a procedência canônica e versionada. O `.skill-lock.json` que o CLI de skills gera em `~/.agents/` é estado local da máquina — **não** é versionado.
- **Rule:** crie/edite em `rules/<tópico>.md` no repo.
- **Instrução global:** edite `instructions/AGENTS.md` no repo.
- **Config de harness:** edite em `harnesses/<harness>/` no repo.
- **MCP:** é **local à máquina** e **não** entra no repo nem em symlink — o launch spec embute caminho absoluto do app (e, às vezes, endpoint efêmero). Cada ferramenta gera o próprio spec no arquivo global do harness (ex.: OpenDesign → `~/.config/opencode/opencode.json`); o `SETUP.md` documenta o passo. O repo mantém só a config portável do harness.

O symlink já existente torna a mudança disponível na máquina na hora; o `git pull` propaga para as outras. Se o symlink ainda não existir na máquina, siga o `SETUP.md`. Nunca duplique conteúdo: o repo é a única cópia canônica.

### Localizar o repo

O caminho do repo muda por máquina e **não é versionado**. Descubra-o resolvendo o alvo do symlink deste arquivo, que é sempre `<repo>/instructions/AGENTS.md`:

- Windows: `(Get-Item "$env:USERPROFILE\.config\opencode\AGENTS.md").Target`
- Unix: `realpath ~/.config/opencode/AGENTS.md`

O repo é o diretório pai de `instructions/`. Se este arquivo não for um symlink, a máquina ainda não foi configurada: siga o `SETUP.md`, usando o caminho do clone como `REPO`. Nunca grave caminho absoluto dentro do repo.

**Libs externas (ex.: RTK):** o repo **não versiona** artefatos gerados por libs (plugins, `RTK.md`, hooks). Cada lib é instalada pelo seu próprio instalador, por harness, conforme o `SETUP.md`. O repo guarda apenas a referência no `AGENTS.md` (ex.: `@RTK.md`), que resolve para o arquivo que o instalador da lib cria ao lado das instruções globais do harness. Assim, atualizar a lib atualiza o artefato automaticamente, sem cópia desatualizada no repo.

## Setup em máquina nova

Quando eu clonar este repo em outra máquina e pedir o setup, execute o [`SETUP.md`](SETUP.md) da raiz até o fim e **entregue o ambiente configurado e verificado** — não pare no meio. Instalar a CLI **não** basta para as ferramentas com setup pós-instalação:

- **RTK:** depois do binário, rode o instalador do harness (`rtk init`) e confirme com `rtk init --show`.
- **ctx7:** depois do `npm i -g ctx7@latest`, autentique com `ctx7 login` (ou defina `CONTEXT7_API_KEY`) para elevar o limite de uso.

O setup só termina quando a verificação do `SETUP.md` passa: symlinks resolvendo para o repo e os comandos de verificação da tabela da Parte 2 executando sem erro. A tabela da Parte 2 é o rastreio canônico — cada ferramenta diz para que skill ou harness é necessária e como se verifica.

## Idioma

- **Documentação de agente:** este arquivo e a governança em português; `rules/*` e padrões de engenharia em inglês.
- **Código:** identificadores (variáveis, funções, métodos, classes, interfaces, types, constantes, parâmetros) sempre em **inglês**, sem exceção. Comentários e strings de exibição (labels de UI, descrições de decorators exibidas ao usuário) podem permanecer em português quando o produto for para usuários brasileiros. Campos de contrato externo (wire format de API terceira) permanecem como estão — não são nossos identificadores.

@RTK.md
