# Debugging notes

## What the code does

`split_bill(subtotal, people, tip_percent)` validates that `people` is a positive integer and that subtotal and tip are non-negative, converts inputs to `Decimal`, computes `tip = subtotal * tip_percent / 100`, rounds `subtotal + tip` to cents (ROUND_HALF_UP), then divides that total by `people`, rounds one share to cents, and returns `[share] * people` (every person gets the identical rounded share).

## Baseline evidence

Ran `python -m pytest -q` on the unchanged code: all existing tests pass (zero subtotal and validation only). Then ran `python -c "from src.bill_splitter import split_bill; s=split_bill('10.00', 3); print(s, sum(s))"` which printed three shares of `3.33` summing to `9.99`, one cent less than the `10.00` bill. This establishes the reported cent mismatch is reproducible and not covered by any test.

## Hypothesis

The bug is in the final step: rounding a single share `total / people` and repeating it `people` times. When the total in cents is not divisible by `people`, the leftover cents are lost or added (e.g. 10.00 / 3 -> 3 x 3.33 = 9.99, and 0.05 / 2 -> 2 x 0.03 = 0.06). If true, the sum of shares differs from the rounded total whenever `total_cents % people != 0`, and distributing the remainder cents one at a time to the first shares will make the sum always equal the total.
