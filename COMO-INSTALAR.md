# Avaliador de Carro Usado — instalação no celular

## Por que precisa ser hospedado

Para o Android/iOS oferecer "Instalar app", a página precisa estar em **HTTPS**.
Abrindo o arquivo direto (`file://`) o service worker não registra e o botão de
instalar não aparece. Só hospedar resolve — é o mesmo caminho do GastosIA.

## Opção 1 — Vercel (recomendado, ~2 min)

1. Crie uma pasta com os 6 arquivos: `index.html`, `manifest.json`, `sw.js`,
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
- A tela é dividida em três etapas recolhíveis: **1 Perfil**, **2 FIPE ao vivo**,
  **3 Análise** — cada uma mostra um resumo quando fechada
- Marca e modelo são campos de busca: digite parte do nome, sem se preocupar com acento
- Na etapa 1 dá para pôr **seu carro na troca** (consultado na mesma FIPE) e simular
  **financiamento** — parcela, juros e o caixa necessário no ato
- Escolher marca/modelo/ano na etapa 2 preenche sozinho a ficha técnica do anúncio
  (motor, câmbio, sincronismo e consumo) — sem lista manual
- Funciona offline, **exceto** a consulta FIPE, que precisa de internet
- Sem internet: o campo FIPE continua editável, você digita e a conta roda igual
- **Imprimir / salvar PDF** no resultado gera uma folha limpa da avaliação;
  **Imprimir comparativo** faz o mesmo com a lista de anúncios salvos

## Atualizar depois

Desde a **v4** o `index.html` é buscado pela rede antes do cache, então uma
versão nova chega ao celular sozinha na próxima abertura com internet. Offline,
o cache assume e o app abre igual.

Ainda assim, ao publicar suba a linha 2 do `sw.js` — é o que renova ícones e
manifesto, que continuam vindo do cache:

    const CACHE = 'avaliador-v7';

## Fonte da FIPE

API pública `parallelum.com.br/fipe/api/v1` — gratuita, sem chave, CORS liberado.
Tem limite de requisições por minuto. Se estourar, a tela avisa ("limite de
consultas atingido") e você digita o valor. Sem internet ela avisa "sem conexão"
e a consulta expira em 8 segundos em vez de travar.
Confira sempre se a versão retornada (combustível, câmbio) é a mesma do anúncio.

Oderli Garcia © 2026 · RitmoProd
