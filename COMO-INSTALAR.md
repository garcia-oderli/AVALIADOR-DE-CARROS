# Avaliador de Carro Usado — instalação no celular

## Por que precisa ser hospedado

Para o Android/iOS oferecer "Instalar app", a página precisa estar em **HTTPS**.
Abrindo o arquivo direto (`file://`) o service worker não registra e o botão de
instalar não aparece. Só hospedar resolve — é o mesmo caminho do GastosIA.

## Opção 1 — Vercel (recomendado, ~2 min)

1. Crie uma pasta com os 5 arquivos: `index.html`, `manifest.json`, `sw.js`,
   `icon-192.png`, `icon-512.png`, `icon-180.png`
2. No terminal, dentro da pasta:

       npx vercel --prod

3. Abra a URL gerada no celular
4. **Android/Chrome:** menu ⋮ → "Instalar app"
   **iOS/Safari:** botão compartilhar → "Adicionar à Tela de Início"

## Opção 2 — arrastar no Vercel

Acesse vercel.com/new, arraste a pasta, publica sozinho.

## Opção 3 — GitHub Pages

Suba os arquivos num repositório, Settings → Pages → branch main.
A URL sai como `https://garcia-oderli.github.io/<repo>/`.

## Depois de instalar

- Abre em tela cheia, sem barra de navegador
- Parâmetros e anúncios salvos ficam no aparelho (localStorage)
- Funciona offline, **exceto** a consulta FIPE, que precisa de internet
- Sem internet: o campo FIPE continua editável, você digita e a conta roda igual

## Atualizar depois

Ao publicar uma versão nova, mude a linha 2 do `sw.js`:

    const CACHE = 'avaliador-v4';

Sem trocar esse nome, o celular continua servindo a versão antiga do cache.

## Fonte da FIPE

API pública `parallelum.com.br/fipe/api/v1` — gratuita, sem chave, CORS liberado.
Tem limite de requisições por minuto. Se falhar, a tela avisa e você digita o valor.
Confira sempre se a versão retornada (combustível, câmbio) é a mesma do anúncio.

Oderli Garcia © 2026 · RitmoProd
