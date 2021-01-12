# FlutterWithStripeExample

This repository contains an example of how to integrate Stripe Connect \w 3DSecure (SCA) in your Flutter based app.

More informations are available [Here](https://itnext.io/flutter-stripe-connect-3dsecure-sca-ab8fd364ca1e).

## createPaymentIntent backend implementation

Your Flutter app will need to call Stripe's `paymentIntents.create()` API (see https://stripe.com/docs/api/payment_intents/create).
This needs to happen on the server side as it involves sending your secret key which you naturally do not wish to expose on the client side.

You can implement this in any language you choose, below are two examples:

### Using Firebase and Stripe's NPM module:

Deploy the below code with:
```sh
$ firebase deploy --only functions
```

To fetch the logs, use:
```sh
$ firebase functions:log --only  createPaymentIntent
```

```js
const functions = require('firebase-functions');
const admin = require('firebase-admin');
admin.initializeApp(functions.config().firebase);

const firestore = admin.firestore();
const settings = { timestampInSnapshots: true };
firestore.settings(settings)
// your secret here
const stripe = require('stripe')('SK_PRIVATE_SECRET');
exports.createPaymentIntent = functions.https.onRequest((req, res) => { 
    var data = req.body;
    functions.logger.log("Here's a request object:", data);

    return stripe.paymentIntents.create({
        amount: data.amount,
        currency: data.currency,
        payment_method_types: ['card'],
        receipt_email: data.email,
        payment_method: data.payment_method_id,
        confirm: true,
    },   
    {    
      stripeAccount: 'YOUR_ACCOUNT_ID_HERE',
     
    }, function(err, paymentIntent) {
                // asynchronously called
                const paymentIntentReference = paymentIntent;
                if (err !== null){
                    functions.logger.log("Error payment intent:", err);
                    res.send('error');
                } else {
                    console.log('Created paymentintent: ', paymentIntent);
                    functions.logger.log("paymentIntent object:", paymentIntent);
                    res.json({
                        clientSecret: paymentIntent.client_secret,
                    });
                }
       }
  );

});
```

### Using Python (tested with AWS' Lambda service)

Install the module:

```sh
# pip3 install stripe
```

```python
import sys
import logging
import json
import stripe
import os


def lambda_handler(data, context):
  email = data['email']
  payment_method_id = data['payment_method_id']
  
  stripe.api_key = 'sk_test_...' #Your test/live secret key
  
  payment_intent = stripe.PaymentIntent.create(
    payment_method_types=['card'],
    payment_method = payment_method_id,
    amount=1000,
    application_fee_amount=140,
    currency='eur',
    stripe_account='acct_1G...',#connected account ID
    receipt_email=email,
    confirm=True
  )
  
  return payment_intent.client_secret
```
