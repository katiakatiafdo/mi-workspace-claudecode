---
name: human-social
description: Human Social — desdobra uma pasta com texto + imagens em peças nativas para Instagram Feed, Instagram Stories e LinkedIn Feed. Não é resize: gera novas imagens por plataforma, com chain de referência e legenda própria para cada rede. Use quando o usuário pedir para desdobrar, adaptar ou multiplicar conteúdo para redes sociais, ou apontar uma pasta com legenda .txt e imagens. Também serve como subfluxo de outras skills.
---

# Human Social — Desdobramento

Pega uma pasta com texto + imagens e gera peças nativas para **Instagram Feed (3:4)**, **3
Instagram Stories (9:16)** e **LinkedIn Feed (16:9)**, cada uma com legenda própria. Cada peça é
um **desdobramento direto da arte-mãe** (mesma foto, fonte, cores, elementos), não um redesign.
A pessoa é fotógrafa/diretora de arte, **não é técnica**: toda configuração é conversa, nunca arquivo.

## Caminhos — leia antes de tudo

`SOCIAL_HOME` = **o diretório desta skill** (onde está este `SKILL.md`; normalmente
`~/.claude/skills/human-social/`, no Windows `%USERPROFILE%\.claude\skills\human-social\`).
Nunca assuma que é a pasta atual.

| No original | Na skill |
|---|---|
| `$SOCIAL_HOME/scripts/desdobrar.py` | `$SOCIAL_HOME/scripts/desdobrar.py` (Windows: `python "%SOCIAL_HOME%\scripts\desdobrar.py"`) |
| `$SOCIAL_HOME/CLAUDE.md`, `AGENTS.md`, `providers.md`, `COMECE-AQUI.md` | `$SOCIAL_HOME/reference/...` |
| `$SOCIAL_HOME/.claude/skills/desdobrar/SKILL.md` | `$SOCIAL_HOME/reference/desdobrar-SKILL.md` |
| saída `<pasta>/desdobramento/` | igual — **exceção da casa**: escreve na pasta que a pessoa apontar, não em `human-output/` |

**O pipeline completo, passo a passo, está em `reference/desdobrar-SKILL.md`** — leia e siga a
partir do Passo 0, aplicando a tabela acima. Contexto e princípios em `reference/CLAUDE.md`;
contrato dos motores em `reference/providers.md` (leia antes do primeiro render). Ignore, neles: a
"Regra zero" sobre não invocar a skill `human-social` (esta é ela), o `.mcp.json` local e o
gatilho "vamos começar".

## Entrada

- Se a pessoa já mandou o caminho da pasta (ou `/desdobrar <pasta>`), vá direto ao pipeline.
- Se não, apresente-se em duas linhas e peça a pasta: ela precisa ter **um `.txt`** com a legenda
  ou briefing e **uma ou mais imagens** (`.png`, `.jpg`, `.jpeg`, `.webp`). No Mac, arrastar a
  pasta do Finder para o chat cola o caminho. Depois pare e espere.
- Como **subfluxo** de outra skill: receba a pasta já pronta e rode o pipeline sem apresentação.

## Pipeline resumido

1. **Passo 0 — motor:** `python3 "$SOCIAL_HOME/scripts/desdobrar.py" check-providers` + verificar
   `mcp__magnific__*`. Resolver pela ordem abaixo.
2. `prep <pasta>` → scaffold + inputs. Abra cada imagem com Read; escolha a **arte-mãe**.
3. Prompt **curto e idêntico nos dois motores**: formato destino, texto exato, o que muda (corte,
   canvas, hierarquia, safe area). Nada de análise longa.
4. Higgsfield: `generate <pasta> <formato> <prompt> --base <arte-mãe>` (`ig-feed`, `ig-stories`,
   `linkedin-feed`; padrão `--reference-mode base-only`). Magnific: `build-prompt` → ferramenta
   MCP → `save-external` (nunca baixe na mão).
5. **Stories em trio**, cada um com texto e imagem/background próprios.
6. **3 copies diferentes do zero** (IG ≠ LinkedIn).
7. `presentation-pdf <pasta>` → `apresentacao-desdobramento.pdf`; sincronizar `desdobramento/output/`.
8. Falha em um formato não derruba os outros; `manifest.json` registra `pronto` ou `parcial`.

## Regras globais Human

- Fale com a pessoa em **português**, sem jargão técnico. Prompts para os modelos em **inglês**.
- Imagem = dois motores oficiais: **Higgsfield CLI** (aqui **`gpt_image_2`** — exceção da casa) ou
  **Magnific MCP** (image-to-image de maior qualidade). Nunca Higgs MCP, fal.ai ou Flow.
- Ordem de resolução do motor (pare no primeiro match): 1) pedido explícito; 2)
  `HUMAN_IMAGE_PROVIDER` (`higgsfield`|`magnific`); 3) só um disponível → use sem perguntar e sem
  anunciar o que faltou; 4) os dois → pergunte em uma linha, uma vez por execução ("tanto faz" →
  Higgsfield); 5) nenhum → não renderize e conduza o setup: Higgsfield
  `npm install -g @higgsfield/cli` + `higgsfield auth login`; Magnific
  `claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp`.
  Um motor por execução, sem fallback silencioso. O prompt não muda por causa do motor.
- Antes de gerar, confirme ou deduza: projeto, quantidade, formatos, resolução, referências,
  objetivo e pasta de saída.
- Saída em `<pasta-de-entrada>/desdobramento/`; finais limpos em `desdobramento/output/`.
  Nunca dentro da skill.
- Relatório final com `desdobramento/output/` em link clicável e todos os arquivos não-`.md` em
  links clicáveis (caminho absoluto), agrupados por formato.
- Confirme login/configuração antes de comandos pagos. Erro vira conversa, nunca stack trace.
- Todo comando Unix tem alternativa Windows (`python` em vez de `python3`, `%SOCIAL_HOME%`).
