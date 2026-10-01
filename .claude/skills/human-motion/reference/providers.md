# PROVIDERS — CAMADA DE RENDER (Higgsfield CLI **ou** Magnific MCP)

> Este produto tem **duas etapas**. A etapa 1 (imagem estática) aceita dois motores.
> A etapa 2 (motion/Seedance) é **sempre Higgsfield CLI** — nenhum outro provedor gera o vídeo.
> Leia este arquivo antes de renderizar qualquer coisa.

---

## 1. REGRA DE OURO

O prompt **não muda** por causa do motor. Você escreve a direção de arte do jeito de sempre
(`prompts/01-frame-gpt-image-2.md`) e só no momento do render escolhe quem executa.

Nunca troque de motor no meio de um lote de imagens. Se as variações da etapa 1 começaram no
Higgsfield, terminam no Higgsfield.

---

## 2. ORDEM DE RESOLUÇÃO DO MOTOR (etapa 1)

Resolva nesta ordem e **pare no primeiro que der match**:

1. **Pedido explícito da pessoa** — "usa o Magnific", "roda pelo Higgsfield", "usa o MCP".
2. **Variável de ambiente** `HUMAN_IMAGE_PROVIDER` — valores aceitos: `higgsfield` ou `magnific`.
3. **Só um disponível** → use esse, sem perguntar nada. Não anuncie o que faltou.
4. **Os dois disponíveis** → **pergunte à pessoa**. Uma linha, uma vez por execução:

   > Posso gerar a imagem por dois motores: **Higgsfield** ou **Magnific**. Qual você prefere?

   Se ela não tiver preferência, use o **Higgsfield** (que também faz a etapa 2) e siga.
   Guarde a escolha no `brief.md` e não pergunte de novo.
5. **Nenhum disponível** → não renderize e não prometa arquivo. Salve os prompts e conduza o
   setup da seção 6, em linguagem simples, sem stack trace.

### Pré-flight obrigatório

```bash
python3 scripts/gerar_frame.py check-providers
```

Windows: `python scripts\gerar_frame.py check-providers`.

Responde o status do Higgsfield. O status do Magnific **você mesmo verifica**: olhe se existem
ferramentas `mcp__magnific__*` na sessão (se estiverem diferidas, use `ToolSearch` com a query
`magnific`).

---

## 3. A ETAPA 2 É SEMPRE HIGGSFIELD

O Magnific gera imagem, não o motion deste produto. Então:

- **Imagem no Magnific + Higgsfield disponível** → a etapa 1 sai pelo Magnific e a etapa 2 pelo
  Higgsfield. Diga isso quando chegar no vídeo, sem drama:
  *"A imagem saiu pelo Magnific; o vídeo vai pelo Higgsfield."*
- **Imagem no Magnific + Higgsfield ausente** → avise que o vídeo não tem como sair nessa
  máquina e entregue o pacote manual (Passo 9 do `CLAUDE.md`). Não invente outro caminho.

---

## 4. MOTOR A — HIGGSFIELD CLI

Modelo da etapa 1: **GPT Image 2** (`gpt_image_2`), porque a imagem estática carrega lettering
e layout.

```bash
python3 scripts/gerar_frame.py render "output/{slug}/01-frame/prompt-frame.txt" \
  --aspect-ratio 9:16 --resolution 2k \
  --output-dir "output/{slug}/01-frame" --output-name "frame-01.png" \
  --reference "assets/referencias/estilo.png"
```

O script checa CLI/login, sobe as referências, submete, espera o job, baixa o arquivo e escreve
`_logs/`.

---

## 5. MOTOR B — MAGNIFIC MCP

Servidor: `https://mcp.magnific.com/mcp` (HTTP, OAuth no primeiro uso).

### 5.1. Setup (uma vez por máquina)

Quem abre o Claude Code **dentro desta pasta** já encontra o servidor declarado no
[.mcp.json](.mcp.json) — basta aprovar quando o Claude Code perguntar.

Para qualquer outra pasta:

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

### 5.2. Como chamar

Os nomes das ferramentas mudam entre versões. **Descubra em runtime**: procure `mcp__magnific__*`
na sessão (se estiverem diferidas, `ToolSearch` com a query `magnific`) e leia a assinatura
antes de chamar. Mapeie:

| Contrato | Como passar no Magnific |
|---|---|
| `prompt` | campo de prompt/texto da ferramenta de geração — **o mesmo texto do `prompt-frame.txt`** |
| `aspect_ratio` | campo de aspect ratio; se só aceitar largura/altura, converta mantendo a proporção |
| `resolution` | `1k` ≈ 1024px, `2k` ≈ 2048px, `4k` ≈ 4096px no lado maior |
| referências | campo de imagem de referência / image-to-image, se existir |
| variações | uma chamada por imagem |

### 5.3. Salvando no padrão da casa

Não deixe o retorno solto e não baixe na mão — sem isso a execução fica sem `_logs/`:

```bash
python3 scripts/gerar_frame.py save-external \
  --url "https://.../resultado.png" \
  --output-dir "output/{slug}/01-frame" --output-name "frame-01.png" \
  --provider magnific_mcp --model "{nome_da_ferramenta}" \
  --prompt-file "output/{slug}/01-frame/prompt-frame.txt" \
  --aspect-ratio 9:16 --resolution 2k
```

Se o MCP gravou arquivo local, troque `--url` por `--file "/caminho/do/arquivo.png"`.

O arquivo precisa cair exatamente em `output/{slug}/01-frame/frame-01.png`, porque é dele que a
etapa 2 parte.

---

## 6. QUANDO NENHUM MOTOR EXISTE

1. Salve os prompts da etapa 1 e da etapa 2 em disco. Esse trabalho não depende de motor.
2. Diga em uma frase que falta o motor de render.
3. Ofereça os dois caminhos:

**Higgsfield CLI** (faz imagem **e** vídeo)

```bash
npm install -g @higgsfield/cli
```

```bash
higgsfield auth login
```

**Magnific MCP** (faz imagem)

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

4. Confirme credenciais antes de rodar qualquer comando pago.

---

## 7. CHECKLIST DO RENDER

- [ ] Motor da etapa 1 resolvido pela ordem da seção 2, não por chute
- [ ] Se os dois estavam disponíveis, a escolha foi **perguntada** — uma vez só, e anotada no `brief.md`
- [ ] `check-providers` rodado antes do primeiro render
- [ ] Lote inteiro da etapa 1 no mesmo motor
- [ ] Resultado do Magnific salvo com `save-external`, nunca baixado na mão
- [ ] `frame-01.png` no caminho certo antes de começar a etapa 2
- [ ] Etapa 2 sempre no Higgsfield CLI; se ele não existir, a pessoa foi avisada
- [ ] Sem fallback silencioso de motor ou de modelo
