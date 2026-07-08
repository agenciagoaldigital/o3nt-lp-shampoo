# LP — Shampoo Antiqueda O3NT

Landing page standalone (HTML único, CSS/JS inline) para o Shampoo Antiqueda O3NT.

- **Hospedagem:** `shampoo.o3nt.com`
- **Checkout:** WooCommerce da loja `o3nt.com` (gateway Itaú/Rede já ativo), via links "comprar agora".
- **Arquivo:** [`index.html`](index.html) — é só publicar esse arquivo.

## Como o checkout funciona

Cada botão aponta para:

```
https://o3nt.com/carrinho/?add-to-cart=<ID>&buy_now=1
```

O parâmetro `buy_now=1` é lido pelo snippet PHP abaixo (na loja): ele esvazia o
carrinho, deixa só o item clicado e manda direto pro checkout. **Sem** o snippet a
compra ainda funciona, mas passa pela página de carrinho antes.

### Mapeamento produto → ID (WooCommerce)

| Botão na LP            | Produto WooCommerce                | ID     | Preço       |
|------------------------|------------------------------------|--------|-------------|
| 1 frasco               | Shampoo Antiqueda O3NT             | 7892   | R$ 233,00   |
| 2 frascos              | 2 Unidades do Shampoo Antiqueda    | 15504  | R$ 396,00   |
| 3 frascos              | 3 Unidades do Shampoo Antiqueda    | 15506  | R$ 558,00   |
| Kit Hair Care          | Sérum + Shampoo Antiqueda          | 7916   | R$ 430,35   |
| Sérum Capilar          | Sérum Capilar Tonificante          | 7177   | R$ 220,00   |
| Kit Completo           | Kit Completo O3NT                  | 11390  | R$ 1.218,75 |

## Snippet PHP necessário na loja (o3nt.com)

Cole em **Code Snippets** (Run everywhere) ou no `functions.php` do tema filho.
Só age quando o link traz `&buy_now=1` — não altera o comportamento normal da loja.

```php
// === O3NT — "Comprar agora" da landing page ===
add_action( 'woocommerce_add_to_cart', function ( $added_key ) {
    if ( empty( $_REQUEST['buy_now'] ) ) return;
    foreach ( WC()->cart->get_cart() as $key => $item ) {
        if ( $key !== $added_key ) {
            WC()->cart->remove_cart_item( $key );
        }
    }
}, 10, 1 );

add_filter( 'woocommerce_add_to_cart_redirect', function ( $url ) {
    if ( ! empty( $_REQUEST['buy_now'] ) ) {
        return wc_get_checkout_url();
    }
    return $url;
} );
```

## Rastreamento (GTM)

- Container **GTM-MSDHGGGW** instalado no `<head>` + `<noscript>`.
- Pixel Meta e Google (GA4/Ads) devem ser criados **como tags dentro do GTM**.
- A LP dispara `begin_checkout` no `dataLayer` ao clicar em comprar
  (`currency`, `value`, `items[].item_id`) — pronto pra virar InitiateCheckout (Meta)
  / begin_checkout (GA4) no GTM.
- **Purchase** ocorre no checkout do `o3nt.com` — depende do GTM/pixel estar
  também instalado lá.

## Pendências fora deste repositório

- [ ] Instalar o snippet PHP na loja `o3nt.com`.
- [ ] Configurar as tags Meta/Google dentro do GTM.
- [ ] Conferir se cupom `primeiracompra` (10%) e desconto PIX (5%) existem no
      WooCommerce — a LP anuncia esses textos.
- [ ] Conferir se as regras de frete grátis (R$299 Sul/Sudeste, R$399 Brasil)
      batem com a configuração de frete do WooCommerce.
- [ ] Apontar o DNS de `shampoo.o3nt.com` para onde a LP for publicada.
