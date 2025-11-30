# Core Databricks <img src="https://cdn.simpleicons.org/databricks/F60022" width="26" style="vertical-align:middle; margin-left:4px;" />

## 1️⃣ What is Databricks?

**Simple interview answer:**

> Databricks is a cloud platform built around Apache Spark that provides managed clusters, notebooks, jobs, and a lakehouse architecture (using Delta Lake and Unity Catalog). It lets data engineers and analysts develop, run, and schedule Spark-based data pipelines on top of cloud storage like ADLS or S3.
> 

Key points to mention:

- Managed Spark → you don’t install/manage Spark yourself.
- Supports **multiple languages**: Python (PySpark), SQL, Scala, R.
- Strong focus on **Delta Lake (DL)** and **Unity Catalog (UC)**.
- Integrated with **cloud storage** (ADLS Gen2 in your case) and tools like Power BI.

For **Azure**:

“Azure Databricks” is a first-class service integrated with:

- ADLS Gen2 (for data lake)
- ADF / Synapse pipelines (orchestrating jobs)
- AAD (for identity & permissions).

---

## 2️⃣ Core Databricks objects: mental map

In the workspace you usually see:

- **Clusters** – compute on which Spark runs.
- **Notebooks** – interactive development (PySpark/SQL/etc.).
- **Jobs** – scheduled / production runs of notebooks or workflows.
- **Repos** – Git integration.
- **Unity Catalog objects** – catalogs, schemas, tables, volumes.
- **DBFS** – Databricks File System (abstraction layer over storage).

You should be able to explain:

> “We develop and test in notebooks on an all-purpose cluster, commit to Git, and then orchestrate production pipelines as Databricks Jobs running on job clusters.”
> 

---

## 3️⃣ Clusters – where Spark actually runs

In Databricks, a **cluster** is:

> A set of VMs with a Spark driver and executors, configured with a Databricks Runtime (DBR), libraries, and settings.
> 

Important concepts:

### 3.1 Cluster types

For junior interviews you just need:

- **All-purpose (interactive) clusters**
    - You attach notebooks.
    - Great for development, ad-hoc analysis.
- **Job clusters**
    - Created automatically for a specific job run, then terminated.
    - Good for production ETL; clean + isolated.

How to explain:

> “I use an interactive cluster for development and testing notebooks. For production pipelines, we configure jobs that use job clusters so each run is clean and scalable.”
> 

### 3.2 Key cluster settings

- **Databricks Runtime (DBR)** version:
    - E.g., 14.x, 15.x, with or without ML.
    - Controls Spark version + libraries.
- **Node type & size**:
    - CPU / RAM per node.
- **Number of workers** / autoscaling:
    - Min / max workers.
    - Autotermination after idle time.
- **Single-node vs multi-node**:
    - Single-node for small dev/testing.
    - Multi-node for big data pipelines.

Note:

- Each cluster has driver + executors.
- More workers → more executors → more parallelism.

---

## 4️⃣ Notebooks – how you actually write PySpark in Databricks

### 4.1 Languages & magic commands

Databricks notebook cells can have:

- `%python` – PySpark / Python
- `%sql` – Spark SQL
- `%scala`, `%r`, `%sh` etc.

Often the default is Python, so you just write PySpark code directly.

**Important**: the `spark` object (SparkSession) is already created for you:

```python
# In Databricks notebook (Python cell)
spark

# You can start using it directly
df = spark.range(10)
df.show()

```

You **don’t** usually write `SparkSession.builder...` in Databricks notebooks.

### 4.2 Display vs show

In Databricks:

```python
df.show()        # prints text table in the cell output
display(df)      # Databricks special: nice UI table, charts, filters

```

`display(df)` is not a generic Spark function; it’s a Databricks notebook feature.

---

## 5️⃣ Storage in Databricks: DBFS, ADLS, and paths

### 5.1 DBFS – Databricks File System

Logical file system that lets you access data using paths like:

- `dbfs:/mnt/...`
- `/dbfs/mnt/...` (from some contexts)

DBFS can have:

- Local storage (cluster-scoped)
- Mount points to cloud storage (legacy pattern)
- Unity Catalog **volumes** (modern pattern).

### 5.2 Accessing ADLS Gen2

In your planned architecture:

> ADF → ADLS Gen2 → Databricks
> 

Spark in Databricks reads/writes to ADLS using:

```python
df = spark.read.parquet("abfss://container@account.dfs.core.windows.net/path/...")

```

or through `dbfs:/mnt/...` if mounted.

Modern best practice is:

- Use **service principals** / managed identities.
- Configure **Unity Catalog volumes** or secure external locations.
- Read/write using UC tables (`catalog.schema.table`) instead of raw paths as much as possible.

This connects with **Delta + UC** (next lessons).

---

## 6️⃣ Jobs – turning notebooks into production pipelines

Interactive notebook = good for development.

But for production, you need **jobs**:

> A Databricks Job is a scheduled or triggered run of one or more tasks (notebook, Python script, SQL, etc.) on a cluster.
> 

Key things:

- A Job:
    - has one or more **tasks** (DAG of tasks),
    - each task runs on a **job cluster** or a shared cluster,
    - can be scheduled (cron-style or periodic),
    - has alerts & retry policies.

Example narrative you’d say in interview:

> “We develop ETL logic in Databricks notebooks on an interactive cluster. Once stable, we configure a Job that runs the notebook on a job cluster, scheduled daily after files land in ADLS (or triggered by ADF). The Job writes results as Delta tables in a Unity Catalog schema for consumption by analysts and Power BI.”
> 

---

## 7️⃣ Libraries & environment in Databricks

You’ll often need extra Python libs (e.g., `requests`, `pydantic`, `great_expectations`).

Databricks supports:

- **Cluster-level libraries**:
    - Installed via UI (Maven, PyPI, wheel, etc.).
    - Available to all notebooks attached to that cluster.
- **Notebook-scoped / `%pip`**:
    - Inside a notebook cell:
        
        ```python
        %pip install some-package
        
        ```
        
    - Scoped to that notebook’s session.

Important for interviews:

> “In Databricks, we typically attach libraries at the cluster level for shared ETL dependencies. For experimentation, we can also use %pip install in notebooks for quick tests.”
> 

---

## 8️⃣ Configuration & Spark settings in Databricks

You can set Spark configs at:

- **Cluster level** (in cluster advanced options)
- **Session level** via `spark.conf.set(...)`

Example:

```python
spark.conf.set("spark.sql.shuffle.partitions", "200")

```

This ties back to performance (Lesson 8):

- For big joins/aggregations, you may adjust `spark.sql.shuffle.partitions`.
- Also for broadcast threshold, etc. (high-level mention is enough for junior).

You can also read configs:

```python
spark.conf.get("spark.sql.shuffle.partitions")

```

---

## 9️⃣ Monitoring & debugging in Databricks

Databricks gives you:

- **Spark UI** (per cluster / per job run) – see:
    - Jobs, stages, tasks
    - Shuffles
    - Skewed tasks
- **Job run logs**:
    - Driver logs, stdout/stderr
- **Cluster metrics**:
    - CPU, memory, disk, etc.

Interview-friendly sentence:

> “If a job is slow or failing, I go to the Job run view, open the Spark UI, and look at which stages are taking time, whether there are many shuffles or skewed tasks. Then I consider options like broadcast joins, adjusting partitions, or filtering data earlier.”
> 

---

## 🔁 Interview Q&A (Databricks basics)

---

**Q1. What is Databricks and how is it related to Spark?**

> Databricks is a managed platform built around Apache Spark. It provides managed Spark clusters, notebooks, jobs, and governance features like Delta Lake and Unity Catalog. Under the hood, it runs Spark, but it handles cluster management, scaling, and integration with cloud storage for you.
> 

---

**Q2. What is a Databricks cluster?**

> A cluster is a set of virtual machines with a Spark driver and executors that run your code. It has a Databricks Runtime version, node types, and configuration. Notebooks and jobs run on clusters.
> 

---

**Q3. What’s the difference between an all-purpose cluster and a job cluster?**

> An all-purpose (interactive) cluster is used for development and ad-hoc work; you attach notebooks and run commands interactively. A job cluster is created automatically by a job for a specific run and then terminated; it’s typically used for production pipelines for isolation and cost control.
> 

---

**Q4. How do you schedule a Spark pipeline in Databricks?**

> I develop the logic in a notebook, then create a Databricks Job that runs that notebook on a job cluster. I configure the schedule (e.g., daily at a certain time or triggered by ADF), set retries and alerts, and the job takes care of running the pipeline.
> 

---

**Q5. What is DBFS?**

> DBFS (Databricks File System) is an abstraction that lets you interact with storage using a familiar file-like path, such as dbfs:/.... It can sit on top of cloud storage like ADLS, and you can use it to store code, small reference files, and access data in mounted or external locations.
> 

---

**Q6. How do you access data stored in ADLS Gen2 from Databricks?**

> We configure authentication (like a service principal or managed identity) and either define external locations/volumes in Unity Catalog or directly read from abfss:// paths. In code, we just use spark.read / spark.write with those paths or use registered tables in Unity Catalog.
> 

---

**Q7. How can you adjust Spark behavior for joins and aggregations in Databricks?**

> At the cluster level I can set configs like spark.sql.shuffle.partitions as defaults. In a specific notebook or job, I can override them using spark.conf.set. For example, I can lower the shuffle partitions when working with small datasets, or tune it based on performance seen in the Spark UI.
> 

**Q8: How do you debug a slow job in Databricks?**

> I open the job run, then the Spark UI. I look at which stages are taking most of the time and if they involve large shuffles. In the stages’ task view I check for skewed tasks or too many small tasks. Based on that, I may add a broadcast join for small dimension tables, adjust shuffle partitions, or filter data earlier in the pipeline.
> 

# Delta Lake **(DL) + Lakehouse Flow**.

I’ll cover:

1. What Delta Lake is (in simple & interview language)
2. Delta vs Parquet & why it matters for banks
3. Bronze / Silver / Gold with Delta (lakehouse flow)
4. Core Delta operations:
    - Create / read / write Delta tables
    - UPDATE / DELETE
    - MERGE (incremental loads, reversals)
    - Time travel
5. Interview questions + model answers

---

## 1️⃣ What is Delta Lake?

> Delta Lake is a storage layer on top of Parquet that brings ACID transactions, schema enforcement, time travel, and efficient metadata to data lakes. In Databricks it’s the default format for building a lakehouse.
> 

Break it down:

- **Storage layer on top of Parquet**
    - Data is still stored as **Parquet files**.
    - Delta adds a **transaction log** (`_delta_log`), like a commit log/Git history.
- **ACID transactions**
    - **A**tomic, **C**onsistent, **I**solated, **D**urable.
    - Multiple writers can safely write; readers see consistent snapshots.
    - Critical in **banking** (no partial writes, no corrupted tables).
- **Schema enforcement & evolution**
    - Prevents bad data (wrong types, missing columns) from silently corrupting the table.
    - Allows controlled schema changes when needed.
- **Time travel**
    - You can query the table **as of an older version or timestamp**.
    - Super useful for **audit, debugging, and regulatory checks**.
- **Batch + streaming on the same table**
    - You can write to a Delta table from batch or streaming jobs and read it consistently.

So: **Delta Lake = Parquet + serious reliability + features** → that’s what makes a **lakehouse** possible.

---

## 2️⃣ Delta vs Parquet – why not just Parquet?

You already know Parquet is:

- Columnar
- Compressed
- Good for analytics

But **Parquet alone has problems** in a data lake:

- No ACID: concurrent writes can corrupt data.
- No built-in transaction history.
- Hard to do UPDATE/DELETE/UPSERT safely.
- Schema drift can silently break things.

**Delta fixes this**:

| Feature | Plain Parquet | Delta Lake |
| --- | --- | --- |
| ACID transactions | ❌ No | ✅ Yes |
| UPDATE / DELETE / MERGE | ❌ Very hard (manual rewrite) | ✅ Native SQL support |
| Time travel | ❌ No | ✅ Yes (version / timestamp) |
| Schema enforcement | ❌ Weak | ✅ Strong (fail on incompatible schema) |
| Metadata scaling | ❌ Weak for many files | ✅ Optimized transaction log & stats |

**Interview way to say it:**

> Parquet is great as a file format, but on its own it doesn’t give you ACID guarantees or easy updates and deletes. Delta Lake uses Parquet underneath but adds a transaction log, so we can do safe upserts, handle schema changes, and time travel, which is important in financial data pipelines.
> 

---

## 3️⃣ Lakehouse & Medallion Architecture (Bronze / Silver / Gold)

This is where **Delta + your project** become real.

### 3.1 Medallion layers

**Bronze** – *Raw data*

- Direct from source systems (T24, logs, flat files, API dumps).
- Minimal transformations (maybe only add ingestion time).
- Example: `bronze.transactions_raw`, `bronze.customers_raw`.

**Silver** – *Clean, conformed data*

- Data is cleaned, typed, deduplicated, standardised.
- Joins with reference tables (customers, FX, branches, KYC flags).
- Example: `silver.transactions_enriched`, all amounts converted to **USD/EGP**, reversals applied.

**Gold** – *Business-ready marts*

- Aggregated tables for reporting / analytics.
- Used by Power BI, risk models, management dashboards.
- Example: `gold.daily_branch_fx_kpis`, `gold.segment_performance`.

**In a Databricks lakehouse**, each of these is implemented as **Delta tables**.

A possible naming under UC later:

- `bank_lake.bronze.transactions_raw`
- `bank_lake.silver.transactions_enriched`
- `bank_lake.gold.daily_kpi`

We’ll wire this properly in Lesson 11 with Unity Catalog.

---

## 4️⃣ Core Delta operations (with code)

I’ll use both **SQL** (Databricks SQL/Notebook `%sql`) and **PySpark**.

Assume your data is in ADLS paths like:

- `abfss://bank@…/bronze/transactions/`
- `abfss://bank@…/silver/transactions_enriched/`
- etc.

### 4.1 Creating & writing Delta tables

### Option A – From PySpark DataFrame

```python
# df_bronze: DataFrame with raw transactions
df_bronze.write.format("delta") \
    .mode("overwrite") \
    .save("abfss://bank/raw/bronze/transactions/")

# Read it back
df_bronze_delta = spark.read.format("delta") \
    .load("abfss://bank/raw/bronze/transactions/")

```

### Option B – As managed tables (using UC catalog)

In a Databricks SQL or `%sql` cell:

```sql
CREATE TABLE bank_lake.bronze.transactions_raw
USING DELTA
LOCATION 'abfss://bank/raw/bronze/transactions/';

```

Then you can:

```sql
SELECT * FROM bank_lake.bronze.transactions_raw;

```

**Important idea:**

- **Path-based** access → `spark.read.format("delta").load(path)`
- **Table-based** (recommended with UC) → `SELECT * FROM catalog.schema.table`

---

### 4.2 UPDATE & DELETE with Delta

Banking / fintech **loves** this, because data changes (corrections, status updates).

Example: mark suspicious transactions:

```sql
UPDATE bank_lake.silver.transactions_enriched
SET is_suspicious = true
WHERE amount_usd > 100000 AND risk_score > 0.8;

```

Delete test data / bad loads:

```sql
DELETE FROM bank_lake.bronze.transactions_raw
WHERE ingestion_date = '2025-11-01' AND source_system = 'TEST';

```

In **plain Parquet** this would be a nightmare (manual file rewrite). With Delta, it’s natural.

---

### 4.3 MERGE – incremental loads & UPSERT (very important)

This is one of the **most important Delta features** for interviews and real projects.

Scenario in your project:

- Every day you ingest **new & updated transactions** from source into a **staging Delta table**:
    - `staging.daily_transactions` (for that day).
- You want to **upsert** them into a **Silver Delta table**:
    - `silver.transactions_enriched`.

We’ll assume each transaction has:

- `txn_id` (unique ID)
- `t24_business_date`
- `amount`, `currency`, `customer_id`, etc.
- Maybe a `status` (e.g., `NORMAL`, `REVERSED`, `CANCELLED`).

### Standard MERGE pattern (upsert)

```sql
MERGE INTO bank_lake.silver.transactions_enriched AS target
USING bank_lake.staging.daily_transactions AS source
ON target.txn_id = source.txn_id
WHEN MATCHED THEN
  UPDATE SET
    target.amount           = source.amount,
    target.currency         = source.currency,
    target.t24_business_date= source.t24_business_date,
    target.status           = source.status,
    target.updated_at       = current_timestamp()
WHEN NOT MATCHED THEN
  INSERT (
    txn_id,
    customer_id,
    branch,
    currency,
    amount,
    t24_business_date,
    status,
    created_at
  )
  VALUES (
    source.txn_id,
    source.customer_id,
    source.branch,
    source.currency,
    source.amount,
    source.t24_business_date,
    source.status,
    current_timestamp()
  );

```

This does:

- If transaction already exists → UPDATE it.
- If it’s new → INSERT it.

This is **CDC (change data capture)** style behavior.

### Handling reversals

If reversals come as separate rows with a flag, e.g., `status = 'REVERSAL'`, there are two common patterns:

1. **Update existing row** to mark it reversed (set `is_reversed = true`, `reversal_date`, maybe set amount 0).
2. **Insert a second row** representing the reversal (with negative amount), then aggregate properly.

Both can be implemented via **MERGE** or combination of MERGE + UPDATE. At junior level, it’s enough to say:

> “We use a Delta MERGE to upsert new/changed transactions, and if a reversal comes in, we update the existing record or insert a compensating row, so our Delta table always reflects the latest correct state.”
> 

---

### 4.4 Time travel – query old versions

Delta keeps a **transaction log**, so you can query past versions:

In SQL:

```sql
-- Current version
SELECT * FROM bank_lake.silver.transactions_enriched;

-- Version as of commit number 5
SELECT * FROM bank_lake.silver.transactions_enriched VERSION AS OF 5;

-- As of timestamp
SELECT * FROM bank_lake.silver.transactions_enriched
TIMESTAMP AS OF '2025-11-01T10:00:00';

```

In PySpark:

```python
df_old = spark.read.format("delta") \
    .option("versionAsOf", 5) \
    .table("bank_lake.silver.transactions_enriched")

```

**Use cases in banking:**

- Audit: “show me what the table looked like before yesterday’s load.”
- Debugging: “see data before & after a MERGE that went wrong.”
- Regulatory: “reproduce a report exactly as it was generated on a given date.”

---

### 4.5 Schema enforcement & evolution (high-level)

**Schema enforcement:**

- If new data doesn’t match the table schema, Delta can **fail** the write instead of silently corrupting data.

Example: trying to write string into a numeric column → exception.

**Schema evolution:**

- Sometimes schema legitimately changes (e.g., new column `risk_score`).
- You can allow **adding new columns** with options like:
    
    ```python
    df.write.format("delta") \
      .option("mergeSchema", "true") \
      .mode("append") \
      .save(path)
    
    ```
    

or table-level settings. At junior level just know:

> Delta enforces schema by default but can be configured to allow controlled schema evolution (like adding new columns).
> 

---

### 4.6 VACUUM (just conceptually)

Delta keeps old versions for time travel.

- Over time, older **files** can be removed to save storage.
- `VACUUM table` physically deletes files no longer in the current log (older than retention period).

You don’t need to know all vacuum configs now, just:

> VACUUM is used to clean up old data files that are no longer needed according to the table’s retention policy.
> 

### 4.7 OPTIMIZE & Z-ORDER (performance)

- `OPTIMIZE table`:
    - Compacts many small files into fewer larger files.
    - Helps with the "small files problem" and improves read performance.
- `OPTIMIZE table ZORDER BY (col1, col2)`:
    - Rewrites data so that rows with similar values of col1/col2 are stored close together.
    - Improves data skipping for queries that filter on these columns.

Interview sentence:

> “For large Delta tables, we can run OPTIMIZE with Z-ORDER on frequently filtered columns to reduce small files and improve query performance using better data skipping.”
> 

---

## 5️⃣ Interview Q&A (Delta + Lakehouse)

Here are key questions you should be ready for:

---

**Q1. What is Delta Lake and why do we use it?**

> Delta Lake is a storage layer on top of Parquet that adds ACID transactions, schema enforcement, time travel, and scalable metadata. We use it to make data lakes reliable and support updates, deletes, and upsets directly on lake storage, which is essential for building a lakehouse.
> 

---

**Q2. How is Delta different from plain Parquet?**

> Delta uses Parquet files under the hood but adds a transaction log. That allows ACID guarantees, easy UPDATE/DELETE/MERGE, time travel, and strong schema enforcement. Plain Parquet doesn’t have those features, so concurrent writes or schema changes are harder to manage safely.
> 

---

**Q3. Explain Bronze / Silver / Gold in a Lakehouse.**

> Bronze holds raw ingested data, Silver holds cleaned and conformed data with joins and standardization, and Gold holds business-ready aggregates or data marts. In Databricks we typically implement each layer as Delta tables, so data flows from Bronze to Silver to Gold through Spark jobs.
> 

---

**Q4. How would you implement an incremental load into a Delta table?**

> I’d load the new/changed records into a staging Delta table and then use a MERGE INTO operation to upsert into the target Delta table based on a business key like txn_id. When a match is found, I update the existing row; when there’s no match, I insert a new row. This way the table stays in sync with the source.
> 

---

**Q5. How can Delta Lake help with handling corrections or reversals in banking transactions?**

> Corrections and reversals arrive as updated records in the feed. Using Delta’s MERGE, we can update the corresponding transaction rows in our Silver table or insert compensating entries. Because Delta provides ACID guarantees, we can safely apply these changes and know readers will see a consistent state.
> 

---

**Q6. What is time travel in Delta Lake and why is it useful?**

> Time travel lets you query a Delta table as it existed at a previous version or timestamp. It’s useful for auditing, debugging, and reproducing historical reports exactly as they were at a given time—for example, to see the state of transactions before a bad load or to satisfy regulatory audits in a bank.
> 

---

**Q7. How do you create and query a Delta table in Databricks?**

> From PySpark I can write a DataFrame as Delta using df.write.format("delta").save(path) and read it with spark.read.format("delta").load(path). With Unity Catalog I can also create a table using CREATE TABLE …. USING DELTA LOCATION 'path' and then query it with SELECT ... FROM catalog.schema.table.
> 

---

# Unity Catalog (UC) **& Governance**

I’ll cover:

1. What Unity Catalog is + what problem it solves
2. Object model: **metastore → catalog → schema → table/view/volume**
3. UC vs old Hive metastore
4. Managed vs external tables (high-level, cloud storage)
5. Permissions: users/groups, GRANT/REVOKE
6. Lineage & data discovery
7. Governance scenarios: sensitive KYC/risk data
8. Interview Q&A 

---

## 1️⃣ What is Unity Catalog?

> Unity Catalog is Databricks’ centralized governance layer for data and AI. It provides a unified catalog of all tables, views, and files across workspaces, with fine-grained access control and lineage on top of Delta Lake.
> 

Key ideas:

- **Centralized**: one catalog for **multiple Databricks workspaces** (e.g., dev, test, prod).
- **Unified**: same model for:
    - tables, views, functions
    - files/volumes (non-tabular data)
    - ML models (in more advanced use).
- **Governance**:
    - Access control (`GRANT`/`REVOKE`)
    - Auditing & lineage
    - Data discovery (search, catalogs, schemas)

In plain words: **UC tells you *what data exists*, *who can access it*, and *who is using it*.**

---

## 2️⃣ UC object model: Metastore → Catalog → Schema → Table

Think of UC like a 3-level hierarchy on top of storage:

```
Metastore
  └── Catalog
        └── Schema
              └── Table / View / Function / Volume

```

### 🔹 Metastore

- Top-level container for all governance in a region.
- One metastore per region per org (usually).
- As a junior, you **don’t configure it**, you just use it.

### 🔹 Catalog

- Highest-level logical grouping inside a metastore.
- Often used for:
    - **Environment separation**: `dev`, `test`, `prod`
    - **Domain separation**: `bank_lake`, `ml_lab`, `shared`
- In SQL:
    
    ```sql
    CREATE CATALOG bank_lake;
    
    ```
    

### 🔹 Schema

- Like “schema” in a database (or “database” in old Hive terms).
- Used to group tables logically, e.g.:
    - By **layer**: `bronze`, `silver`, `gold`
    - By **domain**: `core_banking`, `risk`, `crm`
- In SQL:
    
    ```sql
    CREATE SCHEMA bank_lake.bronze;
    CREATE SCHEMA bank_lake.silver;
    CREATE SCHEMA bank_lake.gold;
    
    ```
    

### 🔹 Table / View / Volume

- **Tables**: structured data (Delta/Parquet/CSV/etc.).
- **Views**: saved queries over tables.
- **Volumes**: governed file directories (for non-tabular data; more advanced, but know they exist).

Example table:

```sql
CREATE TABLE bank_lake.silver.transactions_enriched
USING DELTA
LOCATION 'abfss://bank/silver/transactions_enriched/';

```

You then query it as:

```sql
SELECT * FROM bank_lake.silver.transactions_enriched;

```

**Important:**

Without UC you’d just use `database.table`.

With UC you use **three-part name**: `catalog.schema.table`.

---

## 3️⃣ UC vs old Hive metastore

Before UC, Databricks had:

- **Per-workspace Hive metastores**:
    - Each workspace had its own list of tables.
    - Hard to share tables between workspaces.
    - Access control mostly at cluster/storage level.

Unity Catalog improves this:

- **One central metastore** → multiple workspaces share the same catalogs/schemas/tables.
- **Central permissions**:
    - You GRANT on `catalog.schema.table` instead of fiddling with cluster ACLs.
- Better **lineage** + auditing.

In interviews, you can say:

> Previously each Databricks workspace had its own Hive metastore, which made governance and sharing harder. Unity Catalog provides a centralized metastore with consistent permissions and lineage across workspaces, so we can manage access and track usage in one place.
> 

---

## 4️⃣ Managed vs external tables (high-level)

For a junior, you just need a high-level understanding:

### Managed table (UC-managed)

- Databricks/UC controls the **physical location** under a configured storage root.
- When you `DROP TABLE`, the data files can also be removed (under UC control).

Example:

```sql
CREATE TABLE bank_lake.silver.customers_clean
USING DELTA
AS
SELECT ... FROM ...

```

You didn’t specify `LOCATION` → UC chooses a managed path.

### External table

- You specify a **LOCATION** pointing to existing files in ADLS/S3.

Example:

```sql
CREATE TABLE bank_lake.bronze.transactions_raw
USING DELTA
LOCATION 'abfss://bank/raw/bronze/transactions/';

```

- Dropping the table removes the **metadata** from UC, but files may remain (depending on config).

Interview-level sentence:

> In Unity Catalog, managed tables have their storage fully controlled by UC, while external tables point to pre-existing locations in ADLS or S3. In both cases, we use the same catalog.schema.table naming for access and permissions.
> 

---

## 5️⃣ Permissions & access control in UC

This is where UC is most important for **banking / regulated environments**.

### 5.1 Principals & privileges

- **Principals**:
    - Users, groups, service principals.
- **Privileges**:
    - `USE CATALOG`, `USE SCHEMA`
    - `SELECT` – read data
    - `INSERT`, `UPDATE`, `DELETE`
    - `MODIFY` – write data / alter table
    - `CREATE`, `OWNERSHIP`, etc.

### 5.2 Basic GRANT patterns

Example: give analysts read access to Gold tables.

```sql
-- Allow group analysts to use catalog and schema
GRANT USE CATALOG ON CATALOG bank_lake TO `analysts`;
GRANT USE SCHEMA ON SCHEMA bank_lake.gold TO `analysts`;

-- Allow them to read a specific table
GRANT SELECT ON TABLE bank_lake.gold.daily_branch_fx_kpis TO `analysts`;

```

Example: allow data engineers to modify Silver:

```sql
GRANT USE SCHEMA ON SCHEMA bank_lake.silver TO `data_engineers`;
GRANT SELECT, MODIFY ON ALL TABLES IN SCHEMA bank_lake.silver TO `data_engineers`;

```

You might not write these every day as a junior, but **you should understand what they do**.

**Interview way to explain:**

> In Unity Catalog we grant privileges like SELECT or MODIFY on catalogs, schemas, and tables to users or groups. For example, analysts get SELECT on Gold tables, while data engineers get MODIFY on Bronze and Silver Delta tables to manage ETL.
> 

---

## 6️⃣ Lineage & data discovery

Unity Catalog also tracks **lineage**:

- Which tables/views are **read/written** by:
    - Notebooks
    - Jobs
    - SQL queries
    - Dashboards (if integrated)

This gives:

- Upstream/downstream view:
    - “This Gold KPI table depends on these Silver tables, which depend on these Bronze ingestions.”
- Impact analysis:
    - “If I change this Silver table schema, which reports/notebooks might break?”
- Audit / governance:
    - “Who is using this sensitive KYC table?”

In Databricks UI:

- You open a table in UC.
- You see a **Lineage** tab showing the graph: jobs, notebooks, source tables.

In interviews:

> “One benefit of Unity Catalog is automatic column- and table-level lineage. It helps us understand how Gold tables depend on Silver and Bronze, and which notebooks and BI dashboards use sensitive data, which is important in a bank for governance and impact analysis.”
> 

---

## 7️⃣ Governance scenarios (KYC, risk, etc.)

At high level, show that you **see the governance angle**:

- Sensitive tables:
    - KYC info (`customer_full_name`, `ID_number`, `address`).
    - Risk scores, blacklist flags.

UC allows:

- Restrict access to those tables to **small groups**.
- Expose safe versions as **views** to wider teams:
    
    ```sql
    CREATE VIEW bank_lake_prod.gold.customer_summary AS
    SELECT
      c.customer_id,
      c.segment,
      c.country,
      -- Mask detailed KYC fields
      '***' AS full_name,
      -- metrics from transactions
      SUM(t.amount_usd) AS total_spend_usd
    FROM bank_lake_prod.silver.customers_clean c
    JOIN bank_lake_prod.silver.transactions_enriched t
      ON c.customer_id = t.customer_id
    GROUP BY c.customer_id, c.segment, c.country;
    
    ```
    

Then:

- Risk/KYC group has `SELECT` on `silver.customers_clean`.
- Analysts have `SELECT` only on `gold.customer_summary`.

That’s **real** governance.

---

## 8️⃣  Interview Q&A (UC + governance)

---

**Q1. What is Unity Catalog and why is it important?**

> Unity Catalog is Databricks’ centralized governance layer. It provides a single metastore with catalogs, schemas, and tables across workspaces, plus fine-grained access control and lineage. It’s important because it makes it easier to manage permissions and track data usage in a lakehouse, especially in regulated environments like banking.
> 

---

**Q2. Explain the Unity Catalog object hierarchy.**

> At the top we have a metastore, which contains catalogs. Each catalog contains schemas, and each schema contains tables, views, and volumes. We reference objects using a three-part name: catalog.schema.table.
> 

---

**Q3. How would you structure Unity Catalog for a Bronze/Silver/Gold lakehouse?**

> I’d create a catalog for the environment, for example bank_lake_prod, and within it schemas like bronze, silver, gold, and maybe ref for reference data. Then I’d create Delta tables in each schema, such as bank_lake_prod.bronze.transactions_raw, bank_lake_prod.silver.transactions_enriched, and bank_lake_prod.gold.daily_branch_fx_kpis.
> 

---

**Q4. How do you control who can access which tables in UC?**

> We grant privileges like USE CATALOG, USE SCHEMA, SELECT, and MODIFY to users or groups. For example, data engineers might have MODIFY on Bronze and Silver tables, while analysts only get SELECT on Gold tables. Sensitive KYC tables are restricted to specific groups, with masked views exposed to others.
> 

---

**Q5. What is data lineage in Unity Catalog and why is it useful?**

> Lineage shows how data flows between tables, views, notebooks, and jobs. It’s useful for impact analysis and governance; for example, we can see which Gold tables depend on which Silver/Bronze tables, and which dashboards use a sensitive KYC table. That helps when changing schemas or auditing data usage.
> 

---

**Q6. How does Unity Catalog relate to Delta Lake and the lakehouse?**

> Delta Lake provides reliable storage and ACID tables on the data lake, while Unity Catalog provides governance and a logical organization over those Delta tables. Together they form the Databricks lakehouse: Delta for storage and transactions, UC for catalogs/schemas/tables, permissions, and lineage.
> 

---
