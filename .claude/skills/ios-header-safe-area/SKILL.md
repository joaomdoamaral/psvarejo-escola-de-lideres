---
name: ios-header-safe-area
description: Evita ou corrige o conteúdo do site aparecendo acima do header no iPhone (área da câmera/Dynamic Island, barra de status, faixa "< WhatsApp" do navegador interno). Use ao começar uma landing page/site com header fixo, ou quando aparecer conteúdo acima do header ou em volta da câmera no Safari/WhatsApp.
---

# Header cobrindo a área da câmera no iPhone

Sintoma: no iPhone (Safari ou navegador interno do WhatsApp/Instagram), o conteúdo da página
aparece rolando acima do header, na faixa da barra de status e dos lados da câmera.

Causa: o Safari pinta essa faixa com a cor do **elemento fixo que está mais por cima na borda de
cima da tela**. Se esse elemento for transparente (ou não existir), ele mostra o site por trás.
Os culpados mais comuns:
- camadas decorativas fixas de tela cheia por cima de tudo (granulado/noise, vinheta, brilho,
  `body::after{position:fixed;inset:0 ou -50%;z-index:alto}`);
- header transparente ou com `backdrop-filter` no topo da página (comum em celular deitado/tablet,
  quando o "header sólido" só vale abaixo de certa largura);
- header que não inclui a área segura (`env(safe-area-inset-top)`) na própria altura.

## Checklist (aplique tudo)

1. **Meta viewport** com `viewport-fit=cover` e `theme-color` na cor do header:
   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
   <meta name="theme-color" content="#04050b">
   ```
2. **Variáveis** (troque `76px` pela altura do header):
   ```css
   :root{
     --header-h:76px;
     --safe-top:env(safe-area-inset-top,0px);
     --header-total:calc(var(--header-h) + var(--safe-top));
   }
   html{background:<cor do header>;scroll-padding-top:var(--header-total)}
   body{background:<cor do fundo>}
   ```
3. **Header** fixo, ocupando a área segura, com cor sólida e uma extensão para cima
   (cobre o vão ao puxar a página ou quando a barra do navegador anima):
   ```css
   .header{position:fixed;top:0;left:0;right:0;z-index:50;box-sizing:border-box;
     height:var(--header-total);padding-top:var(--safe-top);background-color:<cor>}
   .header > .inner{height:var(--header-h)}
   .header::before{content:"";position:absolute;left:0;right:0;
     bottom:calc(100% - var(--safe-top));height:calc(var(--safe-top) + 200px);
     background:<cor>;pointer-events:none}
   ```
4. **Header sólido em qualquer tela de toque**, não só abaixo de uma largura. Sem transparência
   nem `backdrop-filter` (efeito vidro):
   ```css
   @media (max-width:760px),(hover:none) and (pointer:coarse){
     .header,.header.scrolled{background:<cor>;backdrop-filter:none;-webkit-backdrop-filter:none}
   }
   ```
5. **Nada fixo e transparente acima do header na borda de cima.** Procure `position:fixed` e
   `z-index` maiores que o do header. Camadas decorativas (granulado etc.) devem começar abaixo do
   header e ficar abaixo dele:
   ```css
   body::after{position:fixed;top:var(--header-total);left:0;right:0;bottom:0;z-index:45}
   ```
   Para animar um granulado, anime `background-position`, não `transform` com `inset` negativo.
6. **Tudo que depende do header usa `--header-total`:** `scroll-padding-top`, `top` de
   elementos `sticky`, `padding-top` do hero/primeira seção, `scroll-margin-top` de âncoras.

## Como verificar

- No navegador (Playwright, iPhone emulado), o elemento mais alto na borda de cima precisa
  ser o header em toda a largura:
  ```js
  [5, innerWidth/2, innerWidth-5].map(x => document.elementsFromPoint(x, 1)[0])
  ```
- Simule a área da câmera (o Chromium não emula `env()`): `:root{--safe-top:47px !important}`.
  Confira em pé e deitado, no topo e no meio da rolagem.
- No iPhone real, abra pelo Safari e pelo WhatsApp com um parâmetro novo no link
  (`?v=2`, `?v=3`…) para não pegar a versão antiga guardada.
