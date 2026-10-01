---
name: desdobrar
description: Desdobra uma pasta com texto + imagens em peças nativas para Instagram Feed, Instagram Stories e LinkedIn Feed via Higgsfield CLI (GPT Image 2) ou Magnific MCP, com chain de referência. Pre-flight conversacional do motor de imagem (usuário nunca toca arquivo). Use quando o usuário digitar `/desdobrar <pasta>` ou pedir para "desdobrar carrossel" / "criar versões pra cada plataforma" a partir de um material local.
---

# /desdobrar — Pipeline com onboarding zero-técnico

Uso:
```
/desdobrar /caminho/da/pasta
```

**Importante:** o usuário deste sistema NÃO É TÉCNICO. É fotógrafo/diretor de arte. Ele não sabe o que é `.env`, "chave de API", "endpoint" ou "variável de ambiente". Toda interação com ele é em linguagem 100% humana. Erros técnicos viram conversa de WhatsApp, não stack trace.

---

## Passo 0 — Pre-flight do motor de imagem (CONVERSACIONAL — antes de qualquer outra coisa)

Resolva `SOCIAL_HOME` antes de rodar qualquer comando: e a raiz da pasta do Human Social, onde ficam o `CLAUDE.md`, o `providers.md` e a pasta `scripts/`. Se o Claude Code foi aberto la dentro, e `.`; se foi aberto em outro projeto, e o caminho absoluto ate essa pasta. O script **nao** fica no projeto atual do usuario:

```bash
SOCIAL_SCRIPT="$SOCIAL_HOME/scripts/desdobrar.py"
test -f "$SOCIAL_SCRIPT"
```

Existem **dois motores oficiais**: Higgsfield CLI e Magnific MCP. A regra completa esta em `$SOCIAL_HOME/providers.md` — leia antes do primeiro render. Roda:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" check-providers
```

Isso responde o status do **Higgsfield**. O status do **Magnific** você mesmo verifica: olhe se existem ferramentas `mcp__magnific__*` na sessão (se estiverem diferidas, use `ToolSearch` com a query `magnific`).

Agora resolva o motor **nesta ordem**, parando no primeiro match:

### 1. O usuário já disse qual quer
"usa o Magnific", "roda pelo Higgsfield" — respeite e siga.

### 2. `preferred_provider_env` veio preenchido
A variável `HUMAN_IMAGE_PROVIDER` trava o motor. Siga sem perguntar.

### 3. Só um está disponível
Use ele e siga pro Passo 1. **Não pergunte nada e não anuncie o que faltou** — pro usuário, isso é ruído.

### 4. Os dois estão disponíveis — PERGUNTE
Uma linha, uma vez só, antes de gerar qualquer peça:

> Posso gerar as imagens por dois motores: **Higgsfield** ou **Magnific**. Qual você prefere?

Espere a resposta. Se ele disser "tanto faz" ou "escolhe você", use o **Higgsfield** e siga sem alongar. **Guarde a escolha para a execução inteira** — não pergunte de novo a cada Story.

### 5. Nenhum disponível — conduza o setup, sem jargão
Não tente renderizar e não prometa arquivo. Explique em linguagem humana e ofereça os dois caminhos:

> Antes de começar, preciso conectar um motor de imagem aqui no Claude Code — é ele que vai gerar as imagens novas pra cada rede. Tem dois caminhos, você escolhe:
>
> **Higgsfield** — eu instalo com `npm install -g @higgsfield/cli` e depois faço o login com você. Se faltar Node.js, eu te aviso e paro.
>
> **Magnific** — você roda `claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp` uma vez, reabre a sessão e autoriza o login.

Se o Higgsfield estiver instalado mas sem login (`status: "login_required"`), é só:

> O Higgsfield está instalado, só falta conectar a conta. Vou abrir o login com `higgsfield auth login`; conclui no navegador e me avisa.

```bash
higgsfield auth login
```

Depois rode `check-providers` de novo. Se voltar `ok`, responde **"Conectado. Vamos seguir."** e segue pro Passo 1.

**Nunca troque de motor no meio da execução**, nem como fallback de erro. Se o Feed saiu no Higgsfield, Stories e LinkedIn saem no Higgsfield também.

---

## Passo 1 — Prep

Roda:
```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" prep "<pasta>"
```

Saída em JSON com:
- `texto_arquivo` — path do `.txt` encontrado
- `texto_conteudo` — conteúdo lido
- `imagens` — paths absolutos das imagens
- `saida` — path de `desdobramento/`
- `output` — pasta limpa de entrega em `desdobramento/output/`
- `manifest` — path do `manifest.json`

Se faltar `.txt` ou imagens, o script aborta com mensagem clara — repassa pro usuário com tom amigável.

---

## Passo 2 — Escolher a arte-mãe

Use o **Read tool** nas imagens originais e escolha UMA arte-mãe.

Prioridade:
1. imagem já diagramada com foto/visual + lettering;
2. imagem que mais parece peça final;
3. imagem principal mais forte, se ainda não existir arte com lettering.

Essa mesma arte-mãe vai para IG Feed, Stories e LinkedIn. Não escolha uma base diferente por rede, porque isso cria peças com mundos visuais diferentes.

Anote só o essencial:
- foto/visual principal;
- estilo da fonte;
- cores;
- elementos gráficos;
- logo/assinatura;
- composição e hierarquia.

Regra operacional: toda geração visual recebe a arte-mãe como primeira referência (`--base`), nos dois motores. O script usa `--reference-mode base-only` por padrão para não deixar outras imagens confundirem o modelo. Use `--reference-mode all` só se o usuário pedir ou se as outras imagens forem claramente parte do mesmo sistema visual. No Higgsfield, o modelo é `gpt_image_2`.

Prompt bom aqui é curto. O modelo já vê a arte. Não escreva briefing longo nem descreva tudo que está na imagem: diga só o formato destino, o texto que entra e o que pode mudar. Em geral, 4 a 7 linhas bastam. Reforce que os elementos visíveis da arte-mãe continuam: foto, fundo, fonte, logo, grafismos, cores e composição. Para Stories, o foco muda: preserve identidade e personagem/produto, mas peça variação real de background/cena entre os frames.

Para ajuste pontual em uma peça já gerada, use a própria peça como `--base` e peça só a troca:

```
Use the attached artwork.
Keep everything the same.
Only replace the headline with: "{NOVA_HEADLINE}"
Preserve photo, background, font style, colors, logo and layout.
```

---

## Passo 2B — Como renderizar (vale para os Passos 3, 4 e 5)

Os Passos 3 a 5 dizem **o que** gerar e **qual prompt escrever**. O **como** depende do motor
que você resolveu no Passo 0. Escreva o prompt em `_prompts/` do mesmo jeito nos dois casos —
o prompt não muda por causa do motor.

### Caminho A — Higgsfield CLI

Um comando por peça:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" generate "<pasta>" <formato> "<prompt-file>" --base <arte-mae.png> [--output <nome.png>]
```

O script sobe as imagens com `higgsfield upload create`, chama `higgsfield generate create gpt_image_2`, espera com `higgsfield generate wait`, baixa o PNG no caminho certo e escreve o log. Retorna JSON com `output_path`, `higgsfield_url`, `base_reference` e `reference_files`.

### Caminho B — Magnific MCP

Dois passos por peça. **Não pule o primeiro.**

**1. Pegue o prompt final.** O `generate` enriquece o seu prompt com as regras de fidelidade à arte-mãe (e, nos Stories, com a regra de variação do frame). Se você mandar o `_prompts/*.txt` cru pro Magnific, a peça sai fora do padrão:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" build-prompt "<pasta>" <formato> "<prompt-file>" --base <arte-mae.png> [--output <nome.png>]
```

Devolve JSON com `prompt`, `base_reference`, `reference_files`, `dimensions` e `output_path`. Use **esse** `prompt`.

**2. Gere e salve.** Chame a ferramenta de geração `mcp__magnific__*` (descubra a assinatura em runtime; se estiver diferida, `ToolSearch` com a query `magnific`) passando o `prompt` e a `base_reference` como imagem de referência — a referência é o que segura a identidade, é obrigatória. Depois grave o retorno no padrão da casa:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" save-external "<pasta>" <formato> \
  --url "<url-retornada>" [--output <nome.png>] \
  --provider magnific_mcp --model "<nome_da_ferramenta>" \
  --base <arte-mae.png> --prompt-file "<prompt-file>"
```

Se o MCP gravou arquivo local, troque `--url` por `--file "/caminho/do/arquivo.png"`.

**Nunca baixe o resultado do Magnific na mão.** O `save-external` é o que garante nome, pasta e log corretos — e é desse log que o PDF final tira qual motor gerou a entrega.

---

## Passo 3 — Gerar Instagram Feed (1 imagem, 3:4)

Escreve um prompt curto em `<pasta>/desdobramento/_prompts/ig-feed.txt`. Use a arte-mãe e peça só a transformação:

```
Use the attached master artwork.
Convert it to Instagram Feed portrait 3:4 (1080x1440).
Keep the same photo/background, font style, colors, graphic elements, logo and composition.
Change only crop, spacing, hierarchy and text. Keep all important visual elements.
Text: "{HEADLINE_CURTA}" / "{SUPORTE_CURTO opcional}"
No redesign. No new photo. No new style.
```

Depois renderize pelo caminho do Passo 2B, com `formato = ig-feed`.

Higgsfield:
```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" generate "<pasta>" ig-feed "<pasta>/desdobramento/_prompts/ig-feed.txt" --base <arte-mae.png>
```

Magnific: `build-prompt` com os mesmos argumentos → ferramenta `mcp__magnific__*` → `save-external "<pasta>" ig-feed --url ...`.

Nos dois casos o arquivo final é `<pasta>/desdobramento/instagram-feed.png`.

Depois do render, compare com a arte-mãe. Se a foto, fonte, paleta ou linguagem mudaram demais, reprove e regenere com prompt ainda mais direto: "make it much closer to the attached master artwork; only change the format and text".

---

## Passo 4 — Gerar Instagram Stories (sempre 3 frames, 1080×1920)

Stories também nascem da mesma arte-mãe. Gere sempre 3 frames. Não use uma imagem-base diferente por story, a não ser que o usuário peça uma sequência baseada em várias imagens.

Cada story precisa trazer uma informação diferente e uma imagem/background diferente. O problema a evitar é xerox com texto trocado. A imagem por trás do texto não pode permanecer igual nos três frames. Pode variar crop, fundo estendido, área sólida para texto, respiro de layout, ângulo, pose, iluminação, situação próxima, cenário derivado ou personagem/produto em outro enquadramento; não pode trocar marca, fonte, logo, paleta nem linguagem principal.

Antes de escrever os prompts, defina um plano de variação:

- Story 01: frame mais próximo da arte-mãe, mas com crop, profundidade, extensão de fundo ou iluminação ajustada. Não pode ser só a mesma foto com texto novo.
- Story 02: variação visual mais clara do trio: outro ângulo, pose, cenário derivado, background diferente ou área sólida forte com cor da marca.
- Story 03: fechamento/CTA com outro background, composição diferente, iluminação diferente ou área de texto sólida; ainda dentro da mesma campanha.

Escreva `<pasta>/desdobramento/_prompts/story-NN.txt` com prompt curto:

```
Use the attached master artwork.
Convert it to Instagram Stories 9:16 (1080x1920).
Keep the same campaign identity: brand/logo, font style, colors and graphic language.
Create a derivative variation, not a copy. The background/image must be visibly different from the other Stories.
Change vertical crop/extension, safe area, text placement, background/image and text.
Use one: alternate angle, pose, related setting, changed lighting, background extension or solid brand-color field.
Text: "{HEADLINE_DO_STORY}" / "{MICROCOPY opcional}"
No unrelated new objects. No new brand style. Do not keep the exact same background photo with only new text.
```

Renderize os três pelo caminho do Passo 2B, com `formato = ig-stories` e `--output story-NN.png`, sempre com a mesma arte-mãe.

Higgsfield — em paralelo:
```bash
# em paralelo
for N in 01 02 03; do
  python3 "$SOCIAL_HOME/scripts/desdobrar.py" generate "<pasta>" ig-stories "<pasta>/desdobramento/_prompts/story-$N.txt" --base "<arte-mae.png>" --output "story-$N.png" &
done
wait
```

Magnific — três `build-prompt` (um por frame, cada um com o seu `--output story-NN.png`, porque é o `--output` que ativa a regra de variação daquele frame), três chamadas à ferramenta, três `save-external`. Pode disparar as chamadas do MCP em paralelo.

Se algum falhar, segue com os outros — registra no manifest.

---

## Passo 5 — Gerar LinkedIn Feed (1 imagem, 16:9)

LinkedIn não ganha outra direção visual. Ele é uma adaptação da mesma arte-mãe para um registro mais editorial/sóbrio, mantendo foto, fonte, cores e elementos reconhecíveis. Escreve o prompt em `<pasta>/desdobramento/_prompts/linkedin-feed.txt`:

```
Use the attached master artwork.
Convert it to LinkedIn Feed landscape 16:9 (1920x1080).
Keep the same photo/background, font style, colors, graphic elements, logo and composition.
Change only crop, horizontal canvas, spacing, readability and text. Keep all important visual elements.
Text: "{TAG opcional}" / "{HEADLINE_LINKEDIN}" / "{SUPORTE_LINKEDIN opcional}"
No redesign. No new photo. No new style.
```

Renderize pelo caminho do Passo 2B, com `formato = linkedin-feed`.

Higgsfield:
```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" generate "<pasta>" linkedin-feed "<pasta>/desdobramento/_prompts/linkedin-feed.txt" --base <arte-mae.png>
```

Magnific: `build-prompt` com os mesmos argumentos → ferramenta `mcp__magnific__*` → `save-external "<pasta>" linkedin-feed --url ...`.

Salva em `<pasta>/desdobramento/linkedin-feed.png`.

---

## Passo 6 — Escrever todas as copies finais

Leia o `texto_conteudo` original (do Passo 1). Escreva **todos os textos finais da entrega**, não só imagens:

### `<pasta>/desdobramento/instagram-feed.txt`
- Gancho nas 2 primeiras linhas (antes do "...ver mais")
- Frases curtas, parágrafos curtos, espaço entre eles
- 60-120 palavras
- CTA leve (1 linha)
- 3-7 hashtags no final, depois de linha em branco
- Tom oral, rítmico, social

### `<pasta>/desdobramento/instagram-stories/roteiro.txt`
Exatamente 3 seções, uma por story, com copy de tela, sticker sugerido e micro-CTA:
```
STORY 01 — story-01.png
Headline na imagem: "QUEM PRECISA DE PERFEIÇÃO MORRE"
Sticker sugerido: caixa de pergunta
Micro-CTA: "responde aqui ↓"

STORY 02 — story-02.png
Headline: "Eles pararam de ter ideias em 2018"
Sticker: (nenhum)

... etc

STORY {N} — story-NN.png  (último)
Headline: "Carrossel completo no feed →"
Sticker: link sticker apontando pro post do feed
```

### `<pasta>/desdobramento/linkedin-feed.txt`
- Insight substantivo na primeira linha (NÃO gancho-clickbait)
- 1ª pessoa ou voz declarativa direta
- 160-320 palavras
- Parágrafos articulados (3-5 linhas cada)
- Inclui pelo menos 1 número / citação / exemplo concreto
- CTA editorial (pergunta aberta) ou nenhum
- 2-4 hashtags no final
- Tom editorial-substantivo

**Crítico:** depois de escrever as 3, compare. As copies podem ter registros diferentes, mas as imagens precisam continuar parecendo desdobramentos da mesma arte-mãe.

---

## Passo 7 — Atualizar manifest

Lê `<pasta>/desdobramento/manifest.json`, preenche `outputs` com os paths e as URLs que vieram do JSON do `generate` (Higgsfield) ou do `save-external` (Magnific), preenche `provider` com o motor realmente usado (`higgsfield_cli` ou `magnific_mcp`), marca `status: "pronto"` ou `"parcial"`. Salva.

---

## Passo 8 — Gerar PDF de apresentação

Depois de gerar imagens e copies, rode:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" presentation-pdf "<pasta>"
```

O arquivo final fica em:

```text
<pasta>/desdobramento/apresentacao-desdobramento.pdf
```

O PDF precisa mostrar:
- texto-base recebido;
- imagens-base usadas como referência;
- Instagram Feed gerado + legenda;
- Stories gerados + roteiro;
- LinkedIn Feed gerado + legenda;
- motor e modelo usados e referências enviadas — o script descobre isso sozinho lendo os logs de `_logs/`, então não há nada a ajustar aqui.

O PDF não deve listar nomes de arquivo na página. Ele deve parecer uma apresentação simples e bonita: layout 16:9, imagens grandes, respiro, hierarquia tipográfica, acentos corretos e textos legíveis. Cada página precisa deixar claro se é Base, Instagram Feed, Instagram Stories, roteiro dos Stories, LinkedIn ou entrega limpa.

O script também cria/sincroniza:

```text
<pasta>/desdobramento/output/
```

Essa é a pasta amigável para o usuário: ela reúne só arquivos finais, sem logs, prompts ou manifest técnico.

Se uma peça falhou, o PDF ainda deve ser gerado com o que existe e o manifest fica `parcial`.

---

## Passo 9 — Resumo no chat

```
✅ Desdobramento pronto. Tá tudo em:
   <pasta>/desdobramento/

📱 Instagram Feed:    instagram-feed.png + instagram-feed.txt
📲 Instagram Stories: 3 frames em instagram-stories/ + roteiro.txt
💼 LinkedIn Feed:     linkedin-feed.png + linkedin-feed.txt
📄 PDF apresentação:  apresentacao-desdobramento.pdf
📦 Entrega limpa:      output/

Custo: conferir créditos no {Higgsfield|Magnific}.
```

Cite o motor que você realmente usou.

Mostra paths absolutos pra usuário poder copiar/abrir no Finder.

---

## Regras invioláveis

- **Pre-flight conversa, não bloqueia.** Nunca diga ao usuário "edite arquivo Y". Conduza instalação/login com linguagem humana.
- **Usuário nunca vê o termo `.env`, "API key", "endpoint" ou "FAL_KEY".** Para ele: "Higgsfield conectado" ou "Magnific conectado".
- **Dois motores, um por execução.** Higgsfield CLI ou Magnific MCP. Se os dois estiverem disponíveis, **pergunte uma vez** qual usar. Nunca troque no meio, nem como fallback de erro.
- **Vision SEMPRE nas imagens originais.** Nunca adivinhe paleta/mood — abre o arquivo e olha.
- **GPT Image 2 no Higgsfield.** Desdobramentos pelo Higgsfield são `higgsfield generate create gpt_image_2`. No Magnific, a ferramenta de image-to-image de maior qualidade.
- **Desdobramento, não redesign.** Se Feed ou LinkedIn parecem uma arte nova inspirada na original, está errado.
- **Imagem-base SEMPRE junto.** Toda chamada usa a arte-mãe como primeira referência: `--base <arte-mae>` no Higgsfield, `base_reference` como imagem de referência no Magnific.
- **Base-only como padrão.** Não mande todas as imagens se elas não forem necessárias. Várias refs competindo causam deriva visual.
- **Prompt curto e idêntico nos dois motores.** O modelo já lê a arte. Peça transformação, formato, copy exata e só o que precisa trocar.
- **No Magnific, o prompt vem de `build-prompt`.** Nunca o `_prompts/*.txt` cru — ele não tem as regras de fidelidade nem a regra de variação dos Stories.
- **Resultado do Magnific passa por `save-external`.** Nunca baixe na mão: é o `save-external` que garante nome, pasta e log.
- **Stories sempre em 3.** Gere três variações da mesma arte-mãe, com textos e imagens/backgrounds diferentes. O trio precisa variar cena/crop/fundo/iluminação sem inventar uma nova direção visual.
- **Chain de referência é do script.** `generate` (ou `build-prompt`) já resolve a ordem das referências e a arte-mãe. Não tente fazer chain manualmente.
- **Cada formato é independente.** Stories falhar não para Feed/LinkedIn.
- **Copies completas, do zero.** Instagram Feed, roteiro de Stories e LinkedIn precisam ter textos finais próprios. Nada de adaptação preguiçosa.
- **PDF final obrigatório.** Toda execução termina com `presentation-pdf`.
- **Output limpo obrigatório.** Toda execução finalizada precisa deixar `desdobramento/output/` organizado para o usuário.
- **Stateless.** Cada `/desdobrar` é uma execução fresca. Sem cache, sem state.
