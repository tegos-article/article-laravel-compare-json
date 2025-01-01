# How to Test for Equal JSON Columns in Laravel Models

Testing equality between JSON columns in Laravel requires special consideration since JSON data is stored as strings in
the database. Differences in how JSON is encoded can lead to unexpected test failures when comparing JSON columns.
This article will guide you on effectively comparing JSON columns in your Laravel application's tests.

## Understanding the Challenge

When JSON data is stored in the database, it is saved as a string. Minor differences in JSON encoding, such as spacing
or ordering of keys, can cause direct string comparisons to fail. This means that even if the content is logically
equivalent, tests using `$this->assertDatabaseHas()` might fail.

## Example Model

First, consider the `PriceSchedule` model, which includes JSON columns:

```php
final class PriceSchedule extends Model
{
    protected $fillable = [
        'user_id',
        'price_supplier_id',
        'weekday',
        'hour',
        'is_active'
    ];

    protected $casts = [
        'weekday' => 'array',
        'hour' => 'array',
    ];
}
```

The `weekday` and `hour` attributes are cast to arrays, allowing easy manipulation in your application.

## Writing the Test

Here's an example test for updating a `PriceSchedule`:

```php
final class PriceExportScheduleTest extends TestCase
{
    public function test_price_export_schedule_update(): void
    {
        $user = UserFactory::new()->create();
        $this->actingAsFrontendUser($user);

        $priceSchedule = PriceScheduleFactory::new()->make();

        $updatedData = [
            'weekday' => $this->faker->randomElements(DayOfWeek::values(), 3),
            'hour' => $priceSchedule->hour,
            'is_active' => true,
        ];

        $response = $this->putJson(route('api-v2:price-export.suppliers.schedules.update'), $updatedData);

        $response->assertNoContent();

        $this->assertDatabaseHas(PriceSchedule::class, [
            'user_id' => $user->id,
            'is_active' => $updatedData['is_active'],
            'weekday' => $updatedData['weekday'],
            'hour' => $updatedData['hour'],
        ]);
    }
}
```

### Common Issue with JSON Comparisons

When using `$this->assertDatabaseHas()` to compare JSON-type values like `weekday` and `hour`, direct comparisons may
fail due to differences in JSON encoding. For example:

- Database-stored JSON: `{"key":"value"}`
- PHP-generated JSON: `{
  "key": "value"
}`

Even though the data is logically identical, the test might fail because the strings differ.

## Solution: Use `$this->castAsJson()`

To ensure consistent comparisons, use `$this->castAsJson()` when asserting JSON columns:

```php
$this->assertDatabaseHas(PriceSchedule::class, [
    'user_id' => $user->id,
    'is_active' => $updatedData['is_active'],
    'weekday' => $this->castAsJson($updatedData['weekday']),
    'hour' => $this->castAsJson($updatedData['hour']),
]);
```

This method ensures that both the test data and the database data are cast to a common JSON format before comparison.

## Test Output

Running the test produces the following result:

```bash
Price Export Schedule (PriceExportSchedule)
✔ Price export schedule update
OK (1 test, 3 assertions)
```

By using `$this->castAsJson()`, you can avoid JSON encoding issues and ensure that your tests are both reliable and
accurate.

