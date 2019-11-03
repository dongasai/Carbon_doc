## IsSet

The PHP function `__isset()` is implemented. This was done as some external systems (ex. [Twig](http://twig.sensiolabs.org/doc/recipes.html#using-dynamic-object-properties)) validate the existence of a property before using it. This is done using the `isset()` or `empty()` method. You can read more about these on the PHP site: [__isset()](http://www.php.net/manual/en/language.oop5.overloading.php#object.isset), [isset()](http://www.php.net/manual/en/function.isset.php), [empty()](http://www.php.net/manual/en/function.empty.php).

```php
var_dump(isset(Carbon::now()->iDoNotExist));       // bool(false)
var_dump(isset(Carbon::now()->hour));              // bool(true)
var_dump(empty(Carbon::now()->iDoNotExist));       // bool(true)
var_dump(empty(Carbon::now()->year));              // bool(false)
```
