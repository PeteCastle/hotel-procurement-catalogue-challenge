# Dataset Description

## Files

- **`train.csv`**: 2,445 labelled catalogue items
- **`test.csv`**: 820 items to classify
- **`sample_submission.csv`**: the submission format
- **`attributes_sample.csv`**: 200 items for the attribute extraction task (submitted by email, not on Kaggle)
- **`attributes_submission_template.csv`**: fill this in for the attribute extraction task and email it with your write-up

## Columns

| Column | Description |
|---|---|
| `id` | Row identifier |
| `supplier` | The supplier that listed the item |
| `supplier_sku` | The supplier's own product code |
| `description` | Free-text product description from the supplier's catalogue |
| `unit_of_measure` | Selling unit (`EA`, `KG`, `PK`, `CTN`, `CS`, `BOX`, `BTL`) |
| `unit_price` | Price per selling unit |
| `buyer_group` | The buyer team assigned by procurement during category review |
| `category` | Target, train only |

## Categories

### Food and beverage

- Fresh Produce
- Meat & Poultry
- Seafood
- Dairy & Eggs
- Bakery
- Dry Goods & Pantry
- Oils, Sauces & Condiments
- Beverages: Non-Alcoholic
- Beverages: Alcoholic
- Coffee & Tea
- Frozen Prepared Foods

### Operating supplies

- Bathroom Amenities
- Cleaning Chemicals
- Food Service Disposables
- Kitchen Equipment
- Tableware
- Linen & Towels
- Pest Control Supplies

## Notice

All data is synthetic. Supplier and brand names are made up. This data is provided only for use in this assessment. Please don't publish or redistribute it.
