## Setters 

The following setters are implemented via PHP's `__set()` method. Its good to take note here that none of the setters, with the obvious exception of explicitly setting the timezone, will change the timezone of the instance. Specifically, setting the timestamp will not set the corresponding timezone to UTC.

```php
$dt = Carbon::now();

$dt->year = 1975;
$dt->month = 13;             // would force year++ and month = 1
$dt->month = 5;
$dt->day = 21;
$dt->hour = 22;
$dt->minute = 32;
$dt->second = 5;

$dt->timestamp = 169957925;  // This will not change the timezone
// Same as:
$dt->setTimestamp(169957925);
$dt->timestamp(169957925);

// Set the timezone via DateTimeZone instance or string
$dt->timezone = new DateTimeZone('Europe/London');
$dt->timezone = 'Europe/London';
$dt->tz = 'Europe/London';

// verbose way:
$dt->setYear(2001);
echo $dt->year;      // 2001
echo "\n";

// set/get method:
$dt->year(2002);
echo $dt->year();    // 0000-05-22 03:32:05
echo "\n";

// dynamic way:
$dt->set('year', 2003);
echo $dt->get('year'); // 2003
echo "\n";

// these methods exist for every units even for calculated properties such as:
echo $dt->dayOfYear(35)->format('Y-m-d'); // 2003-02-04
```
