## Pack-Digital SPA Integration

Welcome to our comprehensive tutorial on integrating Visually.io into your pack-digital headless storefront. This guide will walk you through the process step by step.  
DISCLAIMER: this is an example of integration to the most basic sturcture of pack-digital, if your code looks different please contact our technial support team.

Create the following `VisuallyIo.tsx` file anywhere you want, this file will setup our analytics tracking & cart manipulation components.
Notice that you'll need to replace the `<API_KEY>` and `<ALIAS>` placeholders with the ones given to you by our support team.

```javascript
 // VisuallyIo.tsx
import React, { useEffect } from 'react';
import {
  useCart,
  useCartAddItem,
  useCartClear,
  useCurrency,
} from '@backpackjs/storefront';
import Script from 'next/script';
import { useGlobalContext } from '../contexts';

function getPageType(resourceType) {
  const pageType = 'other';
  if (resourceType === 'home_page') {
    return 'home';
  }
  if (resourceType === 'product_page') {
    return 'product';
  }
  if (resourceType === 'collection_page') {
    return 'collection';
  }
  return pageType;
}

function maybe(f, def = undefined) {
  try {
    return f();
  } catch {
    return def;
  }
}

function transformCart(cart) {
  return cart
    ? {
        item_count: maybe(() => cart.lines.reduce((p, c) => p + c.quantity, 0)),
        items: cart.lines.map((l) => ({
          handle: l.variant.product.handle,
          price: maybe(() => parseInt(l.variant.priceV2.amount, 10)),
          product_id: maybe(() =>
            parseInt(
              l.variant.product.id.replace('gid://shopify/Product/', ''),
              10
            )
          ),
          quantity: l.quantity,
          variant_id: maybe(() =>
            parseInt(
              l.variant.id.replace('gid://shopify/ProductVariant/', ''),
              10
            )
          ),
        })),
        currency: maybe(
          () => parseInt(cart.estimatedCost.totalAmount.currencyCode, 10),
          0
        ),
        total_price: maybe(
          () => parseInt(cart.estimatedCost.totalAmount.amount, 10),
          0
        ),
        token: maybe(() => cart.id.replace('gid://shopify/Cart/', ''), ''),
      }
    : undefined;
}

function useVisuallyIo({ product, resourceType }) {
  const currency = useCurrency();
  const cart = useCart();
  const { cartClear } = useCartClear();
  const { cartAddItem } = useCartAddItem();
  const pageType = getPageType(resourceType);
  const {
    actions: { openCart },
  } = useGlobalContext();

  useEffect(() => {
    maybe(() => window.visually.onCartChanged(transformCart(cart)));
  }, [cart]);

  useEffect(() => {
    maybe(() =>
      window.visually.visuallyConnect({
        cartClear,
        initialProductId: maybe(() =>
          product.id.replace('gid://shopify/Product/', '')
        ),
        initialVariantPrice: maybe(() =>
          parseInt(product.variants[0].priceV2.amount, 10)
        ),
        initialVariantId: maybe(() =>
          product.variants[0].replace('gid://shopify/ProductVariant/', '')
        ),
        addToCart: (variantId, quantity) =>
          cartAddItem({
            merchandiseId: `gid://shopify/ProductVariant/${variantId}`,
            quantity,
          }),
        openCartDrawer: openCart,
        pageType,
        initialCurrency: currency,
        initialLocale: 'en-US',
      })
    );
  }, [pageType, currency, product]);
}

export function VisuallyIo({ page, product }) {
  useVisuallyIo({
    product,
    resourceType: page?.resourceType,
  });
  return (
    <>
      <Script
        strategy="beforeInteractive"
        rel="preconnect prefetch"
        src="https://sdk.loomi-prod.xyz/widgets/vsly-preact.min.js?k=<API_KEY>&e=2&s=<ALIAS>"
      />
      <Script
        strategy="beforeInteractive"
        rel="preconnect prefetch"
        src="https://sdk.loomi-prod.xyz/v/visually-spa.js"
      />
      <Script
        strategy="afterInteractive"
        src="https://sdk.loomi-prod.xyz/v/visually-a-spa.js"
      />
      <span />
    </>
  );
}
```

Afterwards you'll need to add the component we just created to you main global context:

```javascript
// layouts/Storefront.jsx
// ...
  return (
    <GlobalContextProvider>
        <StorefrontHead />
        .
        .
        .
        <VisuallyIo {...props} /> 
        .
        .
        .
    </GlobalContextProvider>
)
```

And that's basically it. 
The SDK connects to Visually.io api, runs all the experiences and reports all the analytics.


