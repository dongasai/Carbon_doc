## Conversion

```php
$dt = Carbon::createFromFormat('Y-m-d H:i:s.u', '2019-02-01 03:45:27.612584');

var_dump($dt->toArray());
/*
array(12) {
  ["year"]=>
  int(2019)
  ["month"]=>
  int(2)
  ["day"]=>
  int(1)
  ["dayOfWeek"]=>
  int(5)
  ["dayOfYear"]=>
  int(32)
  ["hour"]=>
  int(3)
  ["minute"]=>
  int(45)
  ["second"]=>
  int(27)
  ["micro"]=>
  int(612584)
  ["timestamp"]=>
  int(1548992727)
  ["formatted"]=>
  string(19) "2019-02-01 03:45:27"
  ["timezone"]=>
  object(Carbon\CarbonTimeZone)#3438 (2) {
    ["timezone_type"]=>
    int(3)
    ["timezone"]=>
    string(3) "UTC"
  }
}
*/

var_dump($dt->toObject());
/*
object(stdClass)#3438 (12) {
  ["year"]=>
  int(2019)
  ["month"]=>
  int(2)
  ["day"]=>
  int(1)
  ["dayOfWeek"]=>
  int(5)
  ["dayOfYear"]=>
  int(32)
  ["hour"]=>
  int(3)
  ["minute"]=>
  int(45)
  ["second"]=>
  int(27)
  ["micro"]=>
  int(612584)
  ["timestamp"]=>
  int(1548992727)
  ["formatted"]=>
  string(19) "2019-02-01 03:45:27"
  ["timezone"]=>
  object(Carbon\CarbonTimeZone)#3444 (2) {
    ["timezone_type"]=>
    int(3)
    ["timezone"]=>
    string(3) "UTC"
  }
}
*/

var_dump($dt->toDate()); // Same as $dt->toDateTime()
/*
object(DateTime)#3438 (3) {
  ["date"]=>
  string(26) "2019-02-01 03:45:27.612584"
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(3) "UTC"
}
*/

// Note than both Carbon and CarbonImmutable can be cast
// to both DateTime and DateTimeImmutable
var_dump($dt->toDateTimeImmutable());
/*
object(DateTimeImmutable)#3438 (3) {
  ["date"]=>
  string(26) "2019-02-01 03:45:27.612584"
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(3) "UTC"
}
*/

class MySubClass extends Carbon {}
// MySubClass can be any class implementing CarbonInterface or a public static instance() method.

$copy = $dt->cast(MySubClass::class);
// Since 2.23.0, cast() can also take as argument any class that extend DateTime or DateTimeImmutable

echo get_class($copy).': '.$copy; // Same as MySubClass::instance($dt)
/*
MySubClass: 2019-02-01 03:45:27
*/
```
You can use the method `carbonize` to transform many things into a `Carbon` instance based on a given source instance used as reference on need. It returns a new instance.

```php
$dt = Carbon::createFromFormat('Y-m-d H:i:s.u', '2019-02-01 03:45:27.612584', 'Europe/Paris');

// Can take a date string and will apply the timezone from reference object
var_dump($dt->carbonize('2019-03-21'));
/*
object(Carbon\Carbon)#3444 (3) {
  ["date"]=>
  string(26) "2019-03-21 00:00:00.000000"
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(12) "Europe/Paris"
}
*/

// If you pass a DatePeriod or CarbonPeriod, it will copy the period start
var_dump($dt->carbonize(CarbonPeriod::create('2019-12-10', '2020-01-05')));
/*
object(Carbon\Carbon)#3439 (3) {
  ["date"]=>
  string(26) "2019-12-10 00:00:00.000000"
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(3) "UTC"
}
*/

// If you pass a DateInterval or CarbonInterval, it will add the interval to
// the reference object
var_dump($dt->carbonize(CarbonInterval::days(3)));
/*
object(Carbon\Carbon)#3444 (3) {
  ["date"]=>
  string(26) "2019-02-04 03:45:27.612584"
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(12) "Europe/Paris"
}
*/
```