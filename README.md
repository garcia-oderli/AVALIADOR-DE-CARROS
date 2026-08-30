# Avaliador de Carro Usado — RitmoProd

PWA de página única que responde a uma pergunta: **quanto esse carro vale para
mim, considerando risco, uso, custo e capacidade financeira — e qual é o máximo
que devo pagar?**

O produto final é a **decisão**, não o score. Score é meio.

**Stack:** HTML/CSS/JS puro, sem build e sem dependências. Tudo em `index.html`.

## Como se usa — cinco abas

| Aba | O que faz |
|---|---|
| **Perfil** | Uso (km/ano, % estrada, horizonte), limites de compra, custos de entrada e configurações avançadas. |
| **Pagamento** | Seu carro na troca e a forma de pagamento: entrada, prazo, juros, parcela máxima. |
| **Carro** | Escolhe o veículo na FIPE (que monta a ficha técnica sozinha), preenche preço, km, histórico, origem e cautelar. |
| **Resultado** | Decisão, escada de preços, estratégia de negociação, score explicado, alertas e custo total. |
| **Comparar** | Lista de anúncios salvos e comparação direta de até 4, com a melhor compra justificada. |

## Hierarquia de preços

Um conceito, um nome, um valor. Nunca dois números disputando o mesmo rótulo.

| Nome | O que é |
|---|---|
| `precoPedido` | O que o vendedor quer. |
| `precoFipe` | Referência de mercado — consultada ou informada, sempre identificada. |
| `precoAlvo` | Onde a negociação deve chegar: FIPE menos os descontos que os riscos justificam. Nunca acima do teto. |
| `tetoCaixa` | Maior preço que cabe no caixa disponível. |
| `tetoParcela` | Maior preço que a parcela máxima suporta, dado prazo e taxa. |
| `tetoFinanceiro` | **MENOR(tetoCaixa, tetoParcela)** |
| `tetoRisco` | Maior preço em que o carro ainda alcança o score mínimo. |
| `tetoFinal` | **MENOR(tetoFinanceiro, tetoRisco)** — o único "máximo que pode pagar". |
| `descontoNecessario` | Quanto falta negociar para caber no teto. |

A ordem **primeira oferta < preço alvo ≤ teto final** é garantida por construção
e verificada em teste.

## Decisão

`avaliarDecisao()` devolve uma entre:

- **COMPRAR** — preço no alvo ou abaixo, dentro do teto, sem pendência crítica
- **COMPRAR COM CONDIÇÃO** — números fecham, mas há pendência a resolver antes de assinar
- **NEGOCIAR** — acima do alvo ou do teto, com o desconto calculado
- **CARRO APROVADO · PAGAMENTO REPROVADO** — o problema não é o carro, é a forma de pagamento
- **DESCARTAR** — nenhum preço faz o carro alcançar o score mínimo, ou a cautelar reprovou

Score baixo no preço pedido **não** condena o carro: significa que o preço está
alto. Só é descarte quando `tetoRisco <= 0`.

## Score explicável

Soma ponderada de oito critérios, cada um com nota de 0 a 1. Os pesos são
constantes editáveis em Configurações, não números soltos no código.

| Critério | Peso |
|---|---:|
| Preço vs FIPE | 25 |
| Quilometragem | 15 |
| Histórico de manutenção | 15 |
| Risco mecânico | 15 |
| Câmbio | 10 |
| Idade | 10 |
| Origem/uso | 5 |
| Consumo | 5 |

A tela mostra a barra de cada critério e quantos pontos foram perdidos nele.
Só o critério de preço depende do preço — é o que torna o score monotônico e
permite a busca binária do teto de risco.

## O que o app se recusa a fingir

- **Sincronismo** só sai de um catálogo de famílias de motor identificadas. Motor
  fora dele fica "não identificado", com alerta — nunca um chute.
- **Correia banhada a óleo** não é troca automática: depende da comprovação do
  óleo, do km e do intervalo.
- **Nenhum câmbio é condenado pelo tipo.** CVT e DCT partem de base menor, mas
  km rodado e histórico movem a nota.
- **Corrente** aparece como "sem troca preventiva programada" — o que não é o
  mesmo que ausência de risco.
- **FIPE informada à mão** se identifica como manual, nunca como consulta.
- **Carro elétrico** é recusado: o app custeia km/L e sincronismo de motor.

## Custo total de propriedade

Compra (preço, transferência, cautelar, revisão) + uso (combustível, IPVA,
seguro, manutenção preventiva, pneus, freios, provisão de risco, sincronismo)
− revenda. Sai como custo total, custo por km e custo médio mensal, no horizonte
escolhido (1, 2, 3, 5 ou 7 anos).

A revenda tem premissa declarada na tela; a troca não barateia o negócio — o que
custa é o deságio que a loja tira do seu carro.

## Financiamento

Tabela Price: `PMT = PV·i·(1+i)^n / ((1+i)^n − 1)`. A parcela máxima limita o
valor financiável de verdade (`financiavelMax`), e os juros contados no custo
total são os pagos **dentro do horizonte** — o contrato quase nunca coincide com
ele, então a amortização roda mês a mês.

## Autoteste

`rodarTestes()` — 29 verificações cobrindo preço vs FIPE, orçamento, score
mínimo, financiamento, correia, cautelar, histórico, os dois tetos, importação e
exportação, monotonicidade do score, identidades do TCO e ausência de NaN.
Botão **Rodar autoteste** em Configurações avançadas.

## Arquivos

| Arquivo | Papel |
|---|---|
| `index.html` | App inteiro, em módulos lógicos. |
| `sw.js` | Service worker. HTML pela rede primeiro, resto do shell pelo cache. |
| `manifest.json` | Manifesto PWA. |
| `icon-*.png` | Ícones de instalação. |
| `COMO-INSTALAR.md` | Como hospedar e instalar no celular. |

## Rodando localmente

```
python3 -m http.server 8777
```

Precisa de HTTP — em `file://` o service worker não registra.

## Dados

FIPE pela API pública `parallelum.com.br/fipe/api/v1`, sem chave, com limite por
minuto; se falhar, a tela avisa e o campo continua editável. Parâmetros e
anúncios ficam no `localStorage` do aparelho, com versionamento e migração
(`DATA_VERSION`). Nada vai para servidor.

## Publicando uma versão nova

1. Suba `CACHE` em `sw.js`
2. Atualize `APP_VERSION` no `index.html`
3. Publique

Desde a v4 o HTML vem da rede primeiro, então a versão nova chega sozinha.

---

Oderli Garcia © 2026 · RitmoProd
