# DuckDB: Read and Import Local Files

A step-by-step tutorial on how to read and import local files with DuckDB. This covers Parquet, CSV, JSON, and text/markdown files.

> **Original Article**: [DuckDB Basics: Importing Data](https://thefulldatastack.substack.com/p/duckdb-basics-importing-data)


## Prerequisites

- DuckDB CLI installed. Get it [here](https://duckdb.org/docs/installation/)
- Sample data files (we'll show you how to get them)
- Basic familiarity with terminal commands

## Getting Started: Start a DuckDB Session

Before running any SQL commands, start a DuckDB session in your terminal:

```bash
duckdb
```

Once you're in the DuckDB CLI, you'll see a `D` prompt ready for SQL commands. All the commands below should be run within this interactive session.

To exit the DuckDB session, type:

```bash
.exit
```

---

## 1. Parquet Files

Parquet is arguably the most efficient file format to query with DuckDB. Reading from a Parquet file is incredibly simple.

### Basic Parquet Operations

Read from a local Parquet file:

```bash
SELECT * FROM 'test.parquet';
```

Read all Parquet files in a directory using a wildcard:

```bash
SELECT * FROM '*.parquet';
```

Create a persistent DuckDB table from a Parquet file:

```bash
CREATE TABLE test AS SELECT * FROM 'test.parquet';
```

Query the newly created table:

```bash
FROM test;
```

Note: With DuckDB, you can omit the `SELECT` keyword for simple select-all queries. Just use `FROM` and the table name!

---

## 2. CSV Files

CSV files often have schema quirks. DuckDB offers both automatic schema inference and explicit schema definition options.

### Download Sample CSV

To follow along with examples, download the flights CSV file:

```bash
curl https://duckdb.org/data/flights.csv -o flights.csv
```

### Basic CSV Operations

Read from a CSV file with auto-inferred schema:

```bash
SELECT * FROM 'flights.csv';
```

Create a DuckDB table from CSV:

```bash
CREATE TABLE ontime AS SELECT * FROM 'flights.csv';
```

Query the new table:

```bash
FROM ontime;
```

### CSV with Custom Schema

If your CSV has a non-standard format (custom delimiters, specific data types), use the `read_csv()` function:

```bash
SELECT *
FROM read_csv('flights.csv',
    delim = '|',
    header = true,
    columns = {
        'FlightDate': 'DATE',
        'UniqueCarrier': 'VARCHAR',
        'OriginCityName': 'VARCHAR',
        'DestCityName': 'VARCHAR'
    });
```

### Creating a Table from Complex CSV Files

For complex CSV files, create the table schema first, then use COPY to load the data:

```bash
CREATE TABLE ontime (
    FlightDate DATE,
    UniqueCarrier VARCHAR,
    OriginCityName VARCHAR,
    DestCityName VARCHAR
);

COPY ontime FROM 'flights.csv';

FROM ontime;
```

---

## 3. JSON Files

JSON can have similar schema challenges as CSV. DuckDB handles both auto-inference and explicit schema definition.

### Download Sample JSON

To follow along with examples, download the todos JSON file:

```bash
curl https://duckdb.org/data/json/todos.json -o todos.json
```

### Basic JSON Operations

Read from a JSON file with auto-inferred schema:

```bash
SELECT * FROM 'todos.json' LIMIT 5;
```

Create a DuckDB table from JSON:

```bash
CREATE TABLE todos AS SELECT * FROM 'todos.json';
```

Query the new table:

```bash
FROM todos;
```

---

## 4. Text and Markdown Files

DuckDB has a powerful feature for reading `.txt` and `.md` files, returning their content as queryable data.

### Reading Text Files

Read the content of a text file:

```bash
SELECT content FROM read_text('earlysignal.txt');
```

Get file metadata along with content:

```bash
SELECT size, parse_path(filename), content FROM read_text('earlysignal.txt');
```

### Reading Multiple Text Files

Get file size, path, and content for all `.txt` files in a directory:

```bash
SELECT size, parse_path(filename), content FROM read_text('*.txt');
```

Create a DuckDB table from all text files:

```bash
CREATE TABLE texts AS SELECT size, parse_path(filename), content FROM read_text('*.txt');

FROM texts;
```

### Working with Markdown Files

Read from a Markdown file:

```bash
SELECT size, parse_path(filename), content FROM read_text('example.md');
```

Insert Markdown content into an existing table:

```bash
INSERT INTO texts SELECT size, parse_path(filename), content FROM read_text('example.md');

FROM texts;
```

---

## Complete Workflow Example

Here's a complete workflow demonstrating several file types. First, set up the sample data files:

### Run the Complete Workflow

Here's a complete workflow demonstrating several file types:

```bash
# 1. Start DuckDB session
duckdb

# 2. Create a table from a Parquet file
CREATE TABLE products AS SELECT * FROM 'products.parquet';

FROM products;

# 3. Create a table from a CSV file
CREATE TABLE orders (
    order_id INT,
    product_id INT,
    quantity INT,
    order_date DATE
);

COPY orders FROM 'orders.csv';

FROM orders;

# 4. Join data from both sources
SELECT 
    o.order_id,
    p.name,
    o.quantity,
    o.order_date
FROM orders o
JOIN products p ON o.product_id = p.id
LIMIT 10;

# 5. Read metadata from text files
SELECT content FROM read_text('product_notes.txt');

# 6. Create a combined view
CREATE TABLE import_log AS 
SELECT size, parse_path(filename), content 
FROM read_text('product_notes.txt');

FROM import_log;
```

---

## Key Takeaways

1. **Elegance**: DuckDB automatically infers file formats and schemas, reducing boilerplate
2. **Flexibility**: Custom schema definitions are available when auto-inference isn't enough
3. **Efficiency**: Parquet files are the gold standard for performance
4. **Persistence**: Use `CREATE TABLE` to persist imported data for repeated queries
5. **Simplicity**: DuckDB makes it effortless to combine data from multiple file formats

## Useful DuckDB Commands

View all tables in your session:

```bash
SHOW TABLES;
```

Describe a table schema:

```bash
DESCRIBE table_name;
```

Export a DuckDB table to Parquet:

```bash
COPY table_name TO 'output.parquet';
```

Export a DuckDB table to CSV:

```bash
COPY table_name TO 'output.csv';
```

## Next Steps

- Explore reading from remote sources (S3, URLs) - see "Read and Import From File Locations"
- Learn about reading from databases (Postgres, SQLite) - see "Read and Import From a Database"
- Discover next-gen formats (Vortex, Lance) for advanced use cases

---

## Resources

- [DuckDB Official Documentation](https://duckdb.org/docs/)
- [DuckDB CSV Tips & Tricks](https://duckdb.org/docs/data/csv)
- [DuckDB JSON Documentation](https://duckdb.org/docs/data/json)
- Original article: [DuckDB Basics: Importing Data](https://thefulldatastack.substack.com/p/duckdb-basics-importing-data)
