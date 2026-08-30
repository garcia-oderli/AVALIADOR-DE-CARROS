# Avaliador de Carro Usado — RitmoProd

PWA de página única para avaliar anúncios de carro usado: puxa a FIPE do mês ao
vivo, projeta o custo total de posse no seu horizonte de uso e devolve o preço
máximo que ainda faz sentido pagar.

**Stack:** HTML/CSS/JS puro, sem build e sem dependências. Todo o app está em
`index.html`.

## O que ele calcula

| Saída | O que significa |
|---|---|
| **Score (0–100)** | Nota do anúncio: preço vs. FIPE, km vs. idade, tipo de sincronismo, câmbio, idade e consumo. |
| **Custo total** | Compra + entrada + combustível + IPVA + sincronismo + manutenção + seguro − revenda estimada. |
| **Custo por km** | Custo total dividido pela rodagem do período. |
| **Alvo** | Preço de abertura da negociação: FIPE menos os descontos que os riscos identificados justificam. |
| **Máximo** | Maior preço que ainda passa no seu score mínimo **e** cabe no orçamento. Acima dele, você sai da mesa. |

O consumo informado é de estrada. O campo **% rodagem em estrada** mistura
estrada e cidade (cidade ≈ 22% menos eficiente) por média harmônica — que é a
forma correta de combinar km/L de dois regimes.

## Arquivos

| Arquivo | Papel |
|---|---|
| `index.html` | App inteiro: interface, motor de cálculo, consulta FIPE, impressão. |
| `sw.js` | Service worker. HTML pela rede primeiro, resto do shell pelo cache. |
| `manifest.json` | Manifesto PWA. |
| `icon-*.png` | Ícones de instalação (192, 512, 180 para iOS). |
| `COMO-INSTALAR.md` | Como hospedar e instalar no celular. |

## Rodando localmente

```
python3 -m http.server 8777
```

Depois abra `http://localhost:8777`. Precisa de HTTP — em `file://` o service
worker não registra e o botão de instalar não aparece.

## Dados

- **FIPE ao vivo:** API pública `parallelum.com.br/fipe/api/v1`, sem chave, com
  limite de requisições por minuto. Se falhar, a tela avisa e o campo FIPE
  continua editável.
- **Seus dados:** parâmetros e anúncios salvos ficam no `localStorage` do
  aparelho. Nada é enviado para servidor. Use Exportar/Importar JSON para levar
  entre dispositivos.

## Impressão

Os botões **Imprimir / salvar PDF** (avaliação) e **Imprimir comparativo**
(lista) geram uma folha limpa em fundo branco, com cabeçalho identificando o
anúncio e os parâmetros usados. Controles e navegação saem da página impressa.

## Publicando uma versão nova

1. Suba `CACHE` em `sw.js` (`avaliador-v4` → `avaliador-v5`)
2. Atualize o badge de versão no `index.html`
3. Publique

Desde a v4 o HTML é buscado pela rede primeiro, então uma versão nova chega
mesmo sem trocar o nome do cache — o bump continua sendo a garantia para os
ícones e o manifesto.

---

Oderli Garcia © 2026 · RitmoProd
