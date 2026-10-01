# PROVIDERS — CAMADA DE RENDER (Higgsfield CLI **ou** Magnific MCP)

> Esta pasta circula entre pessoas com setups diferentes. Umas têm o **Higgsfield CLI**, outras
> o **Magnific MCP**, algumas têm os dois.
> **Imagem** (still, product shot, character sheet, frame de cena) aceita os dois motores.
> **Vídeo** (Seedance, Kling) é **sempre Higgsfield CLI** — nenhum outro provedor gera vídeo aqui.
> Leia este arquivo antes de renderizar qualquer imagem.

---

## 1. REGRA DE OURO

O prompt **não muda** por causa do motor. A direção de arte, o Visual Intent, o critério de
ambição e a estrutura do prompt continuam iguais — só muda quem executa o render.

Nunca troque de motor no meio de uma série. Se os 6 product shots começaram no Higgsfield,
terminam no Higgsfield.

---

## 2. ORDEM DE RESOLUÇÃO DO MOTOR (imagem)

Resolva nesta ordem e **pare no primeiro que der match**:

1. **Pedido explícito da pessoa** — "usa o Magnific", "roda pelo Higgsfield", "usa o MCP".
2. **Variável de ambiente** `HUMAN_IMAGE_PROVIDER` — valores aceitos: `higgsfield` ou `magnific`.
3. **Só um disponível** → use esse, sem perguntar nada. Não anuncie o que faltou.
4. **Os dois disponíveis** → **pergunte à pessoa**. Uma linha, uma vez por campanha:

   > Posso gerar as imagens por dois motores: **Higgsfield** ou **Magnific**. Qual você prefere?

   Se ela não tiver preferência, use o **Higgsfield** (que também faz o vídeo) e siga.
   Anote a escolha no `internal/` da campanha e não pergunte de novo.
5. **Nenhum disponível** → não renderize e não prometa arquivo. Salve os prompts e conduza o
   setup da seção 6, em linguagem simples.

### Pré-flight obrigatório

```bash
higgsfield account status
```

Isso responde o status do Higgsfield. O status do **Magnific** você mesmo verifica: olhe se
existem ferramentas `mcp__magnific__*` na sessão (se estiverem diferidas, use `ToolSearch` com
a query `magnific`).

---

## 3. O VÍDEO É SEMPRE HIGGSFIELD

O Magnific gera imagem, não os vídeos deste produto (Seedance 2.0, Kling 3.0). Então:

- **Imagens no Magnific + Higgsfield disponível** → frames e stills saem pelo Magnific, o vídeo
  sai pelo Higgsfield. Diga isso ao chegar no vídeo, sem drama.
- **Imagens no Magnific + Higgsfield ausente** → a campanha vai até frames aprovados e para ali.
  Avise antes de começar, para a pessoa decidir se quer seguir assim.

**Atenção à trava do produto:** vídeo só depois de frames aprovados. Isso não muda com o motor.

---

## 4. MOTOR A — HIGGSFIELD CLI (o completo)

Modelo de imagem: **Nano Banana 2** (`nano_banana_2`). Fluxo normal da pasta:
`higgsfield upload create` para as referências → `higgsfield generate create` (batch inteiro,
sem `--wait`) → `higgsfield generate wait <job_id>` para coletar.

Vale a regra nº 9 do `CLAUDE.md`: **série inteira em fila**, nunca um job de cada vez.

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
| `prompt` | campo de prompt/texto — **o mesmo prompt em inglês** que iria para o Nano Banana 2 |
| `aspect_ratio` | campo de aspect ratio; se só aceitar largura/altura, converta mantendo a proporção |
| `resolution` | `1k` ≈ 1024px, `2k` ≈ 2048px, `4k` ≈ 4096px no lado maior |
| referências | campo de imagem de referência / image-to-image |
| série de N peças | uma chamada por peça, **disparadas em paralelo** — a regra da fila continua valendo |

### 5.3. Diferença que importa: UUIDs

O fluxo de UUIDs (`higgsfield upload create` → `ref-ids.md`) é **exclusivo do Higgsfield**.
No Magnific, as referências vão como arquivo/imagem direto na chamada da ferramenta — não há
UUID para guardar. Isso não quebra a campanha: o `ref-ids.md` só fica vazio ou parcial. Se a
pessoa depois migrar para o Higgsfield, é só subir as referências e capturar os UUIDs então.

### 5.4. Salvando o resultado

Grave o arquivo no **mesmo caminho** que o Higgsfield usaria — `output/` numerado, com o resto
em `internal/`, exatamente como manda o `CLAUDE.md`. Registre no `internal/` qual motor e qual
modelo foram usados, já que não haverá log do CLI.

---

## 6. QUANDO NENHUM MOTOR EXISTE

1. Salve os prompts, o Visual Intent e o roteiro. Esse trabalho não depende de motor.
2. Diga em uma frase que falta o motor de render.
3. Ofereça os dois caminhos:

**Higgsfield CLI** (imagem **e** vídeo)

```bash
npm install -g @higgsfield/cli
```

```bash
higgsfield auth login
```

**Magnific MCP** (imagem)

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

4. Confirme credenciais antes de rodar qualquer comando pago.

---

## 7. CHECKLIST DO RENDER

- [ ] Motor resolvido pela ordem da seção 2, não por chute
- [ ] Se os dois estavam disponíveis, a escolha foi **perguntada** — uma vez só, e anotada
- [ ] Pré-flight rodado antes do primeiro render
- [ ] Prompt idêntico ao que seria usado no outro motor
- [ ] Série inteira no mesmo motor, disparada **em paralelo** (teto de 8)
- [ ] Referências repetidas em **todas** as chamadas da série, não só na primeira
- [ ] Arquivos finais em `output/` numerado; dados e logs em `internal/`
- [ ] Motor e modelo registrados no `internal/` quando o render não foi pelo CLI
- [ ] Vídeo só depois de frames aprovados, e sempre no Higgsfield
- [ ] Sem fallback silencioso de motor ou de modelo
