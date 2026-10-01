# 📁 projetos/ — uma pasta por marca

Esta pasta é o **container de todas as marcas** que você constrói com o sistema. Cada projeto vive em sua própria sub-pasta, isolado dos outros. Sem limite de projetos.

---

## Estrutura

```
projetos/
├── README.md          ← este arquivo
├── marca-x/           ← projeto real
├── marca-y/           ← projeto real
└── marca-z/           ← projeto real
```

> O **molde-base** dos projetos não vive aqui — vive em `../inteligencias/_template/`. Faz parte do kernel do sistema, não da área de trabalho. O Maestro usa esse molde quando cria um projeto novo.

---

## Como funciona

Quando você roda `claude` na pasta-raiz do sistema e manda "oi", o Maestro:

1. **Lista os projetos existentes** (as subpastas daqui)
2. **Pergunta** se você quer abrir um deles ou criar um novo
3. **Roteia conforme:**
   - **Novo** → cria `projetos/[slug]/` com `referencias/` e `resultado/`, define como pasta de trabalho e começa o briefing
   - **Existente** → lê o DNA daquela marca, devolve um resumo curto que prova que entendeu, e oferece os próximos passos (gerar peça, auditar, editar o DNA, consultar)

---

## Cada projeto tem

- **`referencias/`** — entrada livre de matéria-prima: logos, fontes, imagens, paletas, textos, decks, PDFs, links salvos em `.txt`, concorrentes e anti-referências
- **`resultado/`** — os entregáveis: `DNA.md` (fonte que as IAs leem), `DNA.pdf` (documento editorial diagramado) e `DNA.html` (auxiliar de inspeção)
- **`CLAUDE.md`** — orienta o Claude Code a ler e seguir o DNA dessa marca
- Arquivos técnicos invisíveis (`.brand.json`, `.discovery-progress.json`, `notion-ids.json` se você sincronizou com Notion)

---

## Como criar um projeto novo manualmente (raro)

Normalmente o Maestro cria sozinho. Se quiser fazer na mão:

```bash
mkdir -p projetos/marca-nova/referencias projetos/marca-nova/resultado
```

Depois rode `claude` na pasta-raiz e diga "trabalhar no projeto marca-nova".

---

## Compartilhando um projeto pronto

Para entregar a marca a um designer, copy ou agência, mande o `resultado/DNA.pdf`. Para entregar a outra IA ou a outro time técnico, mande o `resultado/DNA.md`. Para passar a marca inteira adiante, copie a sub-pasta do projeto — ela é autossuficiente.

---

## Não mexa em `inteligencias/_template/`

Esse template é a estrutura-modelo usada para criar projetos novos. Mexer nele afeta as criações futuras. Se quiser personalizar o molde, edite com cuidado — o padrão atende à maioria dos casos.
