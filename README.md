##What is this?

This is a client implementation of Checkout Anywhere for iOS that allows you to perform checkouts from an App. This also supports ApplePay.

This library also includes an implementation of Shopify's Storefront API. This is an API that allows you to fetch products and collections from a specific shop (i.e. ______.myshopify.com).

## Flow

1. Create a cart.
2. Transform cart into a Checkout.
3. Add information to the Checkout.
4. Post a credit card to our card store (optional if using ApplePay).
5. Complete the checkout with the stored credit card (or an ApplePay token).
6. Poll until the payment job completes.
7. Bask in the glory of a created order.

## Flow Examples

For a view of what a 'synchronous' view of checkout would look like: see the [integration tests](https://github.com/Shopify/checkout-anywhere-ios/blob/master/CheckoutAnywhere/CheckoutAnywhereTests/CHKAnywhereIntegrationTest.m)

Here is a simplistic view:

```
//Create the checkout
[_checkoutDataProvider createCheckout:checkout block:...]

//Fetch shipping rates as necessary
[_checkoutDataProvider getShippingRatesForCheckout:checkout block:...]

//Update the checkout with any updated information (refetch shipping rates if the addresses change)
[_checkoutDataProvider updateCheckout:checkout completion:...] (if necessary)

//Add a payment method and complete the checkout 
[_checkoutDataProvider storeCreditCard:creditCard checkout:checkout completion:...]
[_checkoutDataProvider completeCheckout:checkout block:...]

//Or directly complete the payment using an ApplePay token (PKPaymentToken *)
[_checkoutDataProvider completeCheckout:checkout withApplePayToken:token block:...]

//Poll for completion -- this may take a few seconds
[_checkoutDataProvider getCompletionStatusOfCheckout:checkout block:...]

//Update the checkout to obtain the final information if necessary
[_checkoutDataProvider getCheckout:checkout completion:...]
```

### Error Handling

Please ensure to verify the `error` object returned with every call back to Shopify. If there was an error along the way, the error object will be populated with a dictionary containing a mapping of fields->errors.
This will give you detailed information about what fields are missing, or what fields are wrong.

For example, if you were to complete a checkout with an **email**, the `error.userInfo` will be:

```
{
  "errors" = {
    "checkout" = {
      "email" = [           //One error associated with a "blank" (or missing) email.
		{
			"code" = "blank",
			"message" = "can't be blank"
        }
      ],
	"line_items" = [],      //No errors associated with this
    "shipping_rate_id" = [] //No errors associated with this
  }
}
```

Pods
====
This repo is also a CocoaPod.

Make sure to run `pod spec lint ShopifyCheckoutAnywhere.podspec` before publishing, and update this README.md to be public facing.
