# Supported Databases

This document summarizes database targets commonly used with this SMT, organized by output format category.

---

## Native Target Database Types

These database types are available through `target.database` and have first-class formatter behavior.

### JDBC — Schema + Struct Output

| Database | Config Value | Boolean | Timestamp | Arrays | Notes |
|---|---|---|---|---|---|
| **MySQL** | `mysql` | TINYINT(1) | DATETIME | JSON string | MariaDB compatible |
| **PostgreSQL** | `postgresql` | Native BOOLEAN | TIMESTAMPTZ | Native ARRAY | JSONB for complex objects |
| **SQL Server** | `sqlserver` | BIT | DATETIME2 | NVARCHAR(MAX) JSON | Azure SQL compatible |
| **Oracle** | `oracle` | NUMBER(1) | TIMESTAMP WITH TZ | CLOB JSON | Oracle 19c+ recommended |
| **Snowflake** | `snowflake` | Native BOOLEAN | TIMESTAMP_NTZ | VARIANT | Entire record as VARIANT JSON |
| **Delta Lake** | `delta_lake` | Native BOOLEAN | TIMESTAMP | Struct | Databricks / Spark compatible |
| **Embedded** | `embedded` | Native BOOLEAN | TIMESTAMP | JSON string | SQLite, DuckDB via JDBC |

### Schemaless — JSON Map Output

| Database | Config Value | Notes |
|---|---|---|
| **MongoDB** | `mongodb` | Default. LinkedHashMap preserving field order |
| **Neo4j** | `neo4j` | Graph mapping handled by sink connector |
| **InfluxDB** | `influxdb` | Field/tag mapping handled by sink connector |
| **VectorDB** | `vectordb` | Preserves embedding arrays for Pinecone, Milvus, Weaviate |
| **RDF** | `rdf` | For triple stores — RDF serialization handled by sink |
| **Ledger** | `ledger` | For immutable databases like Amazon QLDB, Hyperledger |
| **Object Store** | `object_store` | For S3, GCS, Azure Blob JSON persistence |

---

## Common JDBC-Compatible Targets

These systems are commonly used with the existing JDBC-oriented formatter behavior. Final compatibility depends on sink connector dialect support and target-side constraints.

| Database | Recommended Config | Why It Works |
|---|---|---|
| **MariaDB** | `mysql` | Wire-compatible with MySQL |
| **Azure SQL** | `sqlserver` | Same TDS protocol as SQL Server |
| **Amazon Aurora (MySQL)** | `mysql` | MySQL-compatible engine |
| **Amazon Aurora (PostgreSQL)** | `postgresql` | PostgreSQL-compatible engine |
| **Google Cloud SQL (MySQL)** | `mysql` | Managed MySQL |
| **Google Cloud SQL (PostgreSQL)** | `postgresql` | Managed PostgreSQL |
| **Azure Database for MySQL** | `mysql` | Managed MySQL |
| **Azure Database for PostgreSQL** | `postgresql` | Managed PostgreSQL |
| **CockroachDB** | `postgresql` | PostgreSQL wire protocol |
| **YugabyteDB (YSQL)** | `postgresql` | PostgreSQL-compatible API |
| **TimescaleDB** | `postgresql` | PostgreSQL extension |
| **Citus** | `postgresql` | PostgreSQL extension |
| **AlloyDB** | `postgresql` | Google's PostgreSQL-compatible |
| **Neon** | `postgresql` | Serverless PostgreSQL |
| **Supabase** | `postgresql` | PostgreSQL-based |
| **PlanetScale** | `mysql` | MySQL-compatible (Vitess) |
| **TiDB** | `mysql` | MySQL-compatible |
| **SingleStore (MemSQL)** | `mysql` | MySQL wire protocol |
| **Percona Server** | `mysql` | MySQL drop-in replacement |
| **Oracle Autonomous DB** | `oracle` | Managed Oracle |
| **Exadata** | `oracle` | Oracle-compatible |
| **SAP HANA** | `sqlserver` | JDBC Struct compatible |
| **Vertica** | `postgresql` | PostgreSQL-style types |
| **Greenplum** | `postgresql` | PostgreSQL-based MPP |
| **Redshift** | `postgresql` | PostgreSQL-compatible |
| **H2** | `embedded` | Embedded JDBC |
| **HSQLDB** | `embedded` | Embedded JDBC |
| **Derby** | `embedded` | Embedded JDBC |
| **SQLite** | `embedded` | Via JDBC driver |
| **DuckDB** | `embedded` | Via JDBC driver |
| **ClickHouse** | `mysql` | MySQL wire protocol support |
| **Databricks SQL** | `delta_lake` | Delta Lake native |

---

## Common Schemaless-Compatible Targets

These systems are commonly used with schemaless JSON-map output, where the sink connector performs target-specific translation.

| Database | Recommended Config | Why It Works |
|---|---|---|
| **Amazon DocumentDB** | `mongodb` | MongoDB API compatible |
| **Azure Cosmos DB (MongoDB API)** | `mongodb` | MongoDB wire protocol |
| **Azure Cosmos DB (SQL API)** | `object_store` | JSON document store |
| **Couchbase** | `mongodb` | JSON document model |
| **CouchDB** | `mongodb` | JSON document store |
| **Firebase / Firestore** | `object_store` | JSON document model |
| **DynamoDB** | `object_store` | JSON-like document store |
| **Elasticsearch** | `object_store` | JSON document indexing |
| **OpenSearch** | `object_store` | Elasticsearch fork |
| **Apache Solr** | `object_store` | JSON document indexing |
| **Redis (JSON module)** | `mongodb` | JSON value storage |
| **ArangoDB** | `mongodb` | Multi-model JSON documents |
| **RavenDB** | `mongodb` | JSON document store |
| **MarkLogic** | `object_store` | Multi-model JSON/XML |
| **Amazon Neptune** | `neo4j` | Graph database |
| **JanusGraph** | `neo4j` | Graph database |
| **TigerGraph** | `neo4j` | Graph database |
| **Amazon Timestream** | `influxdb` | Time-series database |
| **QuestDB** | `influxdb` | InfluxDB line protocol |
| **TDengine** | `influxdb` | Time-series database |
| **CrateDB** | `influxdb` | Time-series + SQL |
| **Pinecone** | `vectordb` | Vector similarity search |
| **Milvus** | `vectordb` | Vector database |
| **Weaviate** | `vectordb` | Vector database |
| **Qdrant** | `vectordb` | Vector database |
| **Chroma** | `vectordb` | Vector database |
| **Apache Jena / Fuseki** | `rdf` | RDF triple store |
| **GraphDB (Ontotext)** | `rdf` | RDF triple store |
| **Stardog** | `rdf` | RDF knowledge graph |
| **Amazon QLDB** | `ledger` | Immutable ledger |
| **Hyperledger Fabric** | `ledger` | Blockchain ledger |
| **Google Cloud Storage** | `object_store` | Object/blob storage |
| **Azure Blob Storage** | `object_store` | Object/blob storage |
| **Amazon S3** | `object_store` | Object/blob storage |
| **MinIO** | `object_store` | S3-compatible storage |

---

## Configuration Example

```properties
# For a PostgreSQL-compatible database (e.g., CockroachDB, YugabyteDB)
transforms.Extract.target.database=postgresql

# For a MongoDB-compatible database (e.g., DocumentDB, Cosmos DB)
transforms.Extract.target.database=mongodb

# For a vector database (e.g., Pinecone, Milvus)
transforms.Extract.target.database=vectordb
```

---

## Summary

| Category | Scope |
|---|---|
| **Native target database types** | Core set configured via `target.database` |
| **JDBC-compatible targets** | Broad ecosystem, validate connector + dialect behavior |
| **Schemaless-compatible targets** | Broad ecosystem, validate connector mapping expectations |
