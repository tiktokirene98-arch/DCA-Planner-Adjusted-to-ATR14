# DCA Planner Adjusted to ATR14

Binance Agent OS Mini Hackathon — **Track A**

An agentic planner that turns live Binance volatility into a dollar-cost-averaging buy ladder.

Most DCA grids use fixed percentages. This tool spaces limit buys as multiples of the asset’s **14-day Average True Range (Wilder)**, fetched through the **Binance MCP server**.

## What it does

1. Reads your pair, USDT budget, number of tiers, ATR spacing, and allocation style.
2. Asks the Binance Agent OS connector for ~20 daily candles.
3. Computes ATR14 with Wilder smoothing.
4. Builds limit-buy tiers below spot:
   - `price_i = spot − i × spacing × ATR14`
5. Splits the budget (linear / equal / exponential).
6. Renders current price, ATR14, ATR as % of spot, the ladder, and a short volatility note.

Levels are **indicative**. The tool does **not** place orders.

## Why Agent OS

| Component | Role |
|---|---|
| MCP `https://agent.binance.com/mcp/agentic` | Daily klines and last price |
| Compatible agent (Claude / Claude Code / Messages API) | Tool calls + ATR + rationale JSON |
| Static HTML UI | Settings + ladder rendering |

Market-data scope only. No trading permission required.

## ATR method

TR = max(high − low, |high − prevClose|, |low − prevClose|)
ATR14_0 = mean(first 14 TRs)
ATR14_t = (ATR14_{t-1} × 13 + TR_t) / 14

Default settings: `BTCUSDT`, 1000 USDT, 5 tiers, `0.75× ATR`, linear weighting (deeper tiers get more size).

## Quick start

### 1. Connect Binance MCP

Claude Code:

```bash
claude mcp add binance-mcp-server --transport http https://agent.binance.com/mcp/agentic

Then /mcp → binance-mcp-server → Authenticate. Grant market-data access.Docs:Agent OS
MCP Server

Do not paste the MCP URL into a normal chat and ask the model to install it.2. Open the UIOpen dca-planner-atr14.html in a browser.3. Generate a planKeep the defaults and click Generate plan.You should see spot, ATR14, five limit rows, allocation bars, and a volatility note. Try ETHUSDT or SOLUSDT and switch weighting.Agent contractThe model must answer with JSON only:json

{
  "symbol": "BTCUSDT",
  "currentPrice": 77931.59,
  "atr14": 2487.35,
  "atrPct": 3.19,
  "asOf": "2026-09-08 14:20:00 UTC",
  "rationale": "Short sentence on current vs recent volatility."
}

System prompt used by the UI:

You are a market volatility calculation module. You are given a crypto pair symbol.
Use the available tools from the market connector to fetch the daily candles (klines)
for the last ~20 days for this symbol (high, low, close per day).
Compute the 14-period ATR using Wilder's smoothing:
1. TR = max(high-low, abs(high-previous_close), abs(low-previous_close))
2. First ATR14 = simple average of the first 14 TR values
3. Subsequent ATR = (previous_ATR * 13 + today_TR) / 14
Use the latest ATR14 and the latest close (or current price).
Respond ONLY with valid JSON, no markdown, keys:
symbol, currentPrice, atr14, atrPct, asOf, rationale

Ladder math (weights, buildLadder) stays in the browser. The agent only returns price + ATR + rationale.Project layout

dca-planner-atr14.html   # UI + client logic


How to reproduceCreate a Binance account in a supported region.
Add the official MCP server in a compatible client (Claude Code, Claude, ChatGPT, Cursor, or VS Code):
https://agent.binance.com/mcp/agentic
Authenticate and grant market-data scope only.
Confirm the connector works by asking the agent for the last 20 daily klines on BTCUSDT.
Save this repository’s HTML file and open it locally.
Click Generate plan. The UI calls the agent with the system prompt above, parses the JSON, and draws the ladder.
Optional: point the MCP URL field at another market-compatible connector.

SafetyIndicative tiers, not financial advice.
Double-check tick size and balances before placing any order.
Keep trading scopes disabled unless you later add an explicit execution step.
Agent OS cannot withdraw from your main account; still fund only what you can afford to use.

