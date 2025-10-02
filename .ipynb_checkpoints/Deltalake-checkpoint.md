{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 1,
   "id": "dbe8857a-e551-479f-91a5-87ae4d247954",
   "metadata": {},
   "outputs": [],
   "source": [
    "from pyspark.sql import SparkSession\n",
    "from pyspark.sql.types import StructType, StructField, StringType, FloatType\n",
    "\n",
    "from delta import *\n",
    "\n",
    "import logging\n",
    "\n",
    "logging.getLogger(\"py4j\").setLevel(logging.DEBUG)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "id": "01af4cee-a4b9-4b23-8f72-766e1cd4509f",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "25/10/01 21:22:19 WARN Utils: Your hostname, DESKTOP-ES2AP8K resolves to a loopback address: 127.0.1.1; using 10.255.255.254 instead (on interface lo)\n",
      "25/10/01 21:22:19 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address\n"
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      ":: loading settings :: url = jar:file:/mnt/c/Users/Alexandre/.venv/lib/python3.13/site-packages/pyspark/jars/ivy-2.5.1.jar!/org/apache/ivy/core/settings/ivysettings.xml\n"
     ]
    },
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "Ivy Default Cache set to: /home/alexandre/.ivy2/cache\n",
      "The jars for the packages stored in: /home/alexandre/.ivy2/jars\n",
      "io.delta#delta-spark_2.12 added as a dependency\n",
      ":: resolving dependencies :: org.apache.spark#spark-submit-parent-196a627c-64fa-4a14-bd1f-e8f3b3779975;1.0\n",
      "\tconfs: [default]\n",
      "\tfound io.delta#delta-spark_2.12;3.2.0 in central\n",
      "\tfound io.delta#delta-storage;3.2.0 in central\n",
      "\tfound org.antlr#antlr4-runtime;4.9.3 in central\n",
      ":: resolution report :: resolve 379ms :: artifacts dl 14ms\n",
      "\t:: modules in use:\n",
      "\tio.delta#delta-spark_2.12;3.2.0 from central in [default]\n",
      "\tio.delta#delta-storage;3.2.0 from central in [default]\n",
      "\torg.antlr#antlr4-runtime;4.9.3 from central in [default]\n",
      "\t---------------------------------------------------------------------\n",
      "\t|                  |            modules            ||   artifacts   |\n",
      "\t|       conf       | number| search|dwnlded|evicted|| number|dwnlded|\n",
      "\t---------------------------------------------------------------------\n",
      "\t|      default     |   3   |   0   |   0   |   0   ||   3   |   0   |\n",
      "\t---------------------------------------------------------------------\n",
      ":: retrieving :: org.apache.spark#spark-submit-parent-196a627c-64fa-4a14-bd1f-e8f3b3779975\n",
      "\tconfs: [default]\n",
      "\t0 artifacts copied, 3 already retrieved (0kB/11ms)\n",
      "25/10/01 21:22:22 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable\n",
      "Setting default log level to \"WARN\".\n",
      "To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).\n"
     ]
    }
   ],
   "source": [
    "spark = (\n",
    "    SparkSession\n",
    "    .builder\n",
    "    .master(\"local[*]\")\n",
    "    .config(\"spark.jars.packages\", \"io.delta:delta-spark_2.12:3.2.0\")\n",
    "    .config(\"spark.sql.extensions\", \"io.delta.sql.DeltaSparkSessionExtension\")\n",
    "    .config(\"spark.sql.catalog.spark_catalog\", \"org.apache.spark.sql.delta.catalog.DeltaCatalog\")\n",
    "    .getOrCreate()\n",
    ")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "id": "eca4378c-7d75-47ed-845c-614d9f35dce7",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "                                                                                "
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "+----------+------------+---+-------+--------------+\n",
      "|ID_CLIENTE|NOME_CLIENTE|UF |STATUS |LIMITE_CREDITO|\n",
      "+----------+------------+---+-------+--------------+\n",
      "|ID001     |CLIENTE_X   |SP |ATIVO  |250000.0      |\n",
      "|ID002     |CLIENTE_Y   |SC |INATIVO|400000.0      |\n",
      "|ID003     |CLIENTE_Z   |DF |ATIVO  |1000000.0     |\n",
      "+----------+------------+---+-------+--------------+\n",
      "\n"
     ]
    }
   ],
   "source": [
    "data = [\n",
    "    (\"ID001\", \"CLIENTE_X\",\"SP\",\"ATIVO\",   250000.00),\n",
    "    (\"ID002\", \"CLIENTE_Y\",\"SC\",\"INATIVO\", 400000.00),\n",
    "    (\"ID003\", \"CLIENTE_Z\",\"DF\",\"ATIVO\",   1000000.00)\n",
    "]\n",
    "\n",
    "schema = (\n",
    "    StructType([\n",
    "        StructField(\"ID_CLIENTE\",     StringType(),True),\n",
    "        StructField(\"NOME_CLIENTE\",   StringType(),True),\n",
    "        StructField(\"UF\",             StringType(),True),\n",
    "        StructField(\"STATUS\",         StringType(),True),\n",
    "        StructField(\"LIMITE_CREDITO\", FloatType(), True)\n",
    "    ])\n",
    ")\n",
    "\n",
    "df = spark.createDataFrame(data=data,schema=schema)\n",
    "\n",
    "df.show(truncate=False)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d1965a58-cb7a-4a60-9b25-a5c1239e7534",
   "metadata": {},
   "source": [
    "# GRAVANDO DELTA TABLE"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "id": "80aa6008-048a-4933-a972-d5f98793e61d",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "                                                                                "
     ]
    }
   ],
   "source": [
    "( \n",
    "    df\n",
    "    .write\n",
    "    .format(\"delta\")\n",
    "    .mode('overwrite')\n",
    "    .save(\"./RAW/CLIENTES\")\n",
    ")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "782763cf-b684-4d61-950a-1a35fba8f548",
   "metadata": {},
   "source": [
    "# INSERINDO NOVOS DADOS"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "id": "07135533-2586-498a-a055-1f86190b1876",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "+----------+------------+---+-------+--------------+\n",
      "|ID_CLIENTE|NOME_CLIENTE| UF| STATUS|LIMITE_CREDITO|\n",
      "+----------+------------+---+-------+--------------+\n",
      "|     ID001|   CLIENTE_X| SP|INATIVO|           0.0|\n",
      "|     ID002|   CLIENTE_Y| SC|  ATIVO|      400000.0|\n",
      "|     ID004|   CLIENTE_Z| DF|  ATIVO|     5000000.0|\n",
      "+----------+------------+---+-------+--------------+\n",
      "\n"
     ]
    }
   ],
   "source": [
    "new_data = [\n",
    "    (\"ID001\",\"CLIENTE_X\",\"SP\",\"INATIVO\", 0.00),\n",
    "    (\"ID002\",\"CLIENTE_Y\",\"SC\",\"ATIVO\",   400000.00),\n",
    "    (\"ID004\",\"CLIENTE_Z\",\"DF\",\"ATIVO\",   5000000.00)\n",
    "]\n",
    "\n",
    "df_new = spark.createDataFrame(data=new_data, schema=schema)\n",
    "\n",
    "df_new.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "77d3a30f-2729-4edd-887f-c1860761d6bd",
   "metadata": {},
   "source": [
    "# OPERAÇÃO MERGE,INSERIR E ATUALIZAR"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "id": "ade818be-71c4-46d2-8a5e-90d475a5022c",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "25/10/01 21:25:57 WARN SparkStringUtils: Truncated the string representation of a plan since it was too large. This behavior can be adjusted by setting 'spark.sql.debug.maxToStringFields'.\n",
      "                                                                                "
     ]
    }
   ],
   "source": [
    "deltaTable = DeltaTable.forPath(spark, \"./RAW/CLIENTES\")\n",
    "\n",
    "(\n",
    "    deltaTable.alias(\"dados_atuais\")\n",
    "    .merge(\n",
    "        df_new.alias(\"novos_dados\"),\n",
    "        \"dados_atuais.ID_CLIENTE = novos_dados.ID_CLIENTE\"\n",
    "    )\n",
    "    .whenMatchedUpdateAll()\n",
    "    .whenNotMatchedInsertAll()\n",
    "    .execute()\n",
    ")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "id": "5fb9e5f0-7313-4981-91f2-30ab59bc9d92",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "+----------+------------+---+-------+--------------+\n",
      "|ID_CLIENTE|NOME_CLIENTE| UF| STATUS|LIMITE_CREDITO|\n",
      "+----------+------------+---+-------+--------------+\n",
      "|     ID001|   CLIENTE_X| SP|INATIVO|           0.0|\n",
      "|     ID002|   CLIENTE_Y| SC|  ATIVO|      400000.0|\n",
      "|     ID004|   CLIENTE_Z| DF|  ATIVO|     5000000.0|\n",
      "|     ID003|   CLIENTE_Z| DF|  ATIVO|     1000000.0|\n",
      "+----------+------------+---+-------+--------------+\n",
      "\n"
     ]
    }
   ],
   "source": [
    "deltaTable.toDF().show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "fe051b2f-0955-4feb-9617-a5b24e49b420",
   "metadata": {},
   "source": [
    "# OPERAÇÃO DELETE"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "id": "7937dd70-c898-473f-be9f-d8b955dd9b40",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "                                                                                "
     ]
    },
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "+----------+------------+---+------+--------------+\n",
      "|ID_CLIENTE|NOME_CLIENTE| UF|STATUS|LIMITE_CREDITO|\n",
      "+----------+------------+---+------+--------------+\n",
      "|     ID003|   CLIENTE_Z| DF| ATIVO|     1000000.0|\n",
      "|     ID004|   CLIENTE_Z| DF| ATIVO|     5000000.0|\n",
      "+----------+------------+---+------+--------------+\n",
      "\n"
     ]
    }
   ],
   "source": [
    "deltaTable.delete(\"LIMITE_CREDITO < 500000.0\")\n",
    "deltaTable.toDF().show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d0280b8c-49f2-45b6-8e6b-933c2c366ac9",
   "metadata": {},
   "source": [
    "# OPERAÇÃO UPDATE"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "id": "cc0b7594-e52d-446f-abb1-8245abde45d3",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "+----------+------------+---+------+--------------+\n",
      "|ID_CLIENTE|NOME_CLIENTE| UF|STATUS|LIMITE_CREDITO|\n",
      "+----------+------------+---+------+--------------+\n",
      "|     ID003|   CLIENTE_Z| DF| ATIVO|     1000000.0|\n",
      "|     ID004|   CLIENTE_Z| DF| ATIVO|         1.2E7|\n",
      "+----------+------------+---+------+--------------+\n",
      "\n"
     ]
    }
   ],
   "source": [
    "eltaTable = DeltaTable.forPath(spark, \"./RAW/CLIENTES\")\n",
    "\n",
    "# Atualizar LIMITE_CREDITO do cliente ID003\n",
    "deltaTable.update(\n",
    "    condition = \"ID_CLIENTE = 'ID004'\",\n",
    "    set = { \"LIMITE_CREDITO\": \"12000000.0\" }\n",
    ")\n",
    "deltaTable.toDF().show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "23abfe82-89eb-43c1-b77c-deb189acf5d2",
   "metadata": {},
   "outputs": [],
   "source": [
    "uv init .\n",
    "pyenv local 3.13.7\n",
    "uv venv\n",
    "source .venv/bin/activate\n",
    "uv add pyspark==3.5.3 delta-spark==3.2.0 jupyterlab ipykernel\n",
    "pip install ipykernel"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "1ecbf42d-c160-47d6-8f09-a52c8dc89600",
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.13.7"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}
