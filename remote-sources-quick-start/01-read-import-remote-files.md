# DuckDB: Read and Import From File Locations

A step-by-step tutorial on how to read and import data from remote file locations using DuckDB. This covers S3 buckets, URLs, and public file locations with authentication and credential management.

> **Original Article**: [DuckDB Basics: Importing Data](https://thefulldatastack.substack.com/p/duckdb-basics-importing-data)

## Prerequisites

- DuckDB CLI installed. Get it [here](https://duckdb.org/docs/installation/)
- Basic familiarity with terminal commands
- For S3: AWS credentials (if accessing private buckets)
- For URLs: Internet access to download files

## Installation

If you don't have DuckDB installed yet, install the DuckDB CLI using the following curl command:

```bash
curl https://install.duckdb.org | sh
```

After installation completes, verify the installation by checking the DuckDB version:

```bash
duckdb --version
```

---

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

## 1. Configuration and Authentication

If you're going to read data from a storage bucket like S3, you'll need to provide credentials unless the storage bucket is public. DuckDB offers a built-in secrets manager that makes credential management easier than traditional environment variables or `.env` files.

### Creating Temporary Credentials

Create temporary credentials that exist only for the current DuckDB session:

```bash
-- open an in memory instance of duckdb
duckdb

-- create temp credentials
CREATE OR REPLACE SECRET secret (
    TYPE s3,
    PROVIDER config,
    KEY_ID 'AKIAIOSFODNN7EXAMPLE',
    SECRET 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
    REGION 'us-east-1'
);
```

View the secrets you've created:

```bash
-- read created secrets table
FROM duckdb_secrets();
```

### Creating Persistent Credentials

For credentials that persist across DuckDB sessions, create a persistent secret:

```bash
-- create persistent credentials
CREATE PERSISTENT SECRET my_persistent_secret (
    TYPE s3,
    PROVIDER config,
    KEY_ID 'AKIAIOSFODNN7EXAMPLE',
    SECRET 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
    REGION 'us-east-1'
);
```

By default, persistent secrets are stored unencrypted in the `~/.duckdb/stored_secrets` directory. You can change this directory if needed:

```bash
-- change the secrets directory
SET secret_directory = 'path/to/my_secrets_dir';
```

### Removing Persistent Credentials

Delete an in-memory secret:

```bash
-- drop an in-memory secret
DROP SECRET IF EXISTS my_secret;
```

Delete a persistent secret whenever you need to:

```bash
-- drop persistent secret
DROP PERSISTENT SECRET my_persistent_secret;
```

---

## 2. Reading From an S3 Bucket

Once you've set up credentials for your S3 bucket, you can pass the S3 URI directly into a SELECT statement. DuckDB automatically infers the file format and handles credential management behind the scenes.

### Public S3 Files

For public S3 files, you don't need to set up credentials. Here's an example using a public file from MotherDuck's S3:

```bash
-- read from public S3 bucket
SELECT * FROM 's3://motherduck-demo/pypi.small.parquet' LIMIT 10;
```

### Reading Multiple Files with Wildcards

If you have multiple Parquet files in an S3 bucket, you can use a wildcard to read them all together:

```bash
-- read all parquet files from S3
SELECT * FROM 's3://s3-public-test-data/*.parquet' LIMIT 10;
```

### Using Credentials with S3

For private S3 buckets, ensure your credentials are set up first (see Configuration and Authentication section above), then use the S3 URI:

```bash
-- assuming credentials are already configured
SELECT * FROM 's3://my-private-bucket/data.parquet' LIMIT 10;
```

---

## 3. Creating Tables From S3 Data

Reading data from S3 is convenient, but creating a local DuckDB table reduces S3 read costs and improves query performance. You can create a new table or insert into an existing table.

### Creating a New Table from S3

```bash
-- create DuckDB table using S3 file
CREATE TABLE pypi AS SELECT * FROM 's3://motherduck-demo/pypi.small.parquet';

-- show all tables in DuckDB database
SHOW TABLES;

-- view the newly created table
FROM pypi LIMIT 10;
```

### Inserting S3 Data into an Existing Table

The process for creating a new table or inserting data is essentially the same across all remote data sources:

```bash
-- insert S3 data into existing table
INSERT INTO existing_table SELECT * FROM 's3://bucket-name/file.parquet';
```

---

## 4. Reading Files From a URL

You can read files directly from URLs without needing to set up credentials or manage S3 buckets. This works great for public datasets and files hosted on web servers.

### Reading CSV from a URL

DuckDB can read CSV files directly from domain URLs:

```bash
-- read CSV file from a URL
SELECT * FROM 'https://blobs.duckdb.org/data/penguins.csv' LIMIT 10;
```

### Reading JSON from GitHub

You can also read JSON files from public GitHub repositories using the raw file URL:

```bash
-- read JSON file from GitHub raw content
SELECT * FROM 'https://raw.githubusercontent.com/nhemerson/streamlit-cloud-run-demo/refs/heads/main/data/streaming_data.json' LIMIT 10;
```

### Reading Other File Formats from URLs

The same pattern works for any file format DuckDB supports (Parquet, JSON, CSV, etc.). Simply pass the full URL:

```bash
-- read parquet file from URL
SELECT * FROM 'https://example.com/data/file.parquet' LIMIT 10;
```

---

## 5. Creating Tables From URL Data

Just like with S3 data, you can create persistent DuckDB tables from remote URLs:

```bash
-- create table from CSV URL
CREATE TABLE penguins AS SELECT * FROM 'https://blobs.duckdb.org/data/penguins.csv';

-- query the new table
FROM penguins LIMIT 10;

-- create table from GitHub JSON URL
CREATE TABLE streaming_data AS SELECT * FROM 'https://raw.githubusercontent.com/nhemerson/streamlit-cloud-run-demo/refs/heads/main/data/streaming_data.json';

FROM streaming_data LIMIT 10;
```

---

## 6. Use Case: Multi-Source Data Integration

One of DuckDB's strengths is its ability to act as a glue layer for diverse data sources. Consider this scenario:

You have:
- A JSON file being continuously updated and stored in S3
- A CSV dataset exported from another system and available at a URL
- Both datasets need to be joined and analyzed locally

DuckDB makes this seamless:

```bash
-- create table from S3 JSON
CREATE TABLE s3_data AS SELECT * FROM 's3://my-bucket/streaming_data.json';

-- create table from URL CSV
CREATE TABLE url_data AS SELECT * FROM 'https://example.com/exports/data.csv';

-- join and analyze both sources
SELECT 
    s3_data.id,
    s3_data.timestamp,
    url_data.value
FROM s3_data
JOIN url_data ON s3_data.id = url_data.id
LIMIT 10;

-- optionally export to a local parquet file for later use
COPY (
    SELECT 
        s3_data.id,
        s3_data.timestamp,
        url_data.value
    FROM s3_data
    JOIN url_data ON s3_data.id = url_data.id
) TO 'combined_data.parquet';
```

---

## Complete Workflow Example

Here's a complete workflow demonstrating reading from multiple remote sources:

```bash
# 1. Start DuckDB session
duckdb

# 2. Set up S3 credentials (if using private buckets)
CREATE PERSISTENT SECRET aws_creds (
    TYPE s3,
    PROVIDER config,
    KEY_ID 'your_key_id',
    SECRET 'your_secret_key',
    REGION 'us-east-1'
);

# 3. Create table from public S3 file
CREATE TABLE s3_demo AS SELECT * FROM 's3://motherduck-demo/pypi.small.parquet' LIMIT 100;

FROM s3_demo LIMIT 5;

# 4. Create table from public URL
CREATE TABLE url_demo AS SELECT * FROM 'https://blobs.duckdb.org/data/penguins.csv' LIMIT 100;

FROM url_demo LIMIT 5;

# 5. View all created tables
SHOW TABLES;

# 6. Combine data from both sources
SELECT 
    COUNT(*) as total_rows,
    'S3 Demo' as source
FROM s3_demo
UNION ALL
SELECT 
    COUNT(*) as total_rows,
    'URL Demo' as source
FROM url_demo;

# 7. Export combined results to local parquet
COPY (
    SELECT * FROM s3_demo
    UNION ALL
    SELECT * FROM url_demo
) TO 'combined_remote_data.parquet';
```

---

## Key Takeaways

1. **Simplicity**: DuckDB's secrets manager and automatic format inference make remote data access straightforward
2. **No Boilerplate**: Pass S3 URIs or URLs directly into SELECT statements—DuckDB handles the rest
3. **Flexibility**: Works with public S3 files, private buckets with credentials, and URLs without setup
4. **Performance**: Create local tables from remote data to minimize repeated reads and improve query speed
5. **Integration**: Use DuckDB as a glue layer to combine data from multiple remote sources
6. **Elegance**: The same patterns work across S3, URLs, and various file formats

---

## Useful DuckDB Commands

View all tables in your session:

```bash
SHOW TABLES;
```

Describe a table schema:

```bash
DESCRIBE table_name;
```

View your configured secrets:

```bash
FROM duckdb_secrets();
```

Export a DuckDB table to Parquet:

```bash
COPY table_name TO 'output.parquet';
```

Export a DuckDB table to CSV:

```bash
COPY table_name TO 'output.csv';
```

---

## Next Steps

- Learn about reading from databases (Postgres, SQLite, DuckLake) - see "Read and Import From a Database"
- Explore reading from local files - see "Read and Import Local Files"
- Discover next-gen formats (Vortex, Lance) for advanced use cases

---

## Resources

- [DuckDB Official Documentation](https://duckdb.org/docs/)
- [DuckDB S3 Documentation](https://duckdb.org/docs/extensions/httpfs)
- [DuckDB Secrets Documentation](https://duckdb.org/docs/sql/statements/create_secret)
- Original article: [DuckDB Basics: Importing Data](https://thefulldatastack.substack.com/p/duckdb-basics-importing-data)
