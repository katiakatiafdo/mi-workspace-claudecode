# PROVIDERS — CAMADA DE RENDER (Higgsfield CLI **ou** Magnific MCP)

> Esta pasta circula entre pessoas com setups diferentes. Umas tem o **Higgsfield CLI**, outras
> o **MCP do Magnific**, algumas tem os dois.
> A **inteligencia do desdobramento e a mesma nos dois casos** — arte-mae, prompts, safe zones,
> trio de Stories e copies nao mudam. So muda quem executa o render.
> Leia este arquivo antes de gerar qualquer peca.

---

## 1. REGRA DE OURO

O prompt **nao muda** por causa do motor. Voce escolhe a arte-mae, escreve o prompt curto e
so no momento do render decide o motor.

Nunca troque de motor no meio de uma execucao. Se o Feed saiu no Higgsfield, os Stories e o
LinkedIn saem no Higgsfield tambem — senao a entrega vira tres mundos visuais diferentes, que e
exatamente o que este produto existe para evitar.

---

## 2. ORDEM DE RESOLUCAO DO MOTOR

Resolva nesta ordem e **pare no primeiro que der match**:

1. **Pedido explicito do usuario** — "usa o Magnific", "roda pelo Higgsfield", "usa o MCP".
2. **Variavel de ambiente** `HUMAN_IMAGE_PROVIDER` — valores aceitos: `higgsfield` ou `magnific`.
3. **So um disponivel** -> use esse, sem perguntar nada. Nao anuncie o que faltou.
4. **Os dois disponiveis** -> **pergunte ao usuario**. Uma linha, uma vez por execucao, antes de
   gerar a primeira peca:

   > Posso gerar por dois motores: **Higgsfield** ou **Magnific**. Qual voce prefere?

   Espere a resposta. Se ele nao tiver preferencia ("tanto faz", "escolhe voce"), use o
   **Higgsfield** e siga sem alongar. Guarde a escolha para a execucao inteira — nao pergunte de
   novo a cada Story.
5. **Nenhum disponivel** -> nao tente renderizar e nao prometa arquivo. Salve os prompts e as
   copies em disco e conduza o setup da secao 6, em linguagem simples, sem stack trace.

### Pre-flight obrigatorio

Antes do primeiro render, rode:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" check-providers
```

Windows:

```bash
python "%SOCIAL_HOME%\scripts\desdobrar.py" check-providers
```

O comando responde o status do Higgsfield CLI. O status do Magnific **voce mesmo verifica**:
olhe se existem ferramentas `mcp__magnific__*` na sessao (se estiverem diferidas, use
`ToolSearch` com a query `magnific`).

---

## 3. CONTRATO COMUM (vale para os dois motores)

Independente de quem renderiza, a entrega e a mesma:

```text
<pasta>/desdobramento/
├── instagram-feed.png             1080×1440, 3:4
├── instagram-feed.txt
├── instagram-stories/
│   ├── story-01.png               1080×1920
│   ├── story-02.png
│   ├── story-03.png
│   └── roteiro.txt
├── linkedin-feed.png              1920×1080, 16:9
├── linkedin-feed.txt
├── apresentacao-desdobramento.pdf
├── output/
├── _logs/                         1 json por peca, com motor e modelo usados
└── manifest.json
```

E os mesmos parametros logicos:

| Parametro | Valor | Observacao |
|---|---|---|
| `formato` | `ig-feed`, `ig-stories`, `linkedin-feed` | define dimensao e pasta de saida |
| `prompt` | texto em ingles | **identico nos dois motores** — pegue com `build-prompt` |
| arte-mae | 1 imagem | sempre a primeira referencia |
| `reference_mode` | `base-only` (padrao) ou `all` | `all` so quando as outras imagens forem do mesmo sistema visual |
| Stories | sempre 3 frames | textos e backgrounds diferentes entre si |

---

## 4. MOTOR A — HIGGSFIELD CLI

Modelo obrigatorio: **GPT Image 2** (`gpt_image_2`) — excecao da casa, porque as pecas tem
lettering e design renderizados junto da imagem.

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" generate "<pasta>" ig-feed \
  "<pasta>/desdobramento/_prompts/ig-feed.txt" --base arte-mae.png
```

O script cuida de tudo: sobe as referencias com `higgsfield upload create`, chama
`higgsfield generate create gpt_image_2` com `--image`, espera o job, baixa o PNG, escreve o log
em `_logs/` e devolve JSON com `output_path` e `higgsfield_url`.

Se falhar, **nao troque de modelo nem de motor como fallback**. Corrija login, referencia,
prompt ou formato e tente de novo.

---

## 5. MOTOR B — MAGNIFIC MCP

Servidor: `https://mcp.magnific.com/mcp` (HTTP, autenticado por OAuth no primeiro uso).

### 5.1. Setup (uma vez por maquina)

Se a pessoa abriu o Claude Code **dentro desta pasta**, o [.mcp.json](.mcp.json) daqui ja declara
o servidor — basta aprovar quando o Claude Code perguntar.

Para usar em **qualquer outra pasta**, adicione o servidor uma vez:

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

Depois disso, reabra a sessao e faca o login/autorizacao do Magnific quando ele pedir.

### 5.2. Pegue o prompt certo — nao improvise

O `generate` do Higgsfield **enriquece** o prompt que voce escreveu com as regras de fidelidade
a arte-mae e, nos Stories, com a regra de variacao do frame. Se voce chamar o Magnific com o
prompt cru do `_prompts/*.txt`, a peca sai fora do padrao.

Pegue o prompt final e as referencias exatas com:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" build-prompt "<pasta>" ig-stories \
  "<pasta>/desdobramento/_prompts/story-02.txt" --base arte-mae.png --output story-02.png
```

Ele devolve JSON com `prompt`, `base_reference`, `reference_files`, `dimensions` e
`output_path`. Use **esse** `prompt`.

### 5.3. Como chamar

Os nomes exatos das ferramentas mudam entre versoes do servidor. **Descubra em runtime**:
procure as ferramentas `mcp__magnific__*` na sessao (se estiverem diferidas, `ToolSearch` com a
query `magnific`) e leia a assinatura antes de chamar. Depois mapeie o contrato:

| Contrato | Como passar no Magnific |
|---|---|
| `prompt` | campo de prompt/texto da ferramenta de geracao (o texto vindo do `build-prompt`) |
| arte-mae | campo de imagem de referencia / image-to-image — **obrigatorio**, e o que segura a identidade |
| `aspect_ratio` | 3:4 (Feed), 9:16 (Stories), 16:9 (LinkedIn); se a ferramenta so aceitar largura/altura, use as de `dimensions` |
| Stories | uma chamada por frame, 3 no total |

Se o Magnific expuser uma ferramenta de **upscale** alem da de geracao, so use quando o usuario
pedir qualidade maxima. Nao faca upscale por conta propria — isso consome credito.

### 5.4. Salvando o resultado no padrao da casa

O MCP devolve uma URL (ou um arquivo). Nao deixe o resultado solto e nao baixe na mao: passe
pelo script, para cair na mesma pasta, com log e nome corretos.

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" save-external "<pasta>" ig-stories \
  --url "https://.../resultado.png" --output story-02.png \
  --provider magnific_mcp --model "{nome_da_ferramenta}" \
  --base arte-mae.png \
  --prompt-file "<pasta>/desdobramento/_prompts/story-02.txt"
```

Se o MCP ja gravou um arquivo local, troque `--url` por `--file "/caminho/do/arquivo.png"`.

O `presentation-pdf` le esses logs, entao o PDF final mostra sozinho qual motor e qual modelo
geraram a entrega. Voce nao precisa ajustar nada.

---

## 6. QUANDO NENHUM MOTOR EXISTE

Nao invente render e nao prometa arquivo. Faca assim:

1. Salve os prompts e **todas as copies** (Feed, roteiro dos Stories, LinkedIn). Esse trabalho
   nao depende de motor nenhum e ja tem valor.
2. Diga em uma frase que falta o motor de imagem.
3. Ofereca os dois caminhos, sem jargao:

**Higgsfield CLI**

```bash
npm install -g @higgsfield/cli
```

```bash
higgsfield auth login
```

**Magnific MCP**

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

4. Confirme credenciais antes de rodar qualquer comando pago.

---

## 7. CHECKLIST DO RENDER

- [ ] Motor resolvido pela ordem da secao 2, nao por chute
- [ ] Se os dois estavam disponiveis, a escolha foi **perguntada** ao usuario — uma vez so
- [ ] `check-providers` rodado antes do primeiro render
- [ ] Arte-mae escolhida **antes** de gerar qualquer peca, e a mesma nas tres redes
- [ ] No Magnific, o prompt veio de `build-prompt` — nunca o `_prompts/*.txt` cru
- [ ] Arte-mae enviada como referencia em **todas** as chamadas, nao so na primeira
- [ ] Execucao inteira no mesmo motor
- [ ] Stories em 3 frames, com backgrounds diferentes entre si
- [ ] Resultado do Magnific salvo com `save-external`, nunca baixado na mao
- [ ] Conferido quais PNGs existem de fato no fim; refeitos so os que falharam
- [ ] `presentation-pdf` rodado ao final, com `output/` sincronizado
- [ ] Sem fallback silencioso de motor ou de modelo
