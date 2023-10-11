# Visually.io SPA integration


- Add visually.io javascript SDK to document HEAD

```html
<!--  index.html  -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/src/assets/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
<!--    ...     -->

    <!--    VISUALLY SDK  -->
    <script type="text/javascript" rel="preconnect prefetch"
            src="https://sdk.loomi-prod.xyz/widgets/vsly-preact.min.js?k=ANALYTICS_KEY&e=2&s=STORE_ALIAS"></script>
    <script type="text/javascript" rel="preconnect prefetch"
            src="https://sdk.loomi-prod.xyz/v/visually-spa.js"></script>
    <script defer type="application/javascript" src="https://sdk.loomi-prod.xyz/v/visually-a-spa.js"></script>
    <!--    END OF VISUALLY SDK  -->

  </head>
  <body>
<!--    ...     -->
  </body>
</html>
```

notice the code between the `<!--    VISUALLY SDK  -->` comments

- ANALYTICS_KEY -> shopify store id
- STORE_ALIAS -> shopify customer alias ( for example, if your shopify store domain is banana-1.myshopify.com, your alias will be BANANA_1 )

---

- initialize visually.io sdk with an instrumentation tool

Visually.io requires access an instrumentation object that will allow us to control ('instrument') the online store using javascript.
In addition, Visually.io exposes some global methods that need to be called from React hooks ( for example )
When important events occur, such as 
- product added to cart
- user logged in


Below are the relevant typescript interfaces

```typescript
export interface CartBase {
  items: Array<{ variant_id: number, quantity: number, product_id: number, price: number }>
  token: string
  currency: string
  total_price: number
}

declare global {
  interface Window {
    visually: {
      onProductChanged: (productId: number, selectedVariantId: number, variantPrice: number) => void;
      onCurrencyChanged: (currency: 'home'|'product'|'catalog'|'other') => void;
      onPageTypeChanged: (currency: string) => void;
      onLocaleChanged: (locale: string) => void;
      onCartChanged: (cart: CartBase) => void;
      onUserIdChanged: (userId: string) => void;
      visuallyConnect: (instrument: VisuallyInstrument) => void
    }
  }

}
export interface VisuallyInstrument {
  openCartDrawer: () => void;
  closeCartDrawer: () => void;
  // should create cart if none
  addToCart: (variantId: number, quantity: number) => Promise<any>;

  initialProductId: number;    // optional - only for PDP pages, current product id
  initialVariantId: number;    // optional - only for PDP pages, current variant id
  initialVariantPrice: number; // optional - only for PDP pages, current variant price
  initialLocale: string;       // optional - initial locale - 'en-US' by default
  initialCurrency: string;     // optional - initial currency - 'USD' by default
  initialCart: CartBase,       // initial cart if user has a cart, ( SEE CartBase interface )
}
```


During initial page load, in the root of your application
we need to be initialized as follows
where instrumentationTool implements the `VisuallyInstrument` interface

```typescript
  useEffect(() => {
    window.visuallyConnect(instrumentationTool)
}, []);
```


In addition, there are the following global methods that need to be called when the following events occur:
- PDP navigation, variant selection
`onProductChanged?: (productId: number, selectedVariantId: number, variantPrice: number) => void;`
- currency chage
`onCurrencyChanged?: (currency: string) => void;`
- page type changed 
`onPageTypeChanged?: (currency: 'home'|'product'|'catalog'|'other') => void;`
- locale changed:
`onLocaleChanged?: (locale: string) => void;`
the locale should be a valid locale string, for example : `en-US`
https://www.science.co.il/language/Locale-codes.php
- on every cart update
`onCartChanged?: (cart: CartBase) => void;`
- on user signin
`onUserIdChanged?: (userId: string) => void;`