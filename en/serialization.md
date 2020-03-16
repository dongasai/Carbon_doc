# Serialization
The Carbon instances can be serialized (including CarbonImmutable, CarbonInterval and CarbonPeriod).
```php
$dt = Carbon::create(2012, 12, 25, 20, 30, 00, 'Europe/Moscow');

echo serialize($dt);                                              // O:13:"Carbon\Carbon":3:{s:4:"date";s:26:"2012-12-25 20:30:00.000000";s:13:"timezone_type";i:3;s:8:"timezone";s:13:"Europe/Moscow";}
// same as:
echo $dt->serialize();                                            // O:13:"Carbon\Carbon":3:{s:4:"date";s:26:"2012-12-25 20:30:00.000000";s:13:"timezone_type";i:3;s:8:"timezone";s:13:"Europe/Moscow";}

$dt = 'O:13:"Carbon\Carbon":3:{s:4:"date";s:26:"2012-12-25 20:30:00.000000";s:13:"timezone_type";i:3;s:8:"timezone";s:13:"Europe/Moscow";}';

echo unserialize($dt)->format('Y-m-d\TH:i:s.uP T');               // 2012-12-25T20:30:00.000000+04:00 MSK
// same as:
echo Carbon::fromSerialized($dt)->format('Y-m-d\TH:i:s.uP T');    // 2012-12-25T20:30:00.000000+04:00 MSK
```