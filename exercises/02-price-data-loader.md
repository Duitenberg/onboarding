# Exercise 2: a price data loader with adapters

This exercise is short and meant to get you familiar with the domain and the data itself. It comes with a prompt you can give your coding agent. The agent sets up the basis and layout of the project, you write the code yourself, and then the agent reviews your work. That way you still do the learning, without having to figure out the whole project setup on day one.

Build one loader with two adapters: one that reads the CSV file from [exercise 1](01-prices-from-csv.md), and one that calls the EODHD API directly. Both return the same typed `Quote` objects, validate every row, and raise an error on missing or invalid values. Then check whether the two sources actually agree on the close prices.

Prompt for your agent:

```text
Set up a new Python project for me with uv: src layout, httpx as a dependency,
ruff, mypy (strict) and pytest as dev dependencies, and a GitHub Actions workflow
that runs all three.

Lay out a price data loader with adapters, but do not implement the logic:
- a frozen Quote dataclass (ticker, date, open, high, low, close, volume)
- a PriceSource Protocol with fetch_daily_quotes(ticker, start_date, end_date) -> list[Quote]
- two empty adapters: CsvPriceSource and EodhdPriceSource (EODHD end-of-day API, demo key)
- a tests/ folder mirroring src/ with named but empty test cases for: a valid file,
  a missing column, a missing close price, and an integration test against the API

I will write the implementations and tests myself. When I ask for a review, check
my code against https://github.com/Duitenberg/onboarding, focusing on types,
validation at the boundary and one job per function. Explain what you would change
and why, without rewriting it for me.
```

Back to the [onboarding guide](../README.md).
