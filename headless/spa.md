## Ecommerce SPA/PWA Integration

Welcome to our comprehensive tutorial on integrating Visually.io into your custom headless storefront. This guide will walk you through the process step by step.

### Add Visually.io SDK
To get started, you need to include the Visually.io runtime dependencies in the <head> section of your index.html file. Be sure to place these script tags as close to the beginning of the <head> tag as possible. Replace `ANALYTICS_KEY` and `STORE_ALIAS` with the values provided to you by Visually.io.

```html
<!--  index.html  -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/src/assets/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

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

### Notify our SDK on context changes:
To enable Visually.io to send analytics and track the user's journey throughout the session, you must set up the following hooks in your code:

```typescript
declare global {
  interface Window {
    visually: {
      onProductChanged: (productId: number, selectedVariantId: number, variantPrice: number) => void;
      onCurrencyChanged: (currency: string) => void;
      onPageTypeChanged: (pageType: 'home'|'product'|'catalog'|'other') => void;
      onLocaleChanged: (locale: string) => void;
      onCartChanged: (cart: CartBase) => void;
      onUserIdChanged: (userId: string) => void;
      visuallyConnect: (instrument: VisuallyInstrument) => void
    }
  }
}
```

For instance, when the currency changes, you should call:
```typescript
window.visually.onCurrencyChanged(currentCurrency || "USD")
```

### Create a Visually.io Instrument
To allow Visually.io to interact with various components on your storefront, you need to create an object that adheres to the following interface:

```typescript
interface CartBase {
  items: Array<{ variant_id: number, quantity: number, product_id: number, price: number }>
  token: string
  currency: string
  total_price: number
}

interface VisuallyInstrument {
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

After defining your instrument, invoke our bootstrap method during the initial page load at the root of your application:
```typescript
useEffect(() => {
  window.visuallyConnect(instrumentationTool)
}, []);
```

Here, instrumentationTool should implement the VisuallyInstrument interface. This ensures that Visually.io is properly integrated into your ecommerce SPA/PWA.


# Allowed domains

If the SPA has a security mechanism that allows the website to run only on specific domains
We require to add the following domains to the domains 'allow list'

- visually.io
- loomi.me
- vsly.local:8000


For any further questions or assistance, please don't hesitate to reach out to us.


