## CarbonTimeZone

Starting with Carbon 2, timezones are now handled with a dedicated class `CarbonTimeZone` extending [DateTimeZone](http://www.php.net/manual/en/class.datetimezone.php).
```php
$tz = new CarbonTimeZone('Europe/Zurich'); // instance way
$tz = CarbonTimeZone::create('Europe/Zurich'); // static way

// Get the original name of the timezone (can be region name or offset string):
echo $tz->getName();                 // Europe/Zurich
echo "\n";
// Casting a CarbonTimeZone to string will automatically call getName:
echo $tz;                            // Europe/Zurich
echo "\n";
echo $tz->getAbbreviatedName();      // cet
echo "\n";
// With DST on:
echo $tz->getAbbreviatedName(true);  // cest
echo "\n";
// Alias of getAbbreviatedName:
echo $tz->getAbbr();                 // cet
echo "\n";
echo $tz->getAbbr(true);             // cest
echo "\n";
// toRegionName returns the first matching region or false, if timezone was created with a region name,
// it will simply return this initial value.
echo $tz->toRegionName();            // Europe/Zurich
echo "\n";
// toOffsetName will give the current offset string for this timezone:
echo $tz->toOffsetName();            // +01:00
echo "\n";
// As with DST, this offset can change depending on the date, you may pass a date argument to specify it:
$winter = Carbon::parse('2018-01-01');
echo $tz->toOffsetName($winter);     // +01:00
echo "\n";
$summer = Carbon::parse('2018-07-01');
echo $tz->toOffsetName($summer);     // +02:00

// With no parameters, a default timezone is created:
echo new CarbonTimeZone();           // UTC
echo "\n";
echo CarbonTimeZone::create();       // UTC
```

The default timezone is given by [date_default_timezone_get](http://php.net/manual/en/function.date-default-timezone-get.php) so it will be driven by the INI settings [date.timezone](http://php.net/manual/en/datetime.configuration.php#ini.date.timezone) but you really should override it at application level using [date_default_timezone_set](http://php.net/manual/en/function.date-default-timezone-set.php) and you should set it to `"UTC"`, if you're temped to or already use an other timezone as default, please read the following article: [Always Use UTC Dates And Times](https://medium.com/@kylekatarnls/always-use-utc-dates-and-times-8a8200ca3164).

It explains why UTC is a reliable standard. And this best-practice is even more important in PHP because the PHP DateTime API has many bugs with offsets changes and DST timezones. Some of them appeared on minor versions and even on patch versions (so you can get different results running the same code on PHP 7.1.7 and 7.1.8 for example) and some bugs are not even fixed yet. So we highly recommend to use UTC everywhere and only change the timezone when you want to display a date. See our [first macro example](https://carbon.nesbot.com/docs/#user-settings).

While, region timezone ("Continent/City") can have DST and so have variable offset during the year, offset timezone have constant fixed offset:

```php
$tz = CarbonTimeZone::create('+03:00'); // full string
$tz = CarbonTimeZone::create(3); // or hour integer short way

echo $tz->getName();                 // +03:00
echo "\n";
echo $tz;                            // +03:00
echo "\n";
// toRegionName will try to guess what region it could be:
echo $tz->toRegionName();            // Europe/Helsinki
echo "\n";
// to guess with DST off:
echo $tz->toRegionName(null, 0);     // Europe/Moscow
echo "\n";
// toOffsetName will give the initial offset no matter the date:
echo $tz->toOffsetName();            // +03:00
echo "\n";
echo $tz->toOffsetName($winter);     // +03:00
echo "\n";
echo $tz->toOffsetName($summer);     // +03:00
```
You also can convert region timezones to offset timezones and reciprocally.
```php
$tz = new CarbonTimeZone(7);

echo $tz;                            // +07:00
echo "\n";
$tz = $tz->toRegionTimeZone();
echo $tz;                            // Asia/Novosibirsk
echo "\n";
$tz = $tz->toOffsetTimeZone();
echo $tz;                            // +07:00
```
You can create a `CarbonTimeZone` from mixed values using `instance()` method.
```php
$tz = CarbonTimeZone::instance(new DateTimeZone('Europe/Paris'));

echo $tz;                            // Europe/Paris
echo "\n";

// Bad timezone will return false without strict mode
Carbon::useStrictMode(false);
$tz = CarbonTimeZone::instance('Europe/Chicago');
var_dump($tz);                       // bool(false)
echo "\n";

// or throw an exception using strict mode
Carbon::useStrictMode(true);
try {
    CarbonTimeZone::instance('Europe/Chicago');
} catch (InvalidArgumentException $exception) {
    $error = $exception->getMessage();
}
echo $error;                         // Unknown or bad timezone (Europe/Chicago)

// as some value cannot be dump as string in an error message or
// have unclear dump, you may pass a second argument to display
// instead in the errors
Carbon::useStrictMode(true);
try {
    $mixedValue = ['dummy', 'array'];
    CarbonTimeZone::instance($mixedValue, json_encode($mixedValue));
} catch (InvalidArgumentException $exception) {
    $error = $exception->getMessage();
}
echo $error;                         // Unknown or bad timezone (["dummy","array"])
```



