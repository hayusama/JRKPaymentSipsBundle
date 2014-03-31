Getting started with JRKPaymentSipsBundle
======================================

Setup
-----
JRKPaymentSipsBundle requires the ATOS api folder


- Using the vendors script

Add jrk/paymentsips-bundle as a dependency in your project's composer.json file:

```
{
    "require": {
        "jrk/paymentsips-bundle": "dev-master"
    }
}
```

Or add to your deps

```
[JRKPaymentSipsBundle]
    git=git://github.com/jreziga/JRKPaymentSipsBundle.git
    target=bundles/JRK/PaymentSipsBundle
```

... and run php bin/vendors install

... and add the JRK namespace to autoloader

``` php
<?php
   // app/autoload.php
   $loader->registerNamespaces(array(
    // ...
    'JRK' => __DIR__.'/../vendor/bundles',
  ));
```

- Add JRKPaymentSipsBundle to your application kernel

``` php
<?php
// app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        // ...
        new JRK\PaymentSipsBundle\JRKPaymentSipsBundle(),
    );
}
```


- Yml configuration

``` yml
# app/config/config.yml
jrk_payment_sips:
    files:
        sips_pathfile: "%kernel.root_dir%/config/sips/param/pathfile"
        sips_request: "%kernel.root_dir%/config/sips/bin/static/request"
        sips_response: "%kernel.root_dir%/config/sips/bin/static/response"
        sips_logs: "%kernel.root_dir%/logs/sips.log"
    params:
        sips_merchant_id: "XXXXXXXXXXXXXXXXXX"
        sips_currency_code: "EUR"   # OR use the currency_code provided by ATOS (978=EUR for example)
        sips_language: "fr"
        sips_payment_means: "CB,2,VISA,2,MASTERCARD,2"
        sips_header_flag: "yes"
        sips_merchant_country: "fr"
    links:
        sips_cancel_return_url: "my_homepage_route"     # Route to redirect if the payment is canceled
        sips_route_response: "my_sips_response"         # Route to redirect if the payment is accepted
```

- Routes import

``` yml
# app/config/routing.yml
jrk_payment_sips:
    resource: "@JRKPaymentSipsBundle/Resources/config/routing.yml"
    prefix: /payment
```

- Console usage 
1. install assets
``` 
php app/console assets:install
```
2. specify param's path directory (by default use [app/config/sips/param])
``` 
php app/console jrk:sips:install
```

- For example, with default values of the bundle, you can extract the API like this : 
    .
    |-- app
    |   `-- config
    |       `-- sips
    |           `-- bin
    |               `-- static
    |                   `-- request
    |                   `-- response
    |           `-- param
    |               `-- certif.XXXXXXXXXXXX
    |               `-- parmcom.XXXXXXXXXXXX
    |               `-- parmcom.mercanet        # if you are using mercanet for example
    |               `-- pathfile (generated)
    |           `-- Version.txt



Usage
-----


 - Using service

Open your controller and call the service.

``` php
<?php
    $sips_form =  $this->get('jrk_paymentsips')->get_sips_request(array("amount"=>10),MyTransactionEntity);
?>
```

Then you can use this method in your "sips_route_response" controller

``` php
<?php
    $order = $this->get('jrk_paymentsips')->sips_load_entity();
    
    // Store the validated order in database for example
    $em = $this->getEntityManager();
    $em->persist($item);
    $em->flush();
?>
```

Controller example

``` php
<?php
    class MyController
    {

        public function paymentpageAction()
        {
    
            // Initialize your order entity or whatever you want
            $order = new OrderExample();
            
         
            // Don't forget to set an amount in array
            // You can dynamically override config parameters here like currency_code etc...
            $sips_form = $this->get('jrk_paymentsips')->get_sips_request(array("amount"=>$order->getAmount()),$order);
    
    
            // Render your payment page, you can render the sips form like that for twig : {{ sips_form }}
            return $this->render('ShopFrontBundle:MyController:paymentpage.html.twig',array("sips_form"=>$sips_form));
    
        }
    
    
        public function my_sips_responseAction()
        {
            $order = $this->get('jrk_paymentsips')->sips_load_entity();
            
            // Store your transaction entity in database for example, or attributes.
            $order->setState("ACCEPTED");
            $em = $this->getEntityManager();
            $em->persist($order);
            $em->flush();
            
            // Notify the user by mail for example
            /* ... */
            
            // Redirect the user in his history orders for example
            return $this->redirect($this->generateUrl("user_history_orders"));
        }
    }
?>
```
