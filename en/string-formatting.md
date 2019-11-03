## String Formatting

All of the available `toXXXString()` methods rely on the base class method [DateTime::format()](http://php.net/manual/en/datetime.format.php). You'll notice the `__toString()` method is defined which allows a Carbon instance to be printed as a pretty date time string when used in a string context.

```php
$dt = Carbon::create(1975, 12, 25, 14, 15, 16);

var_dump($dt->toDateTimeString() == $dt);          // bool(true) => uses __toString()
echo $dt->toDateString();                          // 1975-12-25
echo $dt->toFormattedDateString();                 // Dec 25, 1975
echo $dt->toTimeString();                          // 14:15:16
echo $dt->toDateTimeString();                      // 1975-12-25 14:15:16
echo $dt->toDayDateTimeString();                   // Thu, Dec 25, 1975 2:15 PM

// ... of course format() is still available
echo $dt->format('l jS \\of F Y h:i:s A');         // Thursday 25th of December 1975 02:15:16 PM

// The reverse hasFormat method allows you to test if a string looks like a given format
var_dump($dt->hasFormat('Thursday 25th December 1975 02:15:16 PM', 'l jS F Y h:i:s A')); // bool(true)
```

You can also set the default __toString() format (which defaults to `Y-m-d H:i:s`) thats used when type [juggling occurs](http://php.net/manual/en/language.types.type-juggling.php).

```php
echo $dt;                                          // 1975-12-25 14:15:16
echo "\n";
$dt->settings([
    'toStringFormat' => 'jS \o\f F, Y g:i:s a',
]);
echo $dt;                                          // 25th of December, 1975 2:15:16 pm

// As any setting, you can get the current value for a given date using:
var_dump($dt->getSettings());
/*
array(3) {
  ["toStringFormat"]=>
  string(20) "jS \o\f F, Y g:i:s a"
  ["locale"]=>
  string(2) "en"
  ["timezone"]=>
  string(3) "UTC"
}*/
```

As part of the settings `'toStringFormat'` can be used in factories too. It also may be a closure, so you can run any code on string cast.

If you use Carbon 1 or want to apply it globally as default format, you can use:

```php
$dt = Carbon::create(1975, 12, 25, 14, 15, 16);
Carbon::setToStringFormat('jS \o\f F, Y g:i:s a');
echo $dt;                                          // 25th of December, 1975 2:15:16 pm
echo "\n";
Carbon::resetToStringFormat();
echo $dt;                                          // 1975-12-25 14:15:16
```
Note: For localization support see the [Localization](https://carbon.nesbot.com/docs/#api-localization) section.
