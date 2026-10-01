# HUMAN MOTION — comece por aqui

Este projeto transforma uma ideia em um **vídeo de motion graphics pronto**, em duas etapas:

```
   ETAPA 1                          ETAPA 2
   ┌──────────────────┐             ┌──────────────────┐
   │  IMAGEM ESTÁTICA │   ──────>   │   VÍDEO (MP4)    │
   │   (GPT Image 2)  │             │  (Seedance 2.0)  │
   └──────────────────┘             └──────────────────┘
   Uma tela só, com todos           Decompõe essa tela em camadas,
   os elementos da cena.            anima a entrada de cada uma,
                                    monta a segunda tela e fecha
   ✋ VOCÊ APROVA AQUI              com o logo.
                                    ⚡ automático — sai o vídeo
```

A lógica é essa: **você não anima do nada.** Primeiro a gente cria **uma tela base** com todos os elementos dentro dela. Depois o Seedance usa essa tela como matéria-prima — separa os elementos, faz eles entrarem um a um, desmonta, remonta em outra composição e finaliza com a assinatura da marca.

**Só existe um ponto de aprovação: a imagem.** Depois que você aprovar, o sistema escreve o prompt de animação, sobe os arquivos e devolve o MP4. Você não precisa abrir o Seedance.

---

## Como começar

**1. Coloque seus arquivos na pasta `assets/`** (isso é opcional — veja abaixo)

| Pasta | O que vai aqui |
|---|---|
| `assets/logo/` | O logo do cliente. **PNG com fundo transparente**, de preferência. |
| `assets/produto/` | Fotos do produto, embalagem, packshot. |
| `assets/referencias/` | Referências visuais: estilo, paleta, prints, campanhas que você gosta. |

**Não tem arquivo nenhum? Tudo bem.** Você pode simplesmente descrever o que quer e a gente cria a tela do zero. As referências só ajudam a acertar o estilo mais rápido.

> **A única coisa que vale a pena ter é o logo em arquivo separado.** Ele **não** entra na imagem estática — ele é enviado direto pro Seedance e aparece só no final do motion. Se não tiver o logo agora, dá pra seguir e adicionar depois.

**2. Abra esta pasta no Claude Code e diga "vamos começar"**

A partir daí o sistema conduz você. Ele vai:

1. Ler o que tem em `assets/`
2. Perguntar qual é a sua ideia — e **qual estilo** você quer (3D? colagem? realista? outro?)
3. Escrever o prompt e **gerar a imagem estática**
4. Esperar você aprovar (e refazer quantas vezes precisar)
5. Perguntar **como você quer animar**: o que acontece na segunda tela, se entra texto, se o logo fecha o filme
6. Perguntar só a **resolução** (1080p ou 720p) e se você quer um preview antes
7. **Gerar o vídeo** e te entregar o MP4

Tudo o que for gerado vai parar em `output/{nome-do-projeto}/`.

---

## O que sai no final

```
output/meu-projeto/
├── brief.md                    O que foi combinado
├── 01-frame/
│   ├── prompt-frame.txt        O prompt usado na imagem
│   └── frame-01.png            A imagem estática aprovada
└── 02-motion/
    ├── prompt-seedance.txt     O prompt de animação
    └── motion-01.mp4           ← o vídeo
```

**Você não precisa subir nada no Seedance.** O sistema faz o upload sozinho — a imagem aprovada, o produto se houver, e **o logo como arquivo separado**, porque ele não está na imagem e é o que fecha o motion.

**O vídeo sai com som.** Sempre. O prompt dirige o áudio junto com a animação: swish em cada entrada, tap quando o elemento assenta, base rítmica no tempo do build-up e um impacto no fecho do logo — sem locução e sem trilha cantada. Se você for colocar trilha própria na edição, o som do modelo ainda serve de referência de timing.

Se a máquina não tiver gerador de vídeo, aí sim o sistema entrega um `UPLOAD.md` com a lista de arquivos e o prompt pronto pra você colar no Seedance na mão.

---

## O estilo é seu

**Não existe um visual padrão aqui.** Você pede o que quiser:

| Estilo | Cara disso |
|---|---|
| **2D flat / vetor** | Cores chapadas, formas geométricas. Corporativo moderno, explicativo. |
| **Colagem / papel recortado** | Recorte, textura de impressão, stop-motion. Artesanal, editorial. |
| **3D render** | Volume, material, luz. Produto premium, tech — e a câmera pode girar em volta. |
| **Realista / foto** | Cena real, live-action. Campanha, lifestyle. |
| **Mixed media** | Foto recortada com grafismo e traço por cima. Streetwear, cultural. |
| **Ilustração** | Aquarela, nanquim, anime, risografia. Narrativo, humano. |

Ou qualquer outro que você tenha na cabeça — pixel art, claymation, blueprint, retrô 80s. Se você não disser, **o sistema pergunta** antes de gerar qualquer coisa.

## E o tipo de movimento

O sistema escolhe a partir da sua ideia — você não precisa decidir:

| Estrutura | Quando aparece |
|---|---|
| **Camadas** | Elementos entrando um a um. Processo, jornada, "como funciona". |
| **Cartelas** | Telas com frases curtas se substituindo. Manifesto, benefícios. |
| **Imagem + texto** | Tipografia entrando sobre uma imagem forte. Campanha, oferta. |
| **Câmera** | A câmera gira em volta do objeto. Reveal de produto, turntable. |

Estilo e movimento são independentes: dá pra ter cartelas em 3D, camadas em aquarela, câmera girando numa cena realista.

---

## Antes do primeiro uso (uma vez só)

A geração passa por **dois caminhos possíveis** — o sistema detecta sozinho qual você tem:

| Caminho | Precisa de | Cobre |
|---|---|---|
| **Higgsfield CLI** | `npm install -g @higgsfield/cli` e depois `higgsfield auth login` | Imagem **e** vídeo |
| **Magnific MCP** | `claude mcp add --transport http --scope user magnific https://mcp.magnific.com/mcp` — ou só abrir o Claude Code nesta pasta e aprovar | Imagem |

**Basta um dos dois.** Se você tiver os dois, o sistema pergunta uma vez qual prefere e segue com ele. Se você escolher o Magnific, o vídeo sai pelo Higgsfield — o Magnific faz imagem, não faz o motion. Se não tiver nenhum, ele avisa e mesmo assim escreve os prompts — é só conectar depois e rodar.

Para conferir o que está pronto a qualquer momento:

```bash
python3 scripts/gerar_frame.py check-providers
```

Detalhes dos dois caminhos: [GERADORES.md](GERADORES.md) · regra técnica: [providers.md](providers.md)

---

## A pasta por dentro

```
HUMAN MOTION/
├── START.md          ← você está aqui
├── GERADORES.md      Os dois caminhos de geração de imagem
├── CLAUDE.md         As instruções que o sistema segue (não precisa ler)
├── assets/           ← seus arquivos entram aqui
│   ├── logo/
│   ├── produto/
│   └── referencias/
├── prompts/          A biblioteca de formatos — o cérebro do projeto
├── scripts/          O gerador de imagem
└── output/           ← tudo que for criado sai aqui
```

---

**Pronto. Coloca os arquivos na `assets/` (ou nem isso) e chama.**
