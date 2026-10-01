# Human Social

Voce esta operando o **Human Social**, um sistema white label para desdobrar uma pasta com texto e imagens em pecas nativas para Instagram Feed, Instagram Stories e LinkedIn Feed.

## Caminhos obrigatorios

Antes de agir, resolva estes caminhos:

- `SOCIAL_HOME`: a **raiz desta pasta** — onde estao este `AGENTS.md`, o `CLAUDE.md` e a pasta `scripts/`. Se o Claude Code foi aberto aqui dentro, e `.`. Se foi aberto em outro projeto, e o caminho absoluto ate esta pasta.
- `SCRIPT`: `${SOCIAL_HOME}/scripts/desdobrar.py`.
- `INPUT_FOLDER`: pasta passada pelo usuario no comando.

Nunca assuma que o diretorio atual e o mesmo que `SOCIAL_HOME`. Em uso global, o usuario pode chamar `/social` ou `/desdobrar` de qualquer projeto. Por isso, todos os comandos do script devem passar por `SOCIAL_HOME`:

```bash
python3 "$SOCIAL_HOME/scripts/desdobrar.py" ...
```

Esta pasta e autocontida: nao depende de variavel de ambiente, de outro repositorio nem de pasta fora daqui.

## Leitura obrigatoria

1. `${SOCIAL_HOME}/CLAUDE.md`
2. `${SOCIAL_HOME}/.claude/skills/desdobrar/SKILL.md`

O arquivo `${SOCIAL_HOME}/.Codex/skills/desdobrar/SKILL.md` existe apenas como compatibilidade com roteadores antigos e aponta para a skill canonica acima.

## Regras principais

- O fluxo e white label. Nao usar exemplos, textos, imagens ou marcas antigas como default.
- Todo desdobramento visual usa um de dois motores: **Higgsfield CLI** (com `gpt_image_2`) ou **Magnific MCP**. A ordem de resolucao esta em `${SOCIAL_HOME}/providers.md` — se os dois estiverem disponiveis, **pergunte ao usuario** qual usar, uma vez por execucao.
- Um motor por execucao. Nunca troque no meio, nem como fallback de erro.
- Desdobramento nao e redesign: preserve a identidade visual, a fonte, as cores, os elementos graficos, o logo/assinatura do usuario e a logica de composicao da arte-mae.
- Toda geracao visual recebe a mesma arte-mae como primeira referencia: `--base` no Higgsfield, `base_reference` no Magnific.
- Use `--reference-mode base-only` como padrao. So use `--reference-mode all` quando as outras imagens forem deliberadamente parte do mesmo sistema visual.
- Prompt visual deve ser curto e **identico nos dois motores**: formato destino, texto exato e ajustes permitidos. O modelo ja entende a imagem enviada.
- No caminho Magnific, o prompt vem de `build-prompt` e o resultado e gravado por `save-external`. Nunca use o `_prompts/*.txt` cru nem baixe o arquivo na mao.
- Os elementos visiveis da arte-mae devem permanecer em Feed e LinkedIn: foto/fundo, fonte, paleta, logo, grafismos e composicao. Variar formato e texto nao pode virar uma nova direcao visual.
- Stories sao excecao controlada: os 3 frames usam a arte-mae como identidade, mas precisam ter textos e imagens/backgrounds diferentes. A imagem de fundo nao pode ser igual nos 3. Pode variar crop, cena, fundo, angulo, pose, iluminacao e area de texto, mantendo campanha, marca, fonte, paleta e linguagem.
- As imagens finais devem nascer integradas: imagem + design + lettering + texto no mesmo render.
- O agente deve escrever todas as copies finais: Instagram Feed, roteiro de Stories e LinkedIn.
- Toda execucao concluida gera `desdobramento/apresentacao-desdobramento.pdf`.
- Toda execucao concluida tambem sincroniza `desdobramento/output/`, uma pasta limpa com apenas os finais para o usuario.
- A saida do usuario fica em `INPUT_FOLDER/desdobramento/`, nunca dentro da pasta central do Human Social.

## Fluxo minimo

1. Validar que `${SCRIPT}` existe.
2. Rodar `python3 "${SCRIPT}" check-cli`.
3. Rodar `python3 "${SCRIPT}" prep "${INPUT_FOLDER}"`.
4. Analisar visualmente as imagens de entrada e escolher uma arte-mae.
5. Criar prompts curtos: formato destino + texto exato + o que preservar.
6. Rodar `generate` para IG Feed, 3 Stories e LinkedIn usando a mesma arte-mae em `--base`; nos Stories, pedir variacao real de imagem/background/cena entre os 3 frames.
7. Atualizar `manifest.json`.
8. Rodar `python3 "${SCRIPT}" presentation-pdf "${INPUT_FOLDER}"`.
9. Responder com caminhos absolutos dos arquivos finais e da pasta limpa `desdobramento/output/`.
