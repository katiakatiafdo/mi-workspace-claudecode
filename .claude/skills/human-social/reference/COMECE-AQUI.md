# COMECE AQUI — Human Social

Esta pasta pega **uma peça que você já tem** e a desdobra em três redes.

```
   VOCÊ TEM                          SAI DAQUI
   ┌──────────────────┐              ┌────────────────────────┐
   │  1 pasta com:    │              │  Instagram Feed  (3:4) │
   │  · a legenda .txt│   ───────>   │  3 Instagram Stories   │
   │  · as imagens    │              │  LinkedIn Feed  (16:9) │
   └──────────────────┘              │  + legenda de cada uma │
                                     │  + PDF de apresentação │
                                     └────────────────────────┘
```

**Não é resize.** Cada peça é gerada de novo a partir da sua arte-mãe — mesma foto, mesma
fonte, mesmas cores, mesmos elementos gráficos — com o formato, a copy e as safe zones certas
para cada rede. A peça nova precisa parecer irmã da original, não uma campanha nova inspirada
nela.

---

## Como começar

**Abra o Claude Code nesta pasta e diga "vamos começar".** O sistema se apresenta e pede a
pasta.

Ou vá direto ao ponto e mande o caminho:

```
/desdobrar /caminho/da/pasta
```

No Mac, arraste a pasta do Finder pra dentro do chat — o caminho completo cola sozinho.

### O que precisa ter dentro da sua pasta

| Item | Detalhe |
|---|---|
| **1 arquivo `.txt`** | A legenda ou o briefing original. Qualquer nome serve. |
| **1 ou mais imagens** | `.png`, `.jpg`, `.jpeg` ou `.webp` |

Se houver uma imagem já diagramada (foto + lettering), ela vira a **arte-mãe** — a referência
de identidade de todas as outras. Se houver várias, o sistema escolhe a mais próxima de uma
peça final.

---

## Antes do primeiro uso

Você precisa de **um** motor de imagem. Se tiver os dois, o sistema **pergunta** qual você quer
usar antes de gerar — uma vez por execução, não a cada imagem.

### Opção A — Higgsfield

```bash
npm install -g @higgsfield/cli
```

```bash
higgsfield auth login
```

O modelo usado é o **GPT Image 2** (`gpt_image_2`), porque as peças finais têm lettering,
design e texto renderizados junto da imagem.

### Opção B — Magnific

Se você abrir o Claude Code **dentro desta pasta**, o `.mcp.json` daqui já declara o servidor —
é só aprovar quando o Claude Code perguntar e fazer o login do Magnific.

Para usar em qualquer outra pasta, rode uma vez:

```bash
claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp
```

Só isso. Você nunca vai tocar em arquivo de configuração, token ou chave de API — se faltar
alguma coisa, o sistema pergunta no chat, em português.

---

## O que sai no final

Tudo nasce **dentro da sua pasta**, numa subpasta `desdobramento/`:

```
sua-pasta/desdobramento/
├── output/                          ← a pasta limpa, é aqui que você olha
│   ├── instagram-feed.png
│   ├── instagram-feed.txt
│   ├── story-01.png · story-02.png · story-03.png
│   ├── linkedin-feed.png
│   ├── linkedin-feed.txt
│   └── apresentacao-desdobramento.pdf
└── ...                              o resto é técnico: prompts, logs, manifest
```

**Os Stories são sempre 3**, e os três são diferentes de verdade — muda o crop, o ângulo, o
fundo, a luz. Não é a mesma imagem com o texto trocado.

**As três legendas são escritas do zero.** Instagram não é LinkedIn: uma é oral e rítmica, a
outra é editorial e em primeira pessoa. Se saírem parecidas, o sistema refaz.

---

## A pasta por dentro

| Arquivo | O que é |
|---|---|
| `COMECE-AQUI.md` | ← você está aqui |
| `CLAUDE.md` | As instruções que o sistema segue |
| `AGENTS.md` | Regras de roteamento e compatibilidade |
| `providers.md` | Os dois motores de imagem: Higgsfield e Magnific |
| `.mcp.json` | Declara o Magnific pra quem abrir o Claude Code aqui dentro |
| `.claude/skills/desdobrar/` | O pipeline completo, com os prompts |
| `scripts/desdobrar.py` | O executor que fala com os dois motores |
| `origem/` | Uma pasta de exemplo |

---

## Problemas comuns

**"Higgsfield CLI não encontrado"** — rode `npm install -g @higgsfield/cli`. Ou use o Magnific.

**"precisa de login"** — rode `higgsfield auth login`. Abre o navegador.

**O Magnific não aparece** — confirme que o servidor foi adicionado
(`claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp`), reabra
a sessão e autorize o login quando ele pedir.

**Quero sempre o mesmo motor, sem ser perguntado** — diga "usa sempre o Magnific" (ou o
Higgsfield) no começo da conversa.

**Uma das peças falhou** — as outras continuam. O sistema registra `parcial` no manifest e te
diz qual faltou. É só pedir pra refazer aquela.

**Quero trocar só a headline de uma peça pronta** — diga isso. O sistema usa a própria peça
como base e troca só o texto, sem mexer em foto, fundo, fonte, paleta ou layout.

---

**Pronto. Abre o Claude Code aqui e diz "vamos começar".**
