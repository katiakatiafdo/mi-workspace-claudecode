---
name: human-team
description: Human Team — time de agentes criativos (OpenSquad) que transforma uma ideia, referência ou material existente em produção completa: brief, conceito, roteiro, direção de arte, storyboard, imagens (teaser/principais/secundárias), anúncios 9:16/4:5/16:9, copy-pack, calendário e handoff. Use quando o usuário pedir squad criativa, pipeline multiagente, time de agentes, campanha completa ou produção criativa end-to-end. Não abre menu genérico: inicia a squad criativa padrão.
---

# Human Team — Time de Agentes Criativos

10 agentes que transformam uma ideia em campanha pronta (brief, conceito, roteiro, direção de arte,
storyboard, KV, imagens, anúncios, copy, calendário e handoff), rodando sobre o framework
multiagente **OpenSquad**. A pessoa pode nunca ter usado Claude Code: conduza sem jargão.

## Caminhos — leia antes de tudo

`SKILL_DIR` = o diretório desta skill (onde está este `SKILL.md`; normalmente
`~/.claude/skills/human-team/`, no Windows `%USERPROFILE%\.claude\skills\human-team\`).
`WORK` = `human-output/team/` na **pasta atual da pessoa** (estado e campanhas; nunca dentro da skill).

| No original | Na skill (somente leitura) |
|---|---|
| `.claude/commands/team.md` (o fluxo `/team`) | `$SKILL_DIR/reference/team.md` |
| `.claude/skills/opensquad/SKILL.md` | `$SKILL_DIR/reference/opensquad-SKILL.md` |
| `squads/team/` (squad.yaml, squad-party.csv, agents/, pipeline/) | `$SKILL_DIR/assets/squads/team/` |
| `_opensquad/core/` | `$SKILL_DIR/assets/_opensquad/core/` |
| `skills/` (image-ai-generator, canva, instagram-publisher, …) | `$SKILL_DIR/assets/skills/` |
| `scripts/*.js`, `package.json` | `$SKILL_DIR/assets/scripts/`, `$SKILL_DIR/assets/package.json` |

| No original | Na pasta da pessoa (gravável) |
|---|---|
| `_opensquad/_memory/company.md`, `preferences.md` | `WORK/_memory/company.md`, `WORK/_memory/preferences.md` |
| `squads/team/_memory/memories.md`, `runs.md` | `WORK/_memory/team/memories.md`, `WORK/_memory/team/runs.md` |
| `squads/team/state.json` (dashboard) | `WORK/state.json` (o dashboard não vem na skill; o arquivo é só registro) |
| `Campanhas/{campaign_slug}/` | `WORK/{campaign_slug}/` — um run por pasta |

**Primeira vez em uma pasta** (se `WORK/_memory/` não existir), copie os modelos:
`mkdir -p human-output/team/_memory/team && cp "$SKILL_DIR"/assets/_opensquad/_memory/*.md human-output/team/_memory/ && cp "$SKILL_DIR"/assets/squads/team/_memory/*.md human-output/team/_memory/team/`
— Windows: `New-Item -ItemType Directory -Force human-output\team\_memory\team; Copy-Item "$SkillDir\assets\_opensquad\_memory\*.md" human-output\team\_memory\; Copy-Item "$SkillDir\assets\squads\team\_memory\*.md" human-output\team\_memory\team\`.

**Pasta da campanha:** crie `WORK/{campaign_slug}/` com a mesma árvore e o mesmo `README.md` que
`assets/scripts/create-campaign-folder.js` gera (leia o script: `input/`, `refs/{kv,marca,produto}/`,
`documentos/`, `final/...`, `internal/generation/...`). Não rode o script direto — ele grava em
`./Campanhas/`.

## Por onde começar

**O fluxo completo é `reference/team.md` — leia e siga**, com as tabelas acima. Contexto e regras
em `reference/CLAUDE.md`, `reference/AGENTS.md`, `reference/README.md`. Ignore, neles: a frase de
que skills globais `human-team`/`opensquad` não substituem a pasta (esta skill **é** a pasta
empacotada) e o gatilho "vamos começar" (aqui o gatilho é o pedido da pessoa).

## Entrada

1. Não abra menu genérico.
2. Se `WORK/_memory/company.md` contiver `<!-- NOT CONFIGURED -->`, faça o setup curto, uma
   pergunta por vez (nome e idioma → `preferences.md`; marca e site → WebFetch/WebSearch → resumo
   para confirmar → `company.md` sem a linha NOT CONFIGURED). Se a pessoa quiser pular, siga e
   colete o mínimo no brief.
3. Na primeira resposta de toda run, apresente os **10 agentes** (tabela "Time envolvido" de
   `reference/team.md`) antes de pedir briefing. Se a pessoa já trouxe a demanda, diga em uma frase
   quais agentes entram primeiro e pergunte só o que falta.
4. Informe a pasta da campanha assim que for criada (`human-output/team/{slug}/`, refs de KV em
   `refs/kv/`, entregas em `final/`).

## Regras do produto

- Carregue `WORK/_memory/company.md` antes de qualquer run. Não pule checkpoints; pause antes de
  ação paga, irreversível ou de publicação.
- Markdown técnico vai para `internal/`; a pessoa aprova pelo PDF em `documentos/`.
- PDF de aprovação: `node "$SKILL_DIR/assets/scripts/render-project-document.js" "human-output/team/{slug}"`.
  Se faltar dependência, rode uma vez dentro de `$SKILL_DIR/assets`: `npm install` e
  `npx playwright install chromium`.
- Imagens: `python3 "$SKILL_DIR/assets/skills/image-ai-generator/scripts/generate.py" --check-providers`
  (Higgsfield) + ferramentas `mcp__magnific__*`. KV/lettering usa `gpt_image_2`
  (`--asset-type kv`); imagens soltas `nano_banana_2`. Sem motor → entregue prompt + direção visual
  e siga o resto da run.
- Playwright MCP e Notion MCP são **opcionais** — cite quando úteis, nunca exija.
- Registre aprendizados em `WORK/_memory/team/memories.md` e a execução em
  `WORK/_memory/team/runs.md`.

## Regras globais Human

- Fale com a pessoa em **português** (salvo preferência em `preferences.md`). Prompts em **inglês**.
- Imagem = dois motores oficiais: **Higgsfield CLI** (`nano_banana_2`; `gpt_image_2` para KV) ou
  **Magnific MCP** (modelo de maior qualidade; image-to-image com referência). Nunca Higgs MCP,
  fal.ai ou Flow. Vídeo exclusivo do Higgsfield CLI.
- Ordem de resolução do motor (pare no primeiro match): 1) pedido explícito; 2)
  `HUMAN_IMAGE_PROVIDER` (`higgsfield`|`magnific`); 3) só um disponível → use sem perguntar;
  4) os dois → pergunte em uma linha, uma vez por run; 5) nenhum → não renderize, salve os prompts
  e conduza o setup. Nunca troque de motor no meio de um batch, sem fallback silencioso. O prompt
  não muda por causa do motor. Contrato em `reference/providers.md`; resultados do Magnific passam
  por `generate.py --save-external --from-url|--from-file … --output …` (ver `providers.md`) para
  cair na mesma estrutura, com metadata e log.
- Magnific em qualquer projeto: se escolhido e não registrado, oriente uma vez:
  `claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp`.
- Antes de gerar asset visual, confirme ou deduza: projeto, quantidade, aspect ratio, resolução,
  referências, objetivo de uso e pasta de saída.
- Outputs em `human-output/team/{run}/` na pasta atual da pessoa. Nunca dentro da skill.
- Relatório final com a pasta em link clicável (caminho absoluto) e cada arquivo não-`.md` em
  link clicável.
- Confirme configuração/credenciais antes de comandos pagos ou com login. Sem stack trace.
- Todo comando Unix tem alternativa Windows.
