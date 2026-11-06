---
name: motion-reel
description: Use when the user asks to create a motion video, showreel, animated explainer, product reel or motion ad rendered entirely from code (no After Effects, no video model).
---

# Motion reel

Faz vídeos de motion design escrevendo uma pagina HTML com uma funcao pura `window.seek(t)` que desenha qualquer quadro, renderiza com um navegador headless (Playwright) e junta os quadros em video com ffmpeg. Isso só roda em um agente com shell (Claude Code), nao no chat.

## Passo 0: setup (uma vez por projeto)

```bash
brew install node ffmpeg python   # ou apt install no Linux
pip install numpy librosa soundfile
mkdir motion-studio && cd motion-studio && npm init -y
npm i -D playwright && npx playwright install chromium
```

Crie um `CLAUDE.md` na raiz do projeto com estas regras (elas valem para todo video feito ali, sem precisar repetir):

```
# Motion studio rules

## Render contract
- Cada video e uma funcao pura do tempo: `window.seek(t)` pinta o frame t.
- Sem CSS transitions, sem setTimeout, sem requestAnimationFrame no modo render,
  sem estado carregado entre frames. Ruido com seed fixa (mulberry32), nunca Math.random.
- Renderizar com `node render.mjs`, encode H.264 yuv420p, CRF 16.

## Visual
- Banido: titulo centralizado em gradiente, tudo com fade in, labels e bordas
  no canto, brilho (glow) em elementos de UI, particulas genericas.
- Uma fonte de display, uma fonte de UI. Uma cor de destaque, a menos que
  o briefing diga outra coisa.
- Algo novo tem que acontecer na tela a cada 2 a 4 segundos.

## Som
- Trilha e efeitos sonoros sintetizados em codigo, a menos que uma faixa
  seja fornecida. Hits no beat medido (beats.json). Loudness -14 LUFS.

## Loop antes de mostrar qualquer coisa
1. Renderizar um frame por beat como contact sheet e OLHAR pra ele.
2. Dar nota de 1 a 10 em: gancho nos 2s iniciais, legibilidade em tela de
   celular, qualidade do movimento, variedade, fidelidade a marca, sincronia com o som.
3. Corrigir os 3 piores problemas. Repetir até toda nota ficar 8+.
4. Só entao fazer o render completo.
```

## Inputs a coletar antes de animar

Pergunte isso em uma unica mensagem, sem perguntar de novo depois:
- Produto/tema + URL (se houver)
- Duracao e formatos (9:16 vertical, 1:1 quadrado, 16:9 horizontal)
- Cores da marca, fontes, uma cor de destaque
- Uma referencia visual (um frame, um video, ou uma pasta de imagens/trabalhos antigos)
- Musica (arquivo existente ou "sintetizar")

## Pipeline

1. **Assets**: se houver URL, visite o site com Playwright e salve screenshots reais, logo e cores em `./assets`. Nunca redesenhe a UI de memoria, recorte e anime a UI real.
2. **Referencia** (se houver): extraia frames com ffmpeg (`ffmpeg -i ref.mp4 -vf fps=2 refs/frame_%03d.png`) e escreva `docs/style_guide.md` com paleta (hex), tipografia, duracao dos planos, tipo de transicao, textura. Pegue a gramatica da referencia, nunca o conteudo/logo/personagens dela.
3. **Musica**: se foi fornecida, meça o beat com librosa (`beats.py` gera `beats.json`: bpm, beats, downbeats, hits). Se não, sintetize no mesmo timeline do video.
4. **Roteiro**: escreva `docs/shotlist.md` no grid de beats — um plano por beat, algo novo a cada 2-4s. Mostre a lista de planos e espere confirmação antes de escrever codigo.
5. **Engine**: construa `index.html` com `window.seek(t)` usando springs fechados (damped spring, nunca easing generico), e `render.mjs` (Playwright + ffmpeg) seguindo o `CLAUDE.md`.
6. **Critique loop** (minimo 3 rodadas): gere um contact sheet (`ffmpeg -i out/final.mp4 -vf "fps=2,scale=270:-1,tile=6x5" -frames:v 1 out/contact.png`), olhe pra ele, liste os 3 piores problemas (texto sobrepondo, algo deslizando em vez de suavizar, label no canto, frame morto, solução no loop), corrija, re-renderize só os segundos afetados.
7. **Audio**: gere/mixe os efeitos sonoros no beat grid, normalize para -14 LUFS.
8. **Entrega**: renderize todos os formatos pedidos a partir do mesmo timeline (nao recorte um 16:9 pra virar vertical — reposicione tipografia e UI por formato). Entregue `out/final.mp4`, `out/contact.png`, `out/poster.png`, e diga o que melhoraria numa proxima rodada.

## Regras duras
- UI de produto real, nunca inventada.
- Sem Math.random, sem timers, sem CSS transitions no modo render.
- Banido: labels de canto, titulo centralizado em gradiente, tudo com fade in.
