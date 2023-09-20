# Advancelearn Otp-auth

<img src="https://banners.beyondco.de/advancelearn%2Fmanage-payment-and-orders.png?theme=dark&packageManager=composer+require&packageName=advancelearn%2Fmanage-payment-and-orders&pattern=stripes&style=style_1&description=Orders+and+payments+management+system+in+Laravel+and+the+feature+of+adding+sales+functionality+for+each+model&md=1&showWatermark=0&fontSize=100px&images=https%3A%2F%2Flaravel.com%2Fimg%2Flogomark.min.svg&widths=350" alt="advancelearn-otp-auth">


<a name="introduction"></a>

## Introduction

This package was developed in collaboration with a Sadratek team member and AdvanceLearn. This package aims to create
and manage various aspects related to user orders and payments. It includes tracking the number of products and tagging
all steps of the user's purchase order." This package is currently under active development and there are plans to add a
shopping voucher module in the future, pending support and feasibility.

<a name="installation"></a>

## Installation

You can install the package with Composer.

```bash
composer require advancelearn/manage-payment-and-orders
```

<a name="Config"></a>

## Config

After installation, please add its provider to your config folder in the app file to complete and configure the package:

```php
 \Advancelearn\ManagePaymentAndOrders\ManagePayOrderServiceProvider::class,
```

Then run this command to import and make the package tables public

```php
php artisan vendor:publish
```

Select the row number of this title from among the tags and enter it

```php
  Tag: AdvanceLearnManagePayAndOrder-migrations ...............
```

#### By entering the tag number of the image above, these tables will be added to the tables folder of your program

![img_1.png](img_1.png)

Then enter the following command to add tables in your database

```php 
php artisan migrate
```
##Create new Order By User Request
"In the initial step, it's important to note that we aim to store a user's order within the system. To achieve this, we need to send the required parameters to the 'store' method of the order service package. Upon successful validation of these parameters, we will receive a response containing the order details."
We receive and send the requested parameters from the user:
`$shippingId , $addressId , $description , $items`
**`__!!Pay attention to the type of parameters that they should be__`**
```php
app('orderFunction')->store(int $shippingId, int $addressId, string $description, array $items)
```

### add this method for Relationship in your model with Inventory

```php
    public function inventories()
    {
        return $this->morphMany(app('inventory'), 'inventories');
    }

```

and add nameaspace this file in top youre model :

"Now you have created a new order in the order creation phase, so to redirect the user to the payment gateway, you need to send this order id to this method, you can check whether the order amount has already been paid or not. In case of non-payment, you can redirect the user to the payment gateway."

```php
        $order = app('order')::findOrFail($orderId);

        if ($order->audits->where('id', app('auditTypes')::PAID)->count()) {
            return response()->json(['errors' => 'Already paid'], 422);
        }

```

And finally, send the final order amount to send to the portal:

```php
$invoice = new Invoice;

$invoice->amount($order->payment_price);
```

And in the same method that the user was sent to the payment portal, you can create a payment for the user with a
pending status:

```php
 app('payment')::create
 ([
    'order_id' => $order->id,
    'driver' => $request->input('driver'),
    'transaction_id' => //transactionId from your gateway sent request for pay,
    'amount' => $order->payment_price
]);
```

```php
use Advancelearn\ManagePaymentAndOrders\Models\Inventory;
```

#### next step

And in the last step, you can receive the desired payment in the return method so that the payment is confirmed and
confirmed Update it to confirm. First, add the following relationship to the relevant model of the product example


```php
    public function paymentConfirmation($user_id, $inventory_id, $quantity)
    {
        $inventory = $this->inventories()->find($inventory_id);
        $inventory->count -= $quantity;
        $inventory->save();
    }
```

And for example, in your callback method that returns from the portal, let's assume that your callback method is called
paymentConfirmation.

```php
    public function paymentConfirmation(Request $request)
    {
        $payment = app('payment')::where('transaction_id', $transactionId)->first();
        $order = app('order')::find($payment->order_id);
         $receipt = //api call gateway for verifyPayment and get response add to this variable
        $this->verifyPayAndConfirm($payment, $order, $request);
//          return redirect('https://example.com?gatewayOrderID=' .
//           $paymentTransaction->OrderId ?? null . '&RRN=' .
//           $paymentTransaction->RRN ?? null);
    }
```

create call method verifyPayAndConfirm::

```php
    private function verifyPayAndConfirm($receipt, $payment, Request $request): void
    {
        $payment->reference_id = $receipt->getReferenceId();
        $payment->transaction = $request->all();
        $payment->driver = $receipt->getDriver();
        $payment->save();
        $order->audits()->attach([app('auditTypes')::PAID => ['description' => 'Payment was successful']]);
        foreach ($order->items as $item) {
            $item->PaymentConfirmation($order->address->user_id);
        }
        return $receipt;
    }
```
###You can call this method to get all the information about your orders
```php
app('orderFunction')->getOrders();
```
###Use this method to display individual information of an order
```php
app('orderFunction')->show($order);
```
##Pay attention to the type of parameters
###You can use this method to update an order
```php
app('orderFunction')->update(int $shippingId, int $addressId, string $description, string $shippingDate, array $items, array $audits, int $orderId);
```
###You can use this method to cancel the order from the management side
```php
app('orderFunction')->destroy($order)
```


