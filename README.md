# Cordova Apple Pay Plugin
> An adapted Cordova plugin to provide Apple Pay functionality.

Updated to provide additional data access to the plugin, test calls, and compatibility
with newer versions of Cordova. Uses a Promise based interface in JavaScript.

## Installation
```
$ cordova plugin add cordova-applepay
```

Install the plugin using Cordova 6 and above, which is based on npm. The plugin exposes the `org.cordova.applepay` plugin, accessible in the browser as `window.ApplePay`.

To make changes to this plugin, update the version in the `package.json` as required and run `npm publish`.

## Supported Platforms

- iOS 8 and iOS 9
- Requires Cordova 6 running iOS Platform 4.1.1

## Methods
The methods available all return promises, or accept success and error callbacks.
- ApplePay.canMakePayments
- ApplePay.makePaymentRequest
- ApplePay.completeLastTransaction

## ApplePay.makePaymentRequest
Request a payment with Apple Pay.

```
ApplePay.makePaymentRequest(order, successCallback, errorCallback);
```

## ApplePay.canMakePayments
Detects if the current iDevice supports Apple Pay and has any capable cards registered.

If the `errorCallback` is called, the device does not support Apple Pay. See the message to see why.
If the `successCallback` is called, you're good to go ahead and display the related UI.

```
ApplePay.canMakePayments()
    .then(() => {
        // Apple Pay is enabled and a supported card is setup.
    })
    .catch((message) => {
        // The device is too old for Apple Pay
        // It's a good device, but no *supported* cards have been registered.
    });
```

## ApplePay.completeLastTransaction
Once the `successCallback` of the `makePaymentRequest` has been called, the device will be waiting for a completion event.
This means, that the application must proceed with the token authorisation and return a success, failure, or other validation error. Once this has been passed back, the Apple Pay sheet will be dismissed via an animation.

### Example

The order request object closely follows the format of the `PKPaymentRequest` class and thus its documentation will make excellent reading.

```
ApplePay.makePaymentRequest(
    {
          items: [
              {
                  label: '3 x Basket Items',
                  amount: 49.99
              },
              {
                  label: 'Next Day Delivery',
                  amount: 3.99
              },
                      {
                  label: 'My Fashion Company',
                  amount: 53.98
              }
          ],
          shippingMethods: [
              {
                  identifier: 'NextDay',
                  label: 'NextDay',
                  detail: 'Arrives tomorrow by 5pm.',
                  amount: 3.99
              },
              {
                  identifier: 'Standard',
                  label: 'Standard',
                  detail: 'Arrive by Friday.',
                  amount: 4.99
              },
              {
                  identifier: 'SaturdayDelivery',
                  label: 'Saturday',
                  detail: 'Arrive by 5pm this Saturday.',
                  amount: 6.99
              }
          ],
          merchantIdentifier: 'merchant.apple.test',
          currencyCode: 'GBP',
          countryCode: 'GB'
          billingAddressRequirement: 'none',
          shippingAddressRequirement: 'none',
          shippingType: 'shipping'
    })
    .then((token) => {
        // The user has authorized the payment.

        // Handle the token, asynchronously, i.e. pass to your merchant bank to action the payment, then once finished, depending on the outcome:

        // Displays the 'done' green tick and closes the sheet.
        ApplePay.completeLastTransaction('success');

    })
    .catch((e) => {
        // Failed to open the Apple Pay sheet, or the user cancelled the payment.
    })
```

Valid values for the `shippingType` are:

 * `shipping` (default)
 * `delivery`
 * `store`
 * `service`

Valid values for the `billingAddressRequirement` and `shippingAddressRequirement` properties are:

 * `none` (default)
 * `all`
 * `postcode`
 * `name`
 * `email`
 * `phone`

### Completing The Transaction
```
ApplePay.completeLastTransaction('success');
```

Once you have obtained the authorization with the provided token, you can dismiss or invalidate the Apple Pay sheet by calling `completeLastTransaction` with a status string which can be `success`, `failure`, `invalid-billing-address`, `invalid-shipping-address`, `invalid-shipping-contact`, `require-pin`, `incorrect-pin`, `locked-pin`.
