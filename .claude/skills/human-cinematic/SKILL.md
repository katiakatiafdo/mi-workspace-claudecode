---
name: human-cinematic
description: Human Cinematic — produção cinematográfica: campanhas, product shots impossíveis, roteiros, character sheets, frames por cena e vídeos via Higgsfield (Seedance, Nano Banana 2, Kling); imagens também podem sair pelo Magnific MCP. Use quando o usuário pedir filme, curta, campanha visual, vídeo cinematográfico, product shot premium de estúdio, ou conduzir da ideia até frames aprovados e vídeo. Regra-chave: vídeo só depois de frames aprovados.
---

# Human Cinematic

Sistema de automação de vídeo e imagem cinematográfica: você escreve e dispara os prompts, o
**Higgsfield CLI** (Seedance 2.0, Nano Banana 2, Kling 3.0) renderiza. Imagens também podem sair
pelo **Magnific MCP**. **Vídeo é sempre Higgsfield.**

## Caminhos — leia antes de tudo

`SKILL_DIR` = o diretório desta skill (onde está este `SKILL.md`; normalmente
`~/.claude/skills/human-cinematic/`, no Windows `%USERPROFILE%\.claude\skills\human-cinematic\`).

| No original | Na skill |
|---|---|
| `COMECE-AQUI.md`, `PRODUCT-SHOTS.md`, `SCRIPT_AI_SYSTEM.md`, `seedance-prompt-framework.md`, `providers.md`, `SETUP-GUIDE.md`, `README.md` | `$SKILL_DIR/reference/...` |
| `_template/` | `$SKILL_DIR/assets/_template/` (modelo-mestre — **nunca edite**) |
| `campaigns/{nome}/` (e `Human Cinematic/campaigns/{nome}/`) | `human-output/product/{nome}/` na **pasta atual da pessoa** |
| `campaigns/README.md` | `$SKILL_DIR/reference/campaigns-README.md` |

Criar campanha nova (Parte 1): `mkdir -p human-output/product && cp -R "$SKILL_DIR/assets/_template" "human-output/product/{nome}"`
— Windows (PowerShell): `New-Item -ItemType Directory -Force human-output\product; Copy-Item -Recurse "$SkillDir\assets\_template" "human-output\product\{nome}"`.
Depois crie também `human-output/product/{nome}/output/` (resultados numerados e limpos).

## Por onde começar

O roteiro mestre é `reference/COMECE-AQUI.md` — **leia e siga**, com a tabela de caminhos acima.
As regras do produto estão em `reference/CLAUDE.md`. Ignore, neles: a "Regra zero" sobre não
invocar a skill `human-cinematic` (esta é ela), a exigência de ler `.mcp.json` local e o gatilho
"vamos começar".

**Entrada:** se a pessoa já trouxe ideia, roteiro, produto ou referência, pule o Menu e vá direto
à Parte correspondente do `COMECE-AQUI.md`. Só mostre o Menu (em português, apresentando o sistema
como **Human Cinematic**) quando ela não disser o que quer.

## Regras do produto (resumo de `reference/CLAUDE.md`)

- **Idiomas:** português com a pessoa; prompt do **Seedance em chinês (中文)** (ver
  `seedance-prompt-framework.md` → "Delivery language"); prompts de imagem (Nano Banana 2) em inglês.
- **Roteiro antes da campanha** (Parte R): sem história fechada → modo Script AI
  (`reference/SCRIPT_AI_SYSTEM.md`); roteiro fechado vai para `internal/roteiro.md`.
- **Product Shots** (still, packshot, hero product, anúncio estático): modo
  `reference/PRODUCT-SHOTS.md` antes do wizard longo.
- **A trava:** nunca gere vídeo sem **um frame aprovado por cena** (Parte 4). Só a pessoa dispensa,
  explicitamente.
- **Character sheet** de todo personagem recorrente (Parte 3), com ângulo de corpo inteiro e
  grão/textura de câmera, personagem com personalidade.
- `output/` limpo (resultados numerados); todo o resto em `internal/`. Feedback por chat,
  registrado em `internal/feedback.md`.
- **Geração em fila:** com mais de uma peça, submeta o batch inteiro
  (`higgsfield generate create … --json` sem `--wait`) e depois colete
  (`higgsfield generate wait <job_id>`). Repita as referências em todas as submissões.

## Regras globais Human

- Fale com a pessoa em **português**. Prompts em **inglês**, exceto Seedance em **chinês**.
- Imagem = dois motores oficiais: **Higgsfield CLI** (`nano_banana_2`) ou **Magnific MCP**
  (modelo de maior qualidade; image-to-image quando houver referência). Nunca Higgs MCP, fal.ai
  ou Flow. **Vídeo exclusivo do Higgsfield CLI.**
- Ordem de resolução do motor de imagem (pare no primeiro match): 1) pedido explícito;
  2) `HUMAN_IMAGE_PROVIDER` (`higgsfield`|`magnific`); 3) só um disponível → use sem perguntar;
  4) os dois → pergunte em uma linha, uma vez por execução; 5) nenhum → não renderize, salve os
  prompts e conduza o setup. Nunca troque de motor no meio de um batch, sem fallback silencioso.
  O prompt não muda por causa do motor. Detalhes em `reference/providers.md` — leia antes de
  renderizar imagem.
- Magnific em qualquer projeto: se escolhido e não registrado, oriente uma vez:
  `claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp`.
  Resultados do Magnific são salvos na mesma estrutura da campanha, com registro em
  `internal/prompt-log.md` (motor, ferramenta, prompt, parâmetros, URL) — nunca soltos.
- Antes de gerar, confirme ou deduza: nome do projeto, quantidade, aspect ratio, resolução,
  referências, objetivo de uso e pasta de saída.
- Outputs em `human-output/product/{slug}/` (com `internal/` + `output/`) na pasta atual da
  pessoa. Nunca dentro da skill.
- Relatório final com a pasta da campanha em link clicável e todos os arquivos não-`.md` em links
  clicáveis (caminho absoluto).
- Confirme instalação/login do Higgsfield antes de comandos pagos (Parte 0). Sem stack trace.
- Todo comando Unix tem alternativa Windows.
