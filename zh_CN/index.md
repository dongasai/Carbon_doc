##  介绍

The Carbon class is [inherited](http://php.net/manual/en/keyword.extends.php) from the PHP [DateTime](http://www.php.net/manual/en/class.datetime.php) class.

```php
<?php

namespace Carbon;

class Carbon extends \DateTime
{
    // code here
}

```

从上面的代码片段可以看出，Carbon类是在Carbon名称空间中声明的。您需要导入名称空间来使用Carbon，而不必每次都提供它的完全名称。

```php
use Carbon\Carbon;
```

本文档中的示例将假设您以这种方式导入了Carbon名称空间的类。

我们还提供了CarbonImmutable类 它继承了[DateTimeImmutable](http://www.php.net/manual/en/class.datetimeimmutable.php)。 两种类都可以使用相同的方法，但是在Carbon实例上使用修饰符时，它会修改并返回相同的实例，在CarbonImmutable上使用它时，它将返回具有新值的新实例。

```php
$mutable = Carbon::now();
$immutable = CarbonImmutable::now();
$modifiedMutable = $mutable->add(1, 'day');
$modifiedImmutable = CarbonImmutable::now()->add(1, 'day');

var_dump($modifiedMutable === $mutable);             // bool(true)
var_dump($mutable->isoFormat('dddd D'));             // string(10) "Tuesday 15"
var_dump($modifiedMutable->isoFormat('dddd D'));     // string(10) "Tuesday 15"
// So it means $mutable and $modifiedMutable are the same object
// both set to now + 1 day.
var_dump($modifiedImmutable === $immutable);         // bool(false)
var_dump($immutable->isoFormat('dddd D'));           // string(9) "Monday 14"
var_dump($modifiedImmutable->isoFormat('dddd D'));   // string(10) "Tuesday 15"
// While $immutable is still set to now and cannot be changed and
// $modifiedImmutable is a new instance created from $immutable
// set to now + 1 day.

$mutable = CarbonImmutable::now()->toMutable();
var_dump($mutable->isMutable());                     // bool(true)
var_dump($mutable->isImmutable());                   // bool(false)
$immutable = Carbon::now()->toImmutable();
var_dump($immutable->isMutable());                   // bool(false)
var_dump($immutable->isImmutable());                 // bool(true)
```

该库还提供了CarbonInterface接口，继承了[DateTimeImmutable](http://www.php.net/manual/en/class.datetimeimmutable.php)和[JsonSerializable](http://www.php.net/manual/en/class.jsonserializable.php)，[CarbonInterval](https://carbon.nesbot.com/docs/#api-interval)类继承了[DateInterval](http://www.php.net/manual/en/class.dateinterval.php), [CarbonTimeZone](https://carbon.nesbot.com/docs/#api-timezone)类继承了[DateTimeZone](http://www.php.net/manual/en/class.datetimezone.php) 和[CarbonPeriod](https://carbon.nesbot.com/docs/#api-period) 类polyfills [DatePeriod](http://www.php.net/manual/en/class.dateperiod.php)。

Carbon拥有从基本datetime类继承的所有函数。此方法允许您访问基本功能，如[modify](http://php.net/manual/en/datetime.modify.php)、[format](http://php.net/manual/en/datetime.format.php)或[diff](http://php.net/manual/en/datetime.diff.php)。

## 实例化
---
有几种不同的方法可用于创建Carbon的新实例。 首先有一个构造函数。 它会覆盖[parent constructor](http://www.php.net/manual/en/datetime.construct.php)，最好从PHP手册中了解第一个参数，并了解 date/time 字符串格式 它接受。 希望您会发现自己很少使用构造函数，而是依靠显式的静态方法来提高可读性。

```php
$carbon = new Carbon();                  // equivalent to Carbon::now()
$carbon = new Carbon('first day of January 2008', 'America/Vancouver');
echo get_class($carbon);                 // 'Carbon\Carbon'
```

您会在上面注意到 timezone (第2个参数)参数是作为字符串而不是** \DateTimeZone** 实例传递的。 所有DateTimeZone参数均已增强，因此您可以将DateTimeZone实例，字符串或整数偏移量传递给GMT，然后将为您创建时区。 在下一个示例中再次显示了该示例，该示例还引入了** now()**函数。

```php
$now = Carbon::now(); // will use timezone as set with date_default_timezone_set
// PS: we recommend you to work with UTC as default timezone and only use
// other timezones (such as the user timezone) on display

$nowInLondonTz = Carbon::now(new DateTimeZone('Europe/London'));

// or just pass the timezone as a string
$nowInLondonTz = Carbon::now('Europe/London');
echo $nowInLondonTz->tzName;             // Europe/London
echo "\n";

// or to create a date with a custom fixed timezone offset
$date = Carbon::now('+13:30');
echo $date->tzName;                      // +13:30
echo "\n";

// Get/set minutes offset from UTC
echo $date->utcOffset();                 // 810
echo "\n";

$date->utcOffset(180);

echo $date->tzName;                      // +03:00
echo "\n";
echo $date->utcOffset();                 // 180
```

如果你真的很喜欢你的连续方法调用，并且在使用构造器时被额外的一行或难看的一对括号搞砸了，你会喜欢**parse**方法。

```php
echo (new Carbon('first day of December 2008'))->addWeeks(2);     // 2008-12-15 00:00:00
echo "\n";
echo Carbon::parse('first day of December 2008')->addWeeks(2);    // 2008-12-15 00:00:00
```

传递给 **Carbon::parse** 或 **new Carbon** 的字符串可以表示相对时间（下一个星期日，明天，下个月的第一天，去年）或绝对时间(2008年12月的第一天,2017-01-06)。 您可以使用**Carbon::hasRelativeKeywords()** 测试字符串是产生相对还是绝对日期。

```php
$string = 'first day of next month';
if (strtotime($string) === false) {
    echo "'$string' is not a valid date/time string.";
} elseif (Carbon::hasRelativeKeywords($string)) {
    echo "'$string' is a relative valid date/time string, it will returns different dates depending on the current date.";
} else {
    echo "'$string' is an absolute date/time string, it will always returns the same date.";
}
```
