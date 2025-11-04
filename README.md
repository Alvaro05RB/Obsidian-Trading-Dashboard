# Obsidian Trading Dashboard

A complete **trade logging and analysis system** built in **Obsidian**.  
The project combines detailed trade journaling with automated performance tracking and visualization, offering a full overview of both trading and long-term portfolio performance.

---

## 📁 Overview

The system is designed around a folder-based structure compatible with Obsidian.  
Each trade, chart, and performance file is automatically integrated into the dashboard once placed in its corresponding directory.

Example trades are included as samples.

---

## 🧾 Logging Trades

1. Duplicate the **Trade Template** file.
2. Fill in the trade details.
3. Save it inside the corresponding `Trades/Trade/` folder following the naming convention: TICKERDD-MM-YYYY
4. Paste a screenshot of the trade inside the note, then delete the image from the note.
- The image will appear in the root directory — move it to the correct date folder in `Charts/`.
- Rename the image to match the trade note’s name.
5. Back in the trade note:
- Click the missing image link (`TICKERDD-MM-YY.png could not be found`).
- Rename the text before `.png` to match the trade note’s filename — the image will now be linked.
- Finally, go to the image file and **add a space between the ticker and the date** to fix Obsidian’s linking bug.

---

## 📊 Trading Dashboard

The dashboard automatically detects new folders for each year (just name them `YYYY`), so **no code modification is required**.

### Features:
- View overall trading performance.
- Filter by:
  - Year
  - Month
  - Day
  - Custom date ranges
  - System type
  - Trade direction (Long/Short)
- Switchable performance charts:
- Yearly, monthly, or daily PnL & R visualization.
- View all trades meeting the selected criteria in a table.
- Clicking on a trade’s file link opens its detailed note.

To change the dashboard banner:
1. Enter *Edit Mode* (pencil icon, top-right corner).
2. Replace `Banner.gif` with your desired file name (make sure the extension matches).

---

## 💼 Portfolio Manager

The **Portfolio Manager** separates **short-term trading performance** from **long-term holdings**.

### Functionality:
- Tracks both trade-based profits and simulated portfolio returns.
- Allows manual deposits and withdrawals for more accurate performance metrics.
- Updates portfolio balance upon closing an investment.

> All portfolio data is stored at  
> `Trades/Trade/PortfolioManager.md`

Since Obsidian works offline, market prices aren’t live-updated — but closed trade data keeps portfolio performance consistent with your logs.

---

## 📈 Trades Summary

This section uses Obsidian’s built-in summary files to visualize all trades collectively.  
It’s a lightweight alternative view of trading activity — still in early development due to current Obsidian limitations.

---

## 🧠 Morning market analysis and End-of-Day Review

A complete review framework organized by **year**, including:
- Daily
- Monthly
- Yearly templates

Each section can be duplicated annually.  
There’s no code dependency — feel free to rename and customize as you wish.

---

## ⚙️ Systems

Includes:
- A **System Template**
- Example folders for:
- Mean Reversion
- Trend Trading

These are for documentation and organization purposes; no code dependencies exist, so you can freely expand or rename them.

---

## 🧩 Required Obsidian Plugins

To ensure the dashboard and all visualizations work properly, install the following Obsidian community plugins:

| Plugin | Purpose |
|--------|----------|
| **Advanced Tables** | For enhanced Markdown table editing and formatting. |
| **Banners** | Displays visual banners at the top of dashboard and key notes. |
| **Charts** | Used to render performance charts (PnL, R, etc.). |
| **DataView** | Enables dynamic filtering and querying of trades and portfolio data. |
| **Force Note View Mode** | Forces notes (like the Portfolio Manager) to open in a specific view mode. |
| **Homepage** | Defines a custom default note to open when launching the vault (set to Dashboard). |
| **Iconize** | Adds icons to folders and files for improved visual organization. |

### Plugin Configuration

- **DataView**  
  Enable the following options in settings:  
  - ✅ Inline queries  
  - ✅ JavaScript queries  
  - ✅ Inline JavaScript queries  

- **Homepage**  
  Set the default homepage to:  Dashboard
  
- **Force Note View Mode**  
Add the following note to open in preview mode:  Portfolio Manager

---

## 🧩 Summary

This dashboard brings together:
- Trade logging  
- Visual analysis  
- Portfolio tracking  
- Yearly journaling and system documentation  

All within an **Obsidian-based**, fully offline structure designed for traders & investors who want transparency, order, and performance insights.

---

## 🪪 License

This project is licensed under the **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)** license.

You are free to:
- Share — copy and redistribute the material in any medium or format.
- Adapt — remix, transform, and build upon the material.

Under the following terms:
- **Attribution** — You must give appropriate credit.
- **NonCommercial** — You may not use the material for commercial purposes.
- **ShareAlike** — If you remix or build upon the material, you must distribute your contributions under the same license.

For full legal terms, see:  
[https://creativecommons.org/licenses/by-nc-sa/4.0/](https://creativecommons.org/licenses/by-nc-sa/4.0/)
