# Swing Highs & Lows Alert — MQL4 Script

A MetaTrader 4 script that detects **confirmed swing high and swing low price structures** by scanning a rolling lookback window using a nested bilateral comparison algorithm — requiring each candidate bar's high or low to be strictly greater or lesser than all bars within a configurable `ConfirmationPeriod` on both sides — and fires alerts only when a newly detected swing point differs from the previously stored `lastSwingHigh` or `lastSwingLow` value.

---

## Overview

Swing highs and lows are the foundational building blocks of technical market structure analysis. A confirmed swing high is a price bar whose high is strictly greater than the highs of all neighboring bars within the confirmation window on both its left and right sides — making it a locally dominant peak. A confirmed swing low is symmetrically the lowest low in its bilateral neighborhood. These structures define support and resistance, trend direction (higher highs / higher lows for uptrends; lower highs / lower lows for downtrends), and key levels for stop placement and entry triggers. This script implements a rigorous bilateral confirmation scan using nested `j` loops that check `ConfirmationPeriod` bars on each side of each candidate bar, ensuring only genuinely dominant swing points fire alerts — not mere local wobbles within noise. The `lastSwingHigh` and `lastSwingLow` state variables prevent refiring the same swing level on consecutive cycles.

---

## Features

- **Bilateral `ConfirmationPeriod` validation** — inner `j = 1` to `ConfirmationPeriod` loop checks `iHigh(i) > iHigh(i − j)` AND `iHigh(i) > iHigh(i + j)` simultaneously; a single false on either side breaks and disqualifies the candidate
- **Lookback-windowed scan** — outer `i = ConfirmationPeriod` to `LookbackPeriod` loop scans all candidate bars within the configurable lookback, returning the first confirmed swing encountered
- **`lastSwingHigh` / `lastSwingLow` deduplication** — swing alert only fires when `swingHigh != lastSwingHigh && swingHigh != 0`, preventing repeated alerts on an unchanged confirmed swing across multiple loop cycles
- **Zero-return guard** — `DetectSwingHigh()` and `DetectSwingLow()` both return `0.0` if no confirmed swing is found in the lookback window, cleanly suppressing alerts when market structure is ambiguous
- **Independent high and low tracking** — `lastSwingHigh` and `lastSwingLow` maintained as separate local variables, updated independently, so a new swing low does not interfere with swing high tracking
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)
- Alert message includes the confirmed swing price level with symbol and timeframe for immediate structure reference

---

## How It Works

1. Every minute, `DetectSwingHigh()` and `DetectSwingLow()` independently scan `i = ConfirmationPeriod` to `LookbackPeriod`
2. For each `i`, nested `j` loop verifies bilateral dominance across `ConfirmationPeriod` neighbors on both sides
3. First confirmed swing found is returned; `0.0` returned if none found in the window
4. Main loop compares returned values against `lastSwingHigh` / `lastSwingLow`:
   - New non-zero swing high → **New Swing High Detected** alert; `lastSwingHigh` updated
   - New non-zero swing low → **New Swing Low Detected** alert; `lastSwingLow` updated

---

## Input Parameters

| Parameter            | Type            | Default     | Description                                                          |
|----------------------|-----------------|-------------|----------------------------------------------------------------------|
| `TradeSymbol`        | string          | `EURUSD`    | Symbol for analysis                                                  |
| `Timeframe`          | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for swing detection                                        |
| `LookbackPeriod`     | int             | `10`        | Maximum bars back to scan for swing candidates                       |
| `ConfirmationPeriod` | int             | `3`         | Number of bars on each side required to confirm a swing point        |
| `EnableAlerts`       | bool            | `true`      | Fire an on-screen/sound alert                                        |
| `EnableEmail`        | bool            | `false`     | Send an email notification                                           |
| `EnablePush`         | bool            | `false`     | Send a mobile push notification                                      |

---

## Alert Message Format

```
New Swing High Detected detected on EURUSD (Timeframe: PERIOD_H1)
Price: 1.08640
```

---

## Installation

1. Copy `H_and_L_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

> **Note:** `ConfirmationPeriod` determines how strict the swing definition is. A value of `1` accepts any local high/low with one confirming bar on each side; `3` requires three consecutive confirming bars on each side, producing fewer but higher-quality signals.

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
