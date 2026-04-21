# ShopStream Spark Demo

A local PySpark environment running on Docker, with a Jupyter notebook server for interactive development.

## Browser Links

| Service | URL | Purpose |
|---|---|---|
| Spark Master UI | http://localhost:8080 | Monitor workers, jobs, cluster status |
| Jupyter Notebook | http://localhost:8888 | Write and run PySpark code |
| Spark App UI | http://localhost:4040 | Live job/stage/task details (only active while a Spark app is running) |

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
