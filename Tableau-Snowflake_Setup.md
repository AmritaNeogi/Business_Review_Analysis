# Connecting Tableau Desktop to Snowflake
### Setup Guide for Business Review Analysis Project

---

## Overview

This guide walks through connecting Tableau Desktop to Snowflake using a **live connection** — meaning Tableau queries Snowflake directly in real time. No CSV exports, no stale data.

**Why live connection over CSV export?**
- Data stays in Snowflake — single source of truth
- Dashboard always reflects the latest data
- Demonstrates a real production-grade BI workflow
- More impressive in interviews — shows end-to-end pipeline thinking

---

## Prerequisites

Before starting, make sure you have:

| Requirement | Details |
|---|---|
| Tableau Desktop | Trial or paid license — **not** Tableau Public (Public cannot connect to Snowflake) |
| Snowflake account | With access to `BUSINESS_REVIEWS_DB` and `PUBLIC` schema |
| Snowflake credentials | Username and password |
| Snowflake account URL | Found in your browser address bar when logged into Snowflake |
| Mac or Windows | Steps below cover Mac — Windows steps are nearly identical |

---

## Step 1 — Download and Install Tableau Desktop

1. Go to: **https://www.tableau.com/products/desktop/download**
2. Fill in your details and download the installer
3. Open the installer and follow the prompts
4. When prompted to activate, choose **"Start Trial"** (14-day free trial)

> **Note:** Tableau Desktop is a separate product from Tableau Public and Tableau Cloud. Make sure you download Desktop specifically — it is the only version that supports live Snowflake connections.

---

## Step 2 — Install the Snowflake ODBC Driver (Mac)

Tableau needs an ODBC driver to communicate with Snowflake. This is a one-time setup.

### Why is this needed?
ODBC (Open Database Connectivity) is a standard interface that lets applications like Tableau talk to databases like Snowflake. Without it, Tableau cannot establish the connection.

### Download the driver:
1. Go to: **https://developers.snowflake.com/odbc/**
2. Click the **MacOS** tab
3. Download the latest version — file will be named something like:
   `snowflake_odbc_mac_64universal-3.17.0.dmg`
4. The `macuniversal` architecture works for both Intel and Apple Silicon (M1/M2/M3) Macs

### Install the driver:
1. Open the downloaded `.dmg` file from your Downloads folder
2. Double-click the `.pkg` file inside it
3. Click through the installer — Next → Agree → Install
4. Enter your Mac password if prompted
5. Once installed, **fully quit Tableau Desktop** (`Cmd + Q`)
6. **Reopen Tableau Desktop**

> **Important:** You must restart Tableau after installing the driver. If you skip this, Tableau will not detect the driver and will redirect you to the download page instead of opening the connection dialog.

---

## Step 3 — Find Your Snowflake Account URL

1. Open Snowflake in your browser and log in
2. Look at the URL in the address bar — it will look like:
   `https://abc12345.snowflakecomputing.com`
3. Your account identifier is the part between `https://` and `.snowflakecomputing.com`
4. For example, if your URL is:
   `https://app.snowflake.com/oukpgop/sc41197/#/workspaces/...`
   Your server value is: `oukpgop-sc41197.snowflakecomputing.com`

> **Tip:** The format is `[orgname]-[accountname].snowflakecomputing.com`. Replace the `/` between org and account with a `-`.

---

## Step 4 — Connect Tableau to Snowflake

1. Open **Tableau Desktop**
2. On the left panel under **"To a Server"**, click **"More..."**
3. In the connector list, click **"Snowflake"**
4. The Snowflake connection dialog will open — fill in the following:

| Field | Value |
|---|---|
| Server | `oukpgop-sc41197.snowflakecomputing.com` ← your account URL |
| Role | `ACCOUNTADMIN` |
| Warehouse | `LARGEDATA_WH` |
| Authentication | Change dropdown to `Username and Password` |
| Username | Your Snowflake username |
| Password | Your Snowflake password |

5. Click **Sign In**

---

## Step 5 — Select Database and Table

Once connected, Tableau will show a data source panel on the left:

1. **Warehouse** — should already show `LARGEDATA_WH`
2. **Database** — select `BUSINESS_REVIEWS_DB` from the dropdown
3. **Schema** — select `PUBLIC`
4. Under **Table**, you will see all your Snowflake tables listed

---

## Step 6 — Bring in the Data

For this project, drag **`YELP_REVIEWS_ENRICHED`** into the canvas area.

**Why this table specifically?**
`yelp_reviews_enriched` is the pre-joined table that combines review and business data in one place — `business_id`, `business_name`, `city`, `state`, `review_stars`, `review_date`, `sentiments`, and more. Using this single table avoids complex joins inside Tableau and keeps the dashboard performant on a live connection.

> **Note:** Avoid dragging raw variant tables (`yelp_reviews`, `yelp_businesses`) into Tableau — these contain unparsed JSON and will not render correctly as dimensions or measures.

---

## Step 7 — Verify the Connection

Before building dashboards:

1. Click **"Sheet 1"** at the bottom of the screen
2. You should see dimensions and measures populated on the left panel
3. Drag `REVIEW_DATE` to the Columns shelf and `COUNT(REVIEW_STARS)` to Rows
4. If a chart renders — your live connection is working correctly

---

## Connection Summary

```
Snowflake Account:   oukpgop-sc41197.snowflakecomputing.com
Database:            BUSINESS_REVIEWS_DB
Schema:              PUBLIC
Warehouse:           LARGEDATA_WH
Role:                ACCOUNTADMIN
Primary Table:       YELP_REVIEWS_ENRICHED
Connection Type:     Live (real-time, no extract)
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Clicking Snowflake opens the driver download page | ODBC driver not installed or Tableau not restarted after install — reinstall driver and fully restart Tableau |
| "Invalid account identifier" error | Check your server URL format — should be `orgname-accountname.snowflakecomputing.com` with a `-` not `/` |
| "Incorrect username or password" | Double check Snowflake credentials — make sure you are using your Snowflake login, not your email provider login |
| Tables not visible after connecting | Make sure Database is set to `BUSINESS_REVIEWS_DB` and Schema is set to `PUBLIC` |
| Dashboard loads slowly | Switch warehouse to X-Large in Snowflake for heavy queries, or switch connection type to Extract for static dashboards |

---

## References

- Tableau Snowflake Connector Docs: https://help.tableau.com/current/pro/desktop/en-us/examples_snowflake.htm
- Snowflake ODBC Driver Download: https://developers.snowflake.com/odbc/
- Snowflake + Tableau Best Practices: https://www.snowflake.com/en/partners/technology/tableau/
