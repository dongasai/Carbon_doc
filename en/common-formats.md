## Common Formats
The following are wrappers for the common formats provided in the DateTime class.
```php
$dt = Carbon::createFromFormat('Y-m-d H:i:s.u', '2019-02-01 03:45:27.612584');

// $dt->toAtomString() is the same as $dt->format(DateTime::ATOM);
echo $dt->toAtomString();           // 2019-02-01T03:45:27+00:00
echo $dt->toCookieString();         // Friday, 01-Feb-2019 03:45:27 UTC

echo $dt->toIso8601String();        // 2019-02-01T03:45:27+00:00
// Be aware we chose to use the full-extended format of the ISO 8601 norm
// Natively, DateTime::ISO8601 format is not compatible with ISO-8601 as it
// is explained here in the PHP documentation:
// https://php.net/manual/class.datetime.php#datetime.constants.iso8601
// We consider it as a PHP mistake and chose not to provide method for this
// format, but you still can use it this way:
echo $dt->format(DateTime::ISO8601); // 2019-02-01T03:45:27+0000

echo $dt->toISOString();            // 2019-02-01T03:45:27.612584Z
echo $dt->toJSON();                 // 2019-02-01T03:45:27.612584Z

echo $dt->toIso8601ZuluString();    // 2019-02-01T03:45:27Z
echo $dt->toDateTimeLocalString();  // 2019-02-01T03:45:27
echo $dt->toRfc822String();         // Fri, 01 Feb 19 03:45:27 +0000
echo $dt->toRfc850String();         // Friday, 01-Feb-19 03:45:27 UTC
echo $dt->toRfc1036String();        // Fri, 01 Feb 19 03:45:27 +0000
echo $dt->toRfc1123String();        // Fri, 01 Feb 2019 03:45:27 +0000
echo $dt->toRfc2822String();        // Fri, 01 Feb 2019 03:45:27 +0000
echo $dt->toRfc3339String();        // 2019-02-01T03:45:27+00:00
echo $dt->toRfc7231String();        // Fri, 01 Feb 2019 03:45:27 GMT
echo $dt->toRssString();            // Fri, 01 Feb 2019 03:45:27 +0000
echo $dt->toW3cString();            // 2019-02-01T03:45:27+00:00
```