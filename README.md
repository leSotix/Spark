# ShopStream Spark Demo

A local PySpark environment running on Docker, with a Jupyter notebook server for interactive development.

## Stack

| Service | Image | Port |
|---|---|---|
| Spark master | `bitnami/spark:3.5.1` | `8080` (UI), `7077` (cluster) |
| Spark worker(s) | `bitnami/spark:3.5.1` | scalable via `--scale` |
| Jupyter | `jupyter/pyspark-notebook:spark-3.5.1` | `8888` |

## File layout

```
Spark/
├── data/
│   ├── raw/
│   │   ├── events.csv      # clickstream events (page views, add-to-cart, purchases)
│   │   ├── sessions.csv    # browser session metadata (device, country, referrer)
│   │   ├── products.csv    # product catalogue (name, category, price)
│   │   └── orders.csv      # confirmed transactions
│   └── output/             # written by Spark at runtime (Parquet)
├── notebooks/
│   ├── spark_intro.ipynb        # intro: in-memory employee dataset
│   └── spark_transaction.ipynb  # main demo: clickstream analysis
└── docker-compose.yml
```

The `./data` folder is bind-mounted to `/data` inside **all three containers** (master, worker, Jupyter), so every container reads and writes the same files.

## Getting started

### 1  Set environment variables

```bash
# Defaults already set in .env — override here if needed
export SPARK_WORKER_CORES=2
export SPARK_WORKER_MEMORY=2g
export JUPYTER_TOKEN=group4
```

### 2  Start the cluster

```bash
docker compose up -d
```

Wait ~30 s for the health checks to pass, then open:
- **Jupyter** → `http://localhost:8888` (token: `group4`)
- **Spark master UI** → `http://localhost:8080`

To scale to two workers:

```bash
docker compose up -d --scale spark-worker=2
```

### 3  Open the demo notebook

In Jupyter, navigate to **work → spark_transaction.ipynb** and open it.

---

## Live demo — 3 commands to run in order

Run these three notebook cells (or paste them into a Jupyter cell) to walk through the key Spark concepts:

**Command 1 — show partition counts before and after repartitioning**
```python
print("BEFORE:", events.rdd.getNumPartitions())
events_rp = events.repartition(4)
print("AFTER :", events_rp.rdd.getNumPartitions())
```

**Command 2 — run the grouped aggregation (triggers a shuffle)**
```python
category_stats.show(truncate=False)
```
> While this runs, switch to `http://localhost:8080` → the active application → **Stages**.  
> You will see an **Exchange** stage — that is the shuffle, where data crosses executor boundaries.

**Command 3 — inspect the physical query plan**
```python
category_stats.explain(mode="formatted")
```
> Point out `BroadcastHashJoin` (products — no shuffle) vs `HashAggregate + Exchange` (groupBy — shuffle).

---

## Notebooks

### [spark_intro.ipynb](notebooks/spark_intro.ipynb)

Introduces core PySpark DataFrame operations against a small in-memory employee dataset (6 rows, three departments).

- **Connect** — builds a `SparkSession` pointing at `spark://spark-master:7077`
- **Filter** — selects only Engineering employees
- **Aggregate** — headcount and average salary per department via `groupBy`
- **Derive** — adds an `above`/`below` column comparing each salary to a fixed median threshold

### [spark_transaction.ipynb](notebooks/spark_transaction.ipynb)

End-to-end clickstream analysis using the ShopStream dataset.

- **Load** — reads four CSVs from `/data/raw/` into Spark DataFrames
- **Schema** — `printSchema()` and row counts on all four tables
- **Partitions** — `getNumPartitions()` before and after `repartition(4)`
- **Join** — events enriched with session and product context; `F.broadcast()` used for the small products table
- **Aggregate** — revenue, event counts, and conversion rate per product category (shuffle)
- **explain()** — physical plan showing `BroadcastHashJoin`, `Exchange`, and `HashAggregate` nodes
- **Write** — results saved to `/data/output/` as Parquet (columnar, compressed, splittable)

### Key concept

> Spark handles **processing** — distributing computation across workers.  
> **Replication and durability** live in the storage layer beneath it (HDFS, S3, GCS, etc.).  
> In this demo the shared `/data` bind-mount plays that role.
