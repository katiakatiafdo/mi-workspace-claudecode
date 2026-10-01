# PROVIDERS — CAMADA DE RENDER (Higgsfield CLI **ou** Magnific MCP)

> Esta pasta circula entre pessoas com setups diferentes. Umas têm o **Higgsfield CLI**, outras
> o **Magnific MCP**, algumas têm os dois.
> A **inteligência editorial é a mesma nos dois casos** — pauta, headline, copy dos slides,
> arquitetura narrativa e design system não mudam. Só muda quem executa o render.
> Leia este arquivo antes de renderizar qualquer slide.

---

## 1. REGRA DE OURO

O prompt **não muda** por causa do motor. Você monta o image brief e o prompt do slide do jeito
de sempre (`09-Render-Engine.md`) e só no momento do render escolhe quem executa.

Nunca troque de motor no meio de um carrossel. Capa e slides internos saem todos pelo mesmo —
senão o carrossel perde a coerência slide-a-slide, que é o ponto inteiro do produto.

---

## 2. ORDEM DE RESOLUÇÃO DO MOTOR

Resolva nesta ordem e **pare no primeiro que der match**:

1. **Pedido explícito do usuário** — "usa o Magnific", "roda pelo Higgsfield", "usa o MCP".
2. **Variável de ambiente** `HUMAN_IMAGE_PROVIDER` — valores aceitos: `higgsfield` ou `magnific`.
3. **Só um disponível** → use esse, sem perguntar nada. Não anuncie o que faltou.
4. **Os dois disponíveis** → **pergunte ao usuário**. Uma linha, uma vez por carrossel:

   > Posso gerar os slides por dois motores: **Higgsfield** ou **Magnific**. Qual você prefere?

   Se ele não tiver preferência, use o **Higgsfield** e siga. Guarde a escolha para o carrossel
   inteiro — não pergunte de novo a cada slide.
5. **Nenhum disponível** → não renderize e não prometa arquivo. Salve o brief, a headline e a
   copy de todos os slides, e conduza o setup da seção 6.

### ⚠️ Exceção: dentro de uma Routine não existe ninguém para perguntar

A R2 (Routine Local, `13-R2-Routine-Local.md`) roda **sem supervisão**, de madrugada, sem
usuário na frente. Ali o passo 4 não se aplica. Dentro de uma Routine, resolva assim:

1. `HUMAN_IMAGE_PROVIDER`, se estiver definida.
2. Senão, **Higgsfield CLI**.
3. Se o Higgsfield não estiver disponível e o Magnific estiver, use o Magnific e **registre a
   troca no log da execução**, para a pessoa ver de manhã.
4. Se nenhum estiver disponível, não gere imagem: produza o pacote textual (pauta, headline,
   copy de todos os slides) e registre no log o que faltou.

Quem quiser travar a Routine em um motor específico define `HUMAN_IMAGE_PROVIDER` no setup
(`02-Setup-Wizard.md`) e nunca mais pensa nisso.

### Pré-flight

```bash
higgsfield account status
```

Isso responde o status do Higgsfield. O status do **Magnific** você mesmo verifica: olhe se
existem ferramentas `mcp__magnific__*` na sessão (se estiverem diferidas, use `ToolSearch` com
a query `magnific`).

---

## 3. CONTRATO COMUM (vale para os dois motores)

Independente de quem renderiza:

| Item | Valor |
|---|---|
| Aspect ratio | `3:4` |
| Resolução | `2k` |
| Qualidade | `high` |
| Referências | refs da marca **sempre**; foto da notícia quando existir |
| Ordem | **capa primeiro**, slides internos depois, em paralelo, usando a capa como referência |
| Saída | `human-output/carrossel/{slug}/`, com brief, prompts, imagens, parâmetros e logs |
| Tamanho do PNG | **o original retornado pelo motor** — nada de downscale, crop ou resize |

A regra do PNG original vale para os dois. Se o Magnific devolver 2048px ou outra dimensão 3:4
de alta resolução, mantenha assim.

---

## 4. MOTOR A — HIGGSFIELD CLI

Modelo obrigatório: **GPT Image 2** (`gpt_image_2`) — exceção da casa, porque os slides têm
lettering e texto renderizados junto da imagem.

Fluxo: `higgsfield upload create` para cada referência local → `higgsfield generate create
gpt_image_2` com os UUIDs em `--image` → `higgsfield generate wait`. Detalhes e os blocos de
prompt prontos em `09-Render-Engine.md`.

---

## 5. MOTOR B — MAGNIFIC MCP

Servidor: `https://mcp.magnific.com/mcp` (HTTP, OAuth no primeiro uso).

### 5.1. Setup (uma vez por máquina)

Quem abre o Claude Code **dentro desta pasta** já encontra o servidor declarado no
[.mcp.json](.mcp.json) — basta aprovar quando o Claude Code perguntar.

Para qualquer outra pasta (e para a Routine, que roda fora daqui):

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

### 5.2. Como chamar

Os nomes das ferramentas mudam entre versões. **Descubra em runtime**: procure `mcp__magnific__*`
na sessão (se estiverem diferidas, `ToolSearch` com a query `magnific`) e leia a assinatura
antes de chamar. Mapeie:

| Contrato | Como passar no Magnific |
|---|---|
| `prompt` | campo de prompt/texto — **o mesmo bloco montado em `09-Render-Engine.md`**, com o image brief dentro |
| `aspect_ratio` | `3:4`; se a ferramenta só aceitar largura/altura, converta mantendo a proporção |
| `resolution` | `2k` ≈ 2048px no lado maior |
| refs da marca | campo de imagem de referência / image-to-image — **obrigatório**, é o que ancora estética e fonte |
| capa → internos | gere a capa primeiro e passe **a capa** como referência nos internos |
| N slides | uma chamada por slide, disparadas em paralelo depois que a capa estiver pronta |

**Sem UUIDs.** O ciclo `higgsfield upload create` → `higgsfield-ref-flags.txt` é exclusivo do
Higgsfield. No Magnific as referências vão como arquivo direto na chamada; o arquivo de flags
fica vazio e isso não quebra nada.

### 5.3. Salvando o resultado

Grave o PNG no **mesmo caminho** que o Higgsfield usaria, dentro de
`human-output/carrossel/{slug}/`, com o mesmo padrão de nome dos slides. Registre nos logs da
execução qual motor e qual modelo foram usados — não há log do CLI para isso.

---

## 6. QUANDO NENHUM MOTOR EXISTE

1. Salve o brief, a headline e a copy de **todos** os slides. Esse trabalho é o núcleo editorial
   e não depende de motor nenhum.
2. Diga em uma frase que falta o motor de imagem.
3. Ofereça os dois caminhos:

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

- [ ] Motor resolvido pela ordem da seção 2 — e, se for Routine, pela regra sem-usuário
- [ ] Se os dois estavam disponíveis e havia usuário, a escolha foi **perguntada** — uma vez só
- [ ] Carrossel inteiro no mesmo motor
- [ ] Capa gerada **primeiro**; internos depois, com a capa como referência
- [ ] Refs da marca enviadas em **todas** as chamadas, não só na capa
- [ ] `3:4`, `2k`, `high` nos dois motores
- [ ] PNG entregue no tamanho original, sem downscale/crop/resize
- [ ] Motor e modelo registrados nos logs quando o render não foi pelo CLI
- [ ] Sem fallback silencioso de motor ou de modelo — troca dentro de Routine vai para o log
