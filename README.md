# duckdb-examples

DuckDB examples for streams and demos. This repository contains practical, step-by-step tutorials for working with DuckDB, with a focus on terminal commands and real-world workflows.

## Tutorials

### 1. [Read and Import Local Files](./local-files-quick-start/01-read-import-local-files.md)
A comprehensive guide to reading and importing local files with DuckDB, covering:
- **Parquet files** - The most efficient format for DuckDB
- **CSV files** - Auto-inference and custom schemas
- **JSON files** - Handling complex nested structures
- **Text & Markdown files** - Reading file content as queryable data

Perfect for beginners or anyone learning DuckDB's elegant data import capabilities.

**Location:** `local-files-quick-start/`

### 2. [Read and Import Remote Files](./remote-sources-quick-start/01-read-import-remote-files.md)
A comprehensive guide to reading and importing remote files from the internet with DuckDB, covering:
- **CSV, Parquet & JSON files** - Loading data directly from URLs
- **HTTP headers & authentication** - Working with password-protected resources
- **Performance considerations** - Tips for querying large remote datasets
- **Real-world examples** - Practical workflows for remote data analysis

Learn how to query data without downloading it first!

**Location:** `remote-sources-quick-start/`

## Getting Started

1. Install DuckDB: https://duckdb.org/docs/installation/
2. Choose a tutorial above and follow the terminal commands
3. All examples are designed to run directly in the DuckDB CLI

## Sample Data Files

This repository includes sample data files for hands-on practice:

- **orders.csv** - Sample orders data for CSV import examples
- **products.parquet** - Sample products data for Parquet import examples

These files are used in the "Complete Workflow Example" section of the tutorial. No setup needed—just clone the repo and start experimenting!

## About

These tutorials are based on [The Full Data Stack](https://thefulldatastack.substack.com/) by Hoyt Emerson and demonstrate DuckDB's elegant approach to data import and querying.

---

More tutorials coming soon!
