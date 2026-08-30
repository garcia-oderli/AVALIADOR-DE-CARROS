# Avaliador de Carro Usado — RitmoProd

PWA de página única para avaliar anúncios de carro usado: puxa a FIPE do mês ao
vivo, projeta o custo total de posse no seu horizonte de uso e devolve o preço
máximo que ainda faz sentido pagar.

**Stack:** HTML/CSS/JS puro, sem build e sem dependências. Todo o app está em
`index.html`.

## Como se usa — quatro abas

| Aba | O que faz |
|---|---|
| **Perfil** | Parâmetros de uso: km/ano, combustível, IPVA, orçamento, horizonte, score mínimo. Define uma vez. |
| **Pagamento** | Seu carro na troca e a forma de pagamento — entrada, prazo, juros, parcela máxima. |
| **Anúncio** | Escolhe o carro na FIPE (que **monta a ficha técnica sozinha**) e preenche preço e km. |
| **Resultado** | Score, teto de negociação, composição do custo e o comparativo de anúncios salvos. |

Uma aba por vez, com uma barra de contexto embaixo que mostra o carro escolhido e
os parâmetros ativos em qualquer aba. Primeira visita abre no Perfil; depois, no
Anúncio. Avaliar leva direto para o Resultado.

Marca e modelo são campos de busca — filtram por termo, ignoram acento
(`citroen` acha `Citroën`) e aceitam vários termos (`virtus tsi`).

### Os dois números da negociação

São coisas diferentes e confundi-las custa dinheiro:

- **Folga até o seu teto** — quanto ainda caberia pagar sem furar seu critério.
  Não é margem de negociação.
- **Negociação vs alvo** — a distância até o preço que você deve mirar. É esse
  que se leva para a mesa.

## Meu carro na troca

Consulta o seu carro na mesma FIPE ao vivo e compara com o que a loja está
oferecendo. **Dar o carro na troca não barateia o negócio** — você entrega um bem
no valor dele. O que custa é o deságio: a loja dá desconto no carro novo e retoma
no seu. Esse valor entra no custo total e vira alerta acima de 8% da FIPE.

## Financiamento

Tabela Price: entrada em dinheiro, carro na troca, prazo, juros ao mês e parcela
máxima. Sai parcela, valor financiado, juros totais e o caixa que você precisa ter
no ato.

Os juros pagos **dentro do horizonte** entram no custo total — o prazo do contrato
quase nunca coincide com o horizonte de posse, então o cálculo amortiza mês a mês
e considera o saldo devedor no fim.

Com prazo 0 o app trata como compra à vista e o teto sai do orçamento. Financiando,
o teto passa a sair da parcela máxima.

## Ficha técnica automática

O nome que a FIPE devolve (`VIRTUS 1.6 MSI Flex 16V 4p Aut.`) carrega cilindrada,
aspiração e câmbio. O app lê isso e cruza com um catálogo de famílias de motor
para preencher o anúncio sem digitação.

O que o nome **nunca** diz é o sincronismo — e é ele que separa R$ 0 de R$ 3.600
no ato da compra. Por isso o sincronismo só sai do catálogo de famílias
identificadas (EA211 da VW, 1.0/1.2 turbo da GM, EcoBoost da Ford, PureTech da
PSA, Firefly e E.torQ da Fiat, corrente nos japoneses e coreanos). Motor que não
casa com nenhuma família fica marcado **"não identificado — confirmar"**, com
alerta em vermelho e sem custo de sincronismo lançado.

Preencher "corrente" por omissão esconderia exatamente o risco que o app existe
para achar, então ele prefere admitir que não sabe. O consumo é estimado por
cilindrada e aspiração, e a interface diz que é estimativa: se o anúncio ou a
etiqueta do Inmetro trouxer o número real, corrija o campo.

Carro elétrico é recusado explicitamente — o app custeia combustível em km/L e
sincronismo de motor, e nada disso se aplica.

A lista fixa de modelos continua disponível para sobrepor tudo à mão.

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

1. Suba `CACHE` em `sw.js` (`avaliador-v8` → `avaliador-v9`)
2. Atualize o badge de versão no `index.html`
3. Publique

Desde a v4 o HTML é buscado pela rede primeiro, então uma versão nova chega
mesmo sem trocar o nome do cache — o bump continua sendo a garantia para os
ícones e o manifesto.

---

Oderli Garcia © 2026 · RitmoProd
