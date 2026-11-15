# btc-cent-forecast-01 — damped-momentum BTC forecast

Forecasts the BTC/USD price 240 minutes after each checkpoint.

## Strategy

```
forecast = last_close + 0.5 * (last_close - close_60min_ago)
```

capped so the extrapolated move never exceeds 1.5% of the last close.

- **Why momentum:** on the 5 practice cases whose actuals are derivable
  (p00–p04), the 60-minute trend kept running into the next 4 hours often
  enough that carrying half of it forward beats guessing the price stands
  still.
- **Why damped (k = 0.5, not 0.7–1.0):** the 60-minute move flips sign
  against the 240-minute move in 2 of 5 cases, and the practice window is a
  single trending day — a higher k wins there by overfitting. k = 0.5 is the
  robust middle: it still captures trend continuation while limiting the
  damage when the trend reverses.
- **Why the cap:** guards against a wild last-hour spike (e.g. a large
  trade in the thin cbBTC/USDC pool) producing an extreme forecast.

## Practice backtest (5 derivable cases, cents of absolute error)

| case | this entry | persistence |
|------|-----------:|------------:|
| p00  | 35,187     | 31,711      |
| p01  | 68,172     | 76,697      |
| p02  | 15,600     | 47,033      |
| p03  | 39,992     | 56,685      |
| p04  | 26,895     | 23,147      |
| **mean** | **37,169** | **47,055** |

~21% better than the persistence baseline on practice. Practice is one
trending day, so treat this as indicative, not a guarantee for the final.

## Implementation

`main.py` — pure Python 3.13 stdlib, deterministic, no network. Reads every
`*.in` in `$MAKEFUN_INPUT_DIR` (else `/input`), writes one `.out` per case
to `$MAKEFUN_OUTPUT_DIR` (else `/output`) in the exact
`{"forecasts": [{"case": "<id>", "price": <float>}]}` shape, chmod 0644.
Runs in ~0.05 s for all 6 practice cases.
