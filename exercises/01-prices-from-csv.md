# Exercise 1: prices from a CSV file

This exercise is short and meant to get you familiar with the domain and the data itself. It comes with a prompt you can give your coding agent. The agent sets up the basis and layout of the project, you write the code yourself, and then the agent reviews your work. That way you still do the learning, without having to figure out the whole project setup on day one.

Download daily prices for Apple as a CSV from the EODHD demo API by opening this link in your browser: <https://eodhd.com/api/eod/AAPL.US?api_token=demo&fmt=csv>. The demo key only works for a few tickers, such as `AAPL.US`, `TSLA.US` and `AMZN.US`.

Load it with pandas, compute the daily returns and the 50-day moving average, plot the close price together with the moving average, and find the worst day and the maximum drawdown.

Prompt for your agent:

```text
Set up a new Python project for me with uv: src layout, pandas and matplotlib as
dependencies, ruff, mypy and pytest as dev dependencies.

Create a module with typed function stubs and Google-style docstrings, but do not
implement them:
- load_prices(csv_path) -> DataFrame
- compute_daily_returns(prices) -> Series
- compute_moving_average(prices, window_days) -> Series
- compute_max_drawdown(prices) -> float
- plot_close_with_moving_average(prices, window_days, output_path) -> None

I will write the implementations myself. When I ask for a review, check my code
against https://github.com/Duitenberg/onboarding and explain what you would change
and why, without rewriting it for me.
```

Back to the [onboarding guide](../README.md).
