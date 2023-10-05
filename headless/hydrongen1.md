## Shopify's Hydrogen-1 Integration

In this tutorial we'll cover the process of installing Visually.io to your headless `hydrogen1` storefront.

### Add Visually.io SDK
Include Visually.io runtime dependecies in your index.html `<head>` section,
don't forget to replace `ANALYTICS_KEY` and `STORE_ALIAS` with the ones that Visually.io has given you.
```html
<!--  index.html  -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/src/assets/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hydrogen</title>
    <link rel="stylesheet" href="/src/styles/index.css" />
    <link rel="preconnect" href="https://cdn.shopify.com" />
    <link rel="preconnect" href="https://shop.app/" />
    <link rel="preconnect" href="https://hydrogen-preview.myshopify.com/" />

    <!--    VISUALLY SDK  -->
    <script type="text/javascript" rel="preconnect prefetch"
            src="https://sdk.loomi-prod.xyz/widgets/vsly-preact.min.js?k=ANALYTICS_KEY&e=2&s=STORE_ALIAS"></script>
    <script type="text/javascript" rel="preconnect prefetch"
            src="https://sdk.loomi-prod.xyz/v/visually-hydrogen.js"></script>
    <script defer type="application/javascript" src="https://sdk.loomi-prod.xyz/v/visually-hydrogen-a.js"></script>
    <!--    END OF VISUALLY SDK  -->

  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/@shopify/hydrogen/entry-client"></script>
  </body>
</html>
```

### Connect Visually.io to Your storefront

Create a new component:
```javascript
// src/components/VisuallySDK.client.tsx

import {ClientAnalytics, useCart} from '@shopify/hydrogen';
import {useEffect} from 'react';

export default function VisuallySDK() {
  const cartWithActions = useCart();
  const key = [cartWithActions.lines.map((v) => v.merchandise)].join();
  useEffect(() => {
    // toggleCartDrawer is a function that toggles the state (OPEN/CLOSED) of the cart drawer
    window.visually?.hydrogen1Connect?.(ClientAnalytics, cartWithActions, toggleCartDrawer);
  }, [key]);

  return null;
}
```

And add it to your `src/App.server.tsx` file:
```javascript
import {Suspense} from 'react';
import renderHydrogen from '@shopify/hydrogen/entry-server';
import {
  FileRoutes,
  type HydrogenRouteProps,
  PerformanceMetrics,
  PerformanceMetricsDebug,
  Route,
  Router,
  ShopifyAnalytics,
  ShopifyProvider,
  CartProvider,
} from '@shopify/hydrogen';

import {HeaderFallback} from '~/components';
import type {CountryCode} from '@shopify/hydrogen/storefront-api-types';
import {DefaultSeo, NotFound} from '~/components/index.server';
import VisuallyIO from '~/components/VisuallySDK.client';

function App({request}: HydrogenRouteProps) {
  const pathname = new URL(request.normalizedUrl).pathname;
  const localeMatch = /^\/([a-z]{2})(\/|$)/i.exec(pathname);
  const countryCode = localeMatch ? (localeMatch[1] as CountryCode) : undefined;

  const isHome = pathname === `/${countryCode ? countryCode + '/' : ''}`;

  return (
    <>
      <Suspense fallback={<HeaderFallback isHome={isHome} />}>
        <ShopifyProvider countryCode={countryCode}>
          <CartProvider countryCode={countryCode}>

              {/*USE VISUALLY IO CLIENT COMPONENT */}
               <VisuallyIO />
              {/*USE VISUALLY IO CLIENT COMPONENT */}

              <Suspense>
              <DefaultSeo />
            </Suspense>
            <Router>
              <FileRoutes
                basePath={countryCode ? `/${countryCode}/` : undefined}
              />
              <Route path="*" page={<NotFound />} />
            </Router>
          </CartProvider>
          <PerformanceMetrics />
          {import.meta.env.DEV && <PerformanceMetricsDebug />}
          <ShopifyAnalytics />
        </ShopifyProvider>
      </Suspense>
    </>
  );
}

export default renderHydrogen(App);
```

