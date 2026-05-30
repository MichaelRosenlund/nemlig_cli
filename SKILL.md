---
name: nemlig-cli
description: Drive the nemlig.com (Danish online grocery) CLI to search products, look up product details, view and add to the shopping basket, and view order history. Use whenever the user wants to query or shop on nemlig.com from the terminal or have an agent do it programmatically — e.g. "search nemlig for milk", "what's in my nemlig basket", "add product 701025 to my basket", "show my nemlig order history", or any request mentioning nemlig.com groceries. Always use the --json flag so output is machine-readable.
---

# nemlig.com CLI

A single-file Python CLI for the nemlig.com grocery API. Drive it with the global
`--json` flag so every command emits a parseable JSON object/array to stdout and a
`{"error": ..., "command": ...}` object on failure.

## Invocation

If installed as a tool (`uv tool install`), the executable is `nemlig`:

```bash
nemlig --json <command> [args]
```

Otherwise run in place from the repo: `uv run python nemlig_cli.py --json <command> [args]`.

`--json` works **before or after** the subcommand (`nemlig --json search milk` ==
`nemlig search milk --json`). Always pass it for programmatic use.

## Credentials (required for every command)

All commands authenticate against nemlig.com. Credentials are resolved in this
order:

1. **Flags:** `-u "$NEMLIG_USER" -p "$NEMLIG_PASS"`
2. **Env vars:** `NEMLIG_USER` / `NEMLIG_PASS` (picked up automatically — if these
   are set, no flags are needed: `nemlig --json search "milk"` just works)
3. **Config file** `~/.config/nemlig/login.json`:
   ```json
   {"username": "your@email.com", "password": "yourpassword"}
   ```

Prefer env vars when available; fall back to flags otherwise.

## Commands

| Command | Purpose | stdout JSON on success |
|---|---|---|
| `search QUERY [-l N]` | Search products (default limit 10) | JSON **array** of product objects |
| `details PRODUCT_ID` | Full product info | JSON product **object** |
| `basket` | Current shopping basket | JSON basket **object** (has `Lines`) |
| `add PRODUCT_ID [-q N]` | Add product to basket (qty default 1) | JSON basket **object** after adding |
| `history [ORDER_ID] [-l N]` | Order history, or one order's details | see below |

`history` with no ID → object `{"NumberOfPages", "Orders": [...]}`. With an
`ORDER_ID` → object `{"order": {...}, "lines": [...]}`.

Output is the **raw nemlig.com API JSON** (PascalCase keys), so objects contain more
fields than listed below. Key fields:

- **Product** (search/details): `Id`, `Name`, `Brand`, `Price`, `Description`,
  `UnitPriceCalc`, `UnitPriceLabel`, `Availability.IsAvailableInStock`, `Url`,
  `PrimaryImage`. `details` adds `Category`, `SubCategory`, `Attributes`, `Labels`,
  `Campaign`, `Text` (HTML).
- **Basket line** (`Lines[]`): `Id`, `Name`, `Brand`, `Quantity`, `ItemPrice`, `Price`.
- **Order** (`Orders[]` / `order`): `Id`, `OrderNumber`, `Total`, `SubTotal`,
  `Status` (1=Pending, 2=Processing, 4=Delivered), `OrderDate`, `DeliveryTime`.
- **Order line** (`lines[]`): `ProductNumber`, `ProductName`, `Quantity`, `Amount`,
  `AverageItemPrice`, `HasCampaign`.

`PRODUCT_ID` is the `Id` from a search result. To act on a product by name, `search`
first, then use the returned `Id` with `details`/`add`.

## Error handling

On failure the CLI prints a JSON error object to stdout and **exits non-zero**:

```json
{"error": "Product 000000 not found. ...", "command": "details"}
```

Always check the exit code: `0` = success, non-zero = failure (parse the `error`
field for the reason — auth failure, product not found, network/HTTP error, etc.).
Progress text goes to stderr, so parsing **stdout** alone yields clean JSON.

## Examples

```bash
# Find a product and get its id
nemlig --json search "cocio" -l 5 | python -c "import json,sys;print(json.load(sys.stdin)[0]['Id'])"

# Product details
nemlig --json details 5070417

# View basket
nemlig --json basket

# Add 2 of a product
nemlig --json add 5070417 -q 2

# Recent orders, then one order's line items
nemlig --json history -l 10
nemlig --json history 12345678
```
