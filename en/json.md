## Json
The Carbon instances can be encoded to and decoded from JSON. Since the version 2.4, we explicitly require the PHP JSON extension. It should have no impact as this extension is bundled by default with PHP. If the extension is disabled, be aware you will be locked on 2.3. But you still can use `--ignore-platform-reqs` on composer update/install to upgrade then polyfill the missing `JsonSerializable` interface by including [JsonSerializable.php](https://github.com/briannesbitt/Carbon/blob/version-2.3/src/JsonSerializable.php).

```php
$dt = Carbon::create(2012, 12, 25, 20, 30, 00, 'Europe/Moscow');
echo json_encode($dt);
// "2012-12-25T16:30:00.000000Z"

$json = '{"date":"2012-12-25 20:30:00.000000","timezone_type":3,"timezone":"Europe\/Moscow"}';
$dt = Carbon::__set_state(json_decode($json, true));
echo $dt->format('Y-m-d\TH:i:s.uP T');
// 2012-12-25T20:30:00.000000+04:00 MSK
```

You can use `settings(['toJsonFormat' => $format])` to customize the serialization.

```php
$dt = Carbon::create(2012, 12, 25, 20, 30, 00, 'Europe/Moscow')->settings([
    'toJsonFormat' => function ($date) {
        return $date->getTimestamp();
    },
]);
echo json_encode($dt); // 1356453000
```
If you wan to apply this globally, first consider using factory, else or i you use Carbon 1 you can use:

```php
$dt = Carbon::create(2012, 12, 25, 20, 30, 00, 'Europe/Moscow');
Carbon::serializeUsing(function ($date) {
    return $date->valueOf();
});
echo json_encode($dt); // 1356453000000

// Call serializeUsing with null to reset the serializer:
Carbon::serializeUsing(null);
```

The `jsonSerialize()` method allow you to call the function given to `Carbon::serializeUsing()` or the result of `toJson()` if no custom serialization specified.

