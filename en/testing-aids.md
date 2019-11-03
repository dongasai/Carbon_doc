# Testing Aids

The testing methods allow you to set a Carbon instance (real or mock) to be returned when a "now" instance is created. The provided instance will be used when retrieving any relative time from Carbon (now, today, yesterday, next month, etc.)

```php
$knownDate = Carbon::create(2001, 5, 21, 12);          // create testing date
Carbon::setTestNow($knownDate);                        // set the mock (of course this could be a real mock object)
echo Carbon::getTestNow();                             // 2001-05-21 12:00:00
echo Carbon::now();                                    // 2001-05-21 12:00:00
echo new Carbon();                                     // 2001-05-21 12:00:00
echo Carbon::parse();                                  // 2001-05-21 12:00:00
echo new Carbon('now');                                // 2001-05-21 12:00:00
echo Carbon::parse('now');                             // 2001-05-21 12:00:00
echo Carbon::create(2001, 4, 21, 12)->diffForHumans(); // 1 month ago
var_dump(Carbon::hasTestNow());                        // bool(true)
Carbon::setTestNow();                                  // clear the mock
var_dump(Carbon::hasTestNow());                        // bool(false)
echo Carbon::now();                                    // 2019-10-14 14:49:08

```

A more meaning full example:

```php
class SeasonalProduct
{
    protected $price;

    public function __construct($price)
    {
        $this->price = $price;
    }

    public function getPrice() {
        $multiplier = 1;
        if (Carbon::now()->month == 12) {
            $multiplier = 2;
        }

        return $this->price * $multiplier;
    }
}

$product = new SeasonalProduct(100);
Carbon::setTestNow(Carbon::parse('first day of March 2000'));
echo $product->getPrice();                                             // 100
Carbon::setTestNow(Carbon::parse('first day of December 2000'));
echo $product->getPrice();                                             // 200
Carbon::setTestNow(Carbon::parse('first day of May 2000'));
echo $product->getPrice();                                             // 100
Carbon::setTestNow();
```

Relative phrases are also mocked according to the given "now" instance.

```php
$knownDate = Carbon::create(2001, 5, 21, 12);          // create testing date
Carbon::setTestNow($knownDate);                        // set the mock
echo new Carbon('tomorrow');                           // 2001-05-22 00:00:00  ... notice the time !
echo new Carbon('yesterday');                          // 2001-05-20 00:00:00
echo new Carbon('next wednesday');                     // 2001-05-23 00:00:00
echo new Carbon('last friday');                        // 2001-05-18 00:00:00
echo new Carbon('this thursday');                      // 2001-05-24 00:00:00
Carbon::setTestNow();                                  // always clear it !
```

The list of words that are considered to be relative modifiers are:


* `+`
* `-`
* `ago`
* `first`
* `next`
* `last`
* `this`
* `today`
* `tomorrow`
* `yesterday`

Be aware that similar to the next(), previous() and modify() methods some of these relative modifiers will set the time to 00:00:00.

`Carbon::parse($time, $tz)` and `new Carbon($time, $tz)` both can take a timezone as second argument.

```php
echo Carbon::parse('2012-9-5 23:26:11.223', 'Europe/Paris')->timezone->getName(); // Europe/Paris
```

[See Carbonite for more advanced Carbon testing features](https://github.com/kylekatarnls/carbonite).

Carbonite is an additional package you can easily install using composer: `composer require --dev kylekatarnls/carbonite` then use to travel times in your unit tests as you would tell a story:

Add `use Carbon\Carbonite`; import at the top of the file.

```php
$holidays = CarbonPeriod::create('2019-12-23', '2020-01-06', CarbonPeriod::EXCLUDE_END_DATE);

Carbonite::freeze('2019-12-22'); // Freeze the time to a given date

var_dump($holidays->isStarted());     // bool(false)

// Then go to anytime:
Carbonite::elapse('1 day');

var_dump($holidays->isInProgress());  // bool(true)

Carbonite::jumpTo('2020-01-05 22:00');

var_dump($holidays->isEnded());       // bool(false)

Carbonite::elapse('2 hours');

var_dump($holidays->isEnded());       // bool(true)

Carbonite::rewind('1 microsecond');

var_dump($holidays->isEnded());       // bool(false)

Carbonite::release(); // Release time after each test
```
