# 🤯 Awesome E-commerce & Marketing Data Prep

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![Privacy: Zero Uploads](https://img.shields.io/badge/Privacy-Zero_Uploads-success)](#)
[![Data: CSV & JSON](https://img.shields.io/badge/Data-CSV_%7C_JSON_%7C_Parquet-blue)](#)

Welcome to the ultimate collection of data nightmares. 

If you work with exports from **Shopify, Amazon, Stripe, Apollo.io, or Meta Ads**, you already know the pain: broken CSVs, nested JSONs, multi-line order chaos, and invisible format poisons that crash your Excel.

This repository documents the exact structural traps in these platform exports and provides the logical blueprints to solve them locally.

---

### ⭐ Why Star this Repo?
Even if you prefer writing your own Python scripts, this repository serves as a comprehensive encyclopedia of SaaS data export traps. **Star it** to keep it in your bookmarks for the next time a client hands you a broken Shopify CSV or a toxic Apollo lead list, and you need to understand *why* the data is corrupted before writing your own cleaning script.

---

## 🛠️ The Ultimate Local Solution

Tired of writing Python scripts or watching Excel freeze on a 2GB file? 

This repository is proudly maintained by **[dataprep.dev](https://dataprep.dev)** — The Browser Data Toolkit. We built a pure WebAssembly (DuckDB-Wasm) engine to solve all the problems listed below. 


https://github.com/user-attachments/assets/334bad31-2b7f-4ba2-9211-df8a324b6764




* ⚡ **Blazing Fast**: Process millions of rows in seconds.
* 🛡️ **Zero Uploads**: Everything runs entirely in your browser's memory. Your sensitive data never touches a cloud server.
* 👉 **[Try it for free at dataprep.dev](https://dataprep.dev)**

---

## 📚 The Data Nightmare Directory

*(Click on each topic to see the deep-dive analysis and the exact cleaning logic)*

### 🛒 1. E-commerce & Finance Traps
* [Shopify Raw Order Exports: The Multi-line Nightmare](./shopify-orders.md)
* [Amazon Settlement V2: The FBA Profit Maze](./amazon-settlement.md)
* [WooCommerce Exports: The HTML Artifacts & Missing Headers](./woocommerce-cleaner.md)
* [Stripe Payouts: The Reconciliation Hell](./stripe-formatter.md)
* [Local VLOOKUP: The "Calculating 2%" Excel Crash](./local-vlookup.md)

### 🎯 2. Marketing & CRM Nightmares
* [Apollo.io Leads: Toxic Emails & Hard Bounces](./apollo-leads.md)
* [Klaviyo & Mailchimp: The Strict ESP Schema Hell](./klaviyo-prep.md)
* [Ads ROAS Pivots: Dirty Currency Symbols & Div/0 Errors](./ads-roas-pivot.md)
* [UTM Bulk Tagging: The Broken Links Disaster](./utm-generator.md)

### 🧹 3. Data Engineering & Format Poisons
* [The CSV Deduplication Trap: Why Exact Match Fails](./csv-deduplicator.md)
* [Invisible Format Poisons: BOM, Zero-width Spaces & \r\n](./format-cleaner.md)
* [The Regex Replace Dilemma: Excel's Missing Superpower](./regex-replace.md)
* [Merging 50 CSVs: Schema Drift & 1M Row Limits](./csv-merger.md)
* [Massive CSV Splitting: The 3GB Size Wall](./csv-splitter.md)
* [API JSON to CSV: The Deep Nesting Flattening Hell](./json-to-csv.md)
* [Markdown Tables: The Escaping & Alignment Disaster](./markdown-table.md)

### 🛡️ 4. Privacy, SQL & Advanced Analytics
* [PII Leaks: Why You Must Anonymize GDPR Data Locally](./gdpr-anonymizer.md)
* [Silent CSV Failures: Why Schema Validation is Mandatory](./csv-validator.md)
* [SQL on CSV: The Absurdity of Spinning up Postgres](./sql-on-csv.md)
* [CSV vs Parquet: The Modern Data Engineering Divide](./parquet-converter.md)

---

## 🤝 Contributing

Did a SaaS platform completely ruin their CSV export format recently? We want to know! 
Feel free to open an issue or submit a Pull Request to document the trap.

---

## 🚀 Ready to Stop Fighting with Dirty Data?

Don't let broken CSVs ruin your weekend. Join thousands of data engineers, marketers, and founders who process their sensitive data locally and instantly.

<p align="center">
  <a href="https://dataprep.dev" target="_blank">
    <img src="https://img.shields.io/badge/👉_Launch_dataprep.dev-100%25_Free_%7C_Zero_Uploads-2563eb?style=for-the-badge&logo=webassembly" alt="Launch dataprep.dev" height="40">
  </a>
</p>

---
**Disclaimer:** This is an independent open-source knowledge base. All trademarks and registered trademarks (Shopify, Amazon, Stripe, etc.) are the property of their respective owners.
