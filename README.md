# How to Test for Equal JSON Columns in Laravel Models

This repository contains a guide and example code demonstrating how to test for equality between JSON columns in Laravel models. JSON data, stored as strings in the database, can present challenges for direct comparisons due to differences in encoding. This guide offers a practical solution using `$this->castAsJson()` to ensure reliable and accurate tests.

## Overview
The main focus is on:
- Understanding challenges in comparing JSON columns.
- Implementing model and test code to handle JSON attributes.
- Resolving comparison issues using Laravel-specific solutions.

## Contents

- **Article**:
  [How to Test for Equal JSON Columns in Laravel Models](https://dev.to/tegos/how-to-test-for-equal-json-columns-in-laravel-models-24e)
- **Example Model**: `PriceSchedule`
- **Example Test**: `PriceExportScheduleTest`

## Key Concepts
1. **Challenges**:
    - JSON data stored as strings in the database can have encoding differences.
    - Direct string comparisons may fail even when data is logically equivalent.

2. **Solution**:
    - Use `$this->castAsJson()` to ensure consistent JSON format before comparison.

## Example Usage
### Model
The `PriceSchedule` model includes JSON columns cast to arrays for easy manipulation:

```php
final class PriceSchedule extends Model
{
    protected $fillable = ['user_id', 'price_supplier_id', 'weekday', 'hour', 'is_active'];
    protected $casts = ['weekday' => 'array', 'hour' => 'array'];
}
```

### Test
The `PriceExportScheduleTest` demonstrates updating and verifying JSON columns:

```php
$this->assertDatabaseHas(PriceSchedule::class, [
    'user_id' => $user->id,
    'is_active' => $updatedData['is_active'],
    'weekday' => $this->castAsJson($updatedData['weekday']),
    'hour' => $this->castAsJson($updatedData['hour']),
]);
```

## Running Tests
Run the test suite to verify functionality:

```bash
php artisan test
```

## License
This repository is licensed under the MIT License. See the `LICENSE.md` file for more details.

## Contributing
Contributions are welcome! Please submit a pull request or open an issue if you encounter any problems or have suggestions.

## Feedback
If you find this guide helpful or have suggestions for improvement, feel free to reach out via issues or discussions.
