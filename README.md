# Konekt Enum


[![Build Status](https://travis-ci.org/artkonekt/enum.png?branch=master)](https://travis-ci.org/artkonekt/enum)
[![Latest Stable Version](https://poser.pugx.org/konekt/enum/version.png)](https://packagist.org/packages/konekt/enum)
[![Total Downloads](https://poser.pugx.org/konekt/enum/downloads.png)](https://packagist.org/packages/konekt/enum)
[![Open Source Love](https://badges.frapsoft.com/os/mit/mit.svg?v=102)](https://github.com/ellerbrock/open-source-badge/)

## PHP Enum Class

### Enum

Abstract class that enables creation of PHP enums. All you have to do is extend this class and define some constants.
Alternative implementation of the SplEnum class (as that is often not available on php installations)

Enum is an object with value from one of those constants (or from one of superclass if any).
There is also a __default constant that enables you to create an object without passing enum value.

[Changelog](Changelog.md)

### Installation

Install it via composer:

```
composer require konekt/enum
```

### How To Use


```php
class Status extends \Konekt\Enum\Enum
{
    const __default      = self::PLACED;

    const PLACED         = 'placed';
    const CONFIRMED      = 'confirmed';
    const PROCESSING     = 'processing';
    const COMPLETED      = 'completed';
}
```

**Creating Instances:**

```php
// With plain constructor:
$placed = new Status('placed');
// or
$placed = new Status(Status::PLACED);

// With factory method:
$confirmed  = Status::create(Status::CONFIRMED);
$processing = Status::create('processing');

// With magic constructor (use const name as method):
$completed = Status::COMPLETED();

// Setting a __default:
$placed = new Status();
echo $placed->value();
// outputs: 'placed'

```

Instances are immutable.

Other classes can use it as a standalone type, internally mapping to whatever the original type is:

```php
namespace App\Order;

class Order
{
    /* ... the order entity ... */
    
    /** @var  string */
    protected $status;
    
    /**
     * @return Status
     */
    protected function getStatus()
    {
        return new Status($this->status); 
    }
    
    public function setStatus(Status $status)
    {
        $this->status = $status->value();    
    }

}

```

From the client code perspective


```php

use App\Order\Status;
use App\Order\Order;

class OrderController
{
    
    public function createAction()
    {
        $order = new Order();
        $order->setStatus(Status::PLACED());
        $order->save();
    }
    
    public function confirmAction($order)
    {
        $order->setStatus(Status::CONFIRMED());
        $order->save();
    }
    
    public function processAction($order)
    {
        //Alternatively you can directly use the constructor as well:
        $status = new Status(Status::PROCESSING);
        $order->setStatus($status);
        $order->save();
    }

}
```

#### Labels

You can optionally set labels (display texts) for enum values.
You can define them via a static array `$labels` on your concrete enum class.

If it's necessary to set the labels via runtime a function (eg. translate with `__()`) then it can be done within a custom `boot()` method.

##### Via Static Array

This is protected static array containing enum values as keys and user friendly texts a values:

```php
class OrderStatus extends \Konekt\Enum\Enum
{
    const __default      = self::PLACED;

    const PLACED         = 'placed';
    const CONFIRMED      = 'confirmed';
    const PROCESSING     = 'processing';
    const COMPLETED      = 'completed';
    
    /** @var array Labels are optional */
    protected static $labels = [
        self::PLACED     => 'Placed',
        self::CONFIRMED  => 'Confirmed',
        self::PROCESSING => 'Processing',
        self::COMPLETED  => 'Completed'    
    ];
}
```

##### Within The Boot Method

This can be useful if the label text needs to be set during runtime.

*Example: returning translated display text with gettext:*

```php
final class OffsitePaymentMethod extends \Konekt\Enum\Enum
{
    const __default = self::WIRE_TRANSFER;

    const WIRE_TRANSFER     = 'wire_transfer';
    const CASH              = 'cash';
    const POS               = 'pos';
    const CASH_ON_DELIVERY  = 'cash_on_delivery';
    
    // $labels static property needs to be defined
    public static $labels = [];
    
    protected static function boot()
        {
            static::$labels = [
                self::WIRE_TRANSFER     => __('Wire Transfer'),
                self::CASH              => __('Cash'),
                self::POS               => __('Pos terminal'),
                self::CASH_ON_DELIVERY  => __('Cash on delivery')
            ];
        }
}
```

You can use two methods for utilizing display texts `choices()` _(static)_ and `label()` for an object instance.

The `label()` method returns the label for a specific instance value. If no label was set via the `$label` property, defaults to the enum value:

```php
final class BarType extends \Konekt\Enum\Enum
{
    const CREATED        = 'created';
    const ACTIVE         = 'active';
    const CLOSED         = 'closed';
    
    protected static $labels = [
        self::CREATED    => 'New Bar',
        self::ACTIVE     => 'Ongoing Bar'  
    ];
}

$created = BarType::CREATED();
echo $created->label();
//outputs: 'New Bar'

$active = BarType::ACTIVE();
echo $active->label();
//outputs: 'Ongoing Bar'

$closed = BarType::CLOSED();
echo $closed->label();
//outputs: 'closed' -> fallback to enum value since no label was set
```

The `choices()` method returns all the available choices along with their display texts. Useful for selects/dropdowns:

```php
final class BarType extends Enum
{
    const CREATED        = 'created';
    const ACTIVE         = 'active';
    const CLOSED         = 'closed';
    
    protected static $labels = [
        self::CREATED    => 'New Bar',
        self::ACTIVE     => 'Ongoing Bar'  
    ];
}

print_r(BarType::choices());
//Outputs:
//Array
//(
//    [created] => 'New Bar'
//    [active]  => 'Ongoing Bar'
//    [closed]  => 'closed'
//)

```

#### Method Examples

```php

use App\Order\Status;

//Using the shortcut/magic factory:
$completed = Status::COMPLETED();

echo $completed;
//outputs: 'Completed'

echo $completed->value();
//outputs: 'completed'



//Using directly the constructor:
$confirmed = new Status(Status::CONFIRMED);

echo $confirmed;
//output: 'Confirmed'

echo $confirmed->value();
//output: 'confirmed'



//creating with no explicit initial value
$status = new Status();

echo $status;
//output: 'Placed'
//due to having a __default value set and magic __toString() method


echo $status->value();
//output: 'placed'

print_r($status->toArray());
//output:
//Array
//  (
//      [PLACED] => placed
//      [CONFIRMED] => confirmed
//      [PROCESSING] => processing
//      [COMPLETED] => completed
//  )

print_r(Status::toArray());
//output:
//Array
//  (
//      [PLACED] => placed
//      [CONFIRMED] => confirmed
//      [PROCESSING] => processing
//      [COMPLETED] => completed
//  )

print_r(Status::choices());
//Array
//(
//    [placed]     => 'Placed'
//    [confirmed]  => 'Confirmed'
//    [processing] => 'Processing'
//    [completed]  => 'Completed'
//)

echo Status::hasConst('PLACED') ? 'yes' : 'no';
//output: 'yes'

echo Status::hasConst('placed') ? 'yes' : 'no';
//output: 'no'

echo Status::has('PLACED') ? 'yes' : 'no';
//output: 'no'

echo Status::has('placed') ? 'yes' : 'no';
//output: 'yes'

//Using the equals() comparison

$placed = Status::PLACED();

echo $placed->equals($completed) ? 'yes' : 'no';
//output: 'no'

echo $placed->equals($status) ? 'yes' : 'no';
//output: 'no'

$placed2 = new Status(Status::PLACED);
echo $placed->equals($placed2) ? 'yes' : 'no';
//output: 'yes'

```
