# Geradores — os dois caminhos

Este projeto circula entre pessoas com setups diferentes. Ele funciona com **qualquer um dos dois**, e a escolha é automática: no começo da conversa o sistema detecta o que você tem e usa.

| Caminho | O que é | Cobre |
|---|---|---|
| **A — Higgsfield CLI** | Linha de comando: `gpt_image_2` (imagem) e `seedance_2_0` (vídeo) | Etapa 1 **e** etapa 2 |
| **B — Magnific MCP** | Servidor MCP conectado ao Claude Code | Etapa 1 (imagem) |

Não precisa ter os dois. Se tiver os dois, o sistema **pergunta uma vez** qual você prefere e segue com ele até o fim. Como o Magnific faz imagem e não faz o motion deste produto, se você escolher o Magnific a etapa 1 sai por ele e a **etapa 2 sai pelo Higgsfield**.

A regra técnica completa (ordem de resolução, comandos, o que fazer quando falta motor) está em [providers.md](providers.md).

---

## Caminho A — Higgsfield CLI

**Instalar:**

```bash
npm install -g @higgsfield/cli
```

**Logar (uma vez):**

```bash
higgsfield auth login
```

**Conferir a qualquer momento** (os dois scripts respondem igual):

```bash
python3 scripts/gerar_frame.py check
```

O `check` responde uma destas três coisas:

| Resposta | Significa |
|---|---|
| `ok` | Instalado e logado. Pode gerar. |
| `login_required` | Instalado, mas a sessão expirou. Rode `higgsfield auth login`. |
| `missing` | Não instalado. Instale, ou use o caminho B. |

**Gerar:**

```bash
python3 scripts/gerar_frame.py render "output/{slug}/01-frame/prompt-frame.txt" \
  --aspect-ratio "9:16" --resolution 2k \
  --output-dir "output/{slug}/01-frame" --output-name "frame-01.png"
```

Referências são opcionais e repetíveis: `--reference "assets/referencias/estilo.png"`.

O script recusa automaticamente qualquer referência que pareça um logo — o logo não entra na imagem estática.

O modelo padrão é `gpt_image_2`. Para trocar sem mexer no código:

```bash
export HUMAN_MOTION_IMAGE_MODEL=nano_banana_2
```

---

## Caminho B — Magnific MCP

**Conectar:** se você abrir o Claude Code **dentro desta pasta**, o `.mcp.json` daqui já declara o servidor — é só aprovar quando ele perguntar e fazer o login do Magnific.

Para usar em qualquer outra pasta, rode uma vez:

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

Depois de conectado, as ferramentas aparecem sozinhas — não precisa configurar mais nada.

**Como o sistema usa:** o mesmo prompt de `output/{slug}/01-frame/prompt-frame.txt`, os mesmos parâmetros (aspect ratio, resolução), e o arquivo gravado **no mesmo lugar**, pelo próprio script (`save-external`), para a execução ficar com log igual à do caminho A:

```
output/{slug}/01-frame/frame-01.png
```

O resultado é idêntico do ponto de vista do fluxo. A etapa 2 (Seedance) não muda em nada — ela só precisa do PNG.

**Se o Magnific não estiver conectado**, o sistema avisa e oferece o caminho A.

---

## O que não muda, nos dois caminhos

- O prompt é escrito em inglês, seguindo `prompts/01-frame-gpt-image-2.md`.
- O arquivo final vai para `output/{slug}/01-frame/frame-NN.png`.
- **O logo nunca entra na imagem** — ele sobe separado no Seedance.
- A aprovação da imagem é obrigatória antes de montar o motion.

---

## Etapa 2 — o vídeo (Seedance 2.0)

O vídeo é **automático**. Depois que você aprova a imagem, o sistema escreve o prompt, sobe os arquivos e gera o MP4 — você não precisa abrir o Seedance.

**Modelo:** `seedance_2_0`

| Parâmetro | Opções | Padrão aqui |
|---|---|---|
| `--duration` | 4 a 15 segundos | 15 |
| `--resolution` | `480p` · `720p` · `1080p` | 1080p |
| `--aspect-ratio` | `9:16` `1:1` `16:9` `4:3` `3:4` `21:9` `auto` | 9:16 |
| `--mode` | `std` (final) · `fast` (preview) | std |
| `--sound` | `on` · `off` | **on — sempre** |
| `--genre` | `auto` e outros | auto |

**O som é sempre ligado.** É padrão do projeto, não uma pergunta: o prompt já traz a direção de áudio (foley de entrada, pouso, base rítmica, sweep de transição, impacto no logo) e a trava contra locução e trilha cantada. Se o seu build do modelo não aceitar o parâmetro `--sound`, o script tenta de novo sem ele e anota isso no log.

**Estimar o custo antes:**

```bash
python3 scripts/gerar_motion.py cost "output/{slug}/02-motion/prompt-seedance.txt" \
  --duration 15 --resolution 1080p --aspect-ratio "9:16"
```

**Gerar:**

```bash
python3 scripts/gerar_motion.py render "output/{slug}/02-motion/prompt-seedance.txt" \
  --frame "output/{slug}/01-frame/frame-01.png" \
  --logo "assets/logo/logo.png" \
  --duration 15 --resolution 1080p --aspect-ratio "9:16" --mode std \
  --output-dir "output/{slug}/02-motion" --output-name "motion-01.mp4"
```

`--frame` e `--produto` são repetíveis. A ordem de envio das imagens é **frames → produto → logo**, e o script cuida disso — o prompt se refere ao logo como "the attached logo image" no bloco final.

**Se a etapa 1 saiu pelo Magnific**, a etapa 2 sai pelo Higgsfield CLI — o Magnific não gera o motion deste produto. O sistema avisa quando isso acontecer.

**Se não houver gerador de vídeo na máquina**, o sistema entrega o pacote manual — `02-motion/UPLOAD.md`, com a lista de arquivos, os parâmetros e o prompt pronto pra colar no Seedance na mão.

## Sobre custo

Vídeo custa mais que imagem. Por isso a ordem importa: a imagem é aprovada **antes** de qualquer crédito de vídeo ser gasto. Se quiser ver o movimento sem pagar o preço cheio, peça `fast` — sai em preview, e depois você roda o `std`.
