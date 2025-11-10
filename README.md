# 🧠 Hadoop + Spark + Jupyter Cluster (Docker Compose)

Este repositorio contén un **entorno completo de Big Data** baseado en contedores Docker, que integra:
- **Hadoop (HDFS + YARN)**  
- **Spark** (con *History Server* para monitorizar aplicacións)
- **JupyterLab** como interface interactiva de análise e desenvolvemento en PySpark

---

## 📂 Estrutura do repositorio

```
.
├── conf/                     # Configuración de Hadoop (XML)
│   ├── core-site.xml
│   ├── hdfs-site.xml
│   ├── mapred-site.xml
│   └── yarn-site.xml
│
├── jupyter/                  # Imaxe e dependencias do contedor Jupyter
│   ├── Dockerfile            # Constrúe o notebook con Spark e dependencias Python
│   └── requirements.txt      # Lista de paquetes Python instalados
│
├── work/                     # Espazo compartido de traballo (ligado ao notebook)
│                             # Aquí podes gardar os notebooks e datos
│
├── start-dfs.sh              # Script de arranque do NameNode (formateo e inicio do HDFS)
├── start-history.sh          # Script de arranque do Spark History Server
│
├── Dockerfile                # Imaxe base con Hadoop + Spark
├── docker-compose.yml        # Orquestración dos contedores
└── README.md                 # Este documento
```

---

## 🚀 Despregue do clúster

### 1️⃣ Construír as imaxes

Constrúe primeiro a imaxe base con Hadoop + Spark:

```bash
docker build -t adbgonzalez/spark:test-lean -f Dockerfile .
```

Logo a imaxe de Jupyter:

```bash
docker build -t adbgonzalez/spark-notebook:test-lean -f jupyter/Dockerfile jupyter
```

---

### 2️⃣ Arrancar o clúster

```bash
docker compose up -d
```

Isto lanza os seguintes servizos:

| Servizo | Rol | Portos principais |
|----------|-----|------------------|
| `namenode` | HDFS NameNode | 9870 (UI), 9000 (servizo) |
| `datanode` | HDFS DataNode | 9864 |
| `resourcemanager` | YARN ResourceManager | 8088 |
| `nodemanager` | YARN NodeManager | 8042 |
| `spark-history` | Spark History Server | 18080 |
| `notebook` | JupyterLab con PySpark | 8888 |

---

### 3️⃣ Acceso ás interfaces web

| Servizo | URL | Descrición |
|----------|-----|------------|
| **JupyterLab** | [http://localhost:8888](http://localhost:8888) | Entorno de traballo en Python / PySpark |
| **HDFS NameNode** | [http://localhost:9870](http://localhost:9870) | Navegador de ficheiros de HDFS |
| **YARN ResourceManager** | [http://localhost:8088](http://localhost:8088) | Seguimento de tarefas YARN |
| **Spark History Server** | [http://localhost:18080](http://localhost:18080) | Histórico de aplicacións Spark |

---

## 🧩 Uso básico

### 📘 1. Proba rápida de Spark dende o notebook

Abre [http://localhost:8888](http://localhost:8888) e crea un novo Notebook Python con este código:

```python
import findspark
findspark.init("/opt/spark")

from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("Test-YARN")
    .master("yarn")
    .config("spark.eventLog.enabled", "true")
    .config("spark.eventLog.dir", "hdfs://namenode:9000/spark-logs")
    .getOrCreate()
)

spark.range(1, 1000000).selectExpr("sum(id)").show()
```

O resultado deberías poder velo tamén no **Spark History Server** (http://localhost:18080).

---

## ⚙️ Directorios e volumes

Os volumes de datos de HDFS están xestionados automaticamente por Docker Compose:

| Volume | Ruta dentro do contedor | Contido |
|---------|--------------------------|----------|
| `namenode_data` | `/home/hadoop/namenode` | Metadatos do NameNode |
| `datanode_data` | `/home/hadoop/datanode` | Bloques de datos do HDFS |
| `./work` | `/home/hadoop/work` | Directorio compartido co host (para notebooks e datos) |

---

## 🧼 Apagar o clúster

```bash
docker compose down
```

Se queres eliminar tamén os datos persistentes (⚠️ borra HDFS!):

```bash
docker compose down -v
```

---

## 🧠 Notas adicionais

- O History Server require que o directorio `/spark-logs` exista en HDFS:
  ```bash
  docker compose exec namenode bash -lc "hdfs dfs -mkdir -p /spark-logs && hdfs dfs -chmod 1777 /spark-logs"
  ```
- O notebook xa trae todas as dependencias listadas en `jupyter/requirements.txt`.
- Podes engadir notebooks novos no directorio `work/` e aparecerán dentro de JupyterLab.

---

## 🧩 Crédits

Configuración adaptada e mantida por **Adrián Blanco (CIFP A Carballeira)**  
Baseada en imaxes personalizadas de Hadoop + Spark para docencia e práctica en contornos de Big Data.
