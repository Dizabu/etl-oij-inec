# ETL Pipeline: Crime and Census Data of Costa Rica (2011)

An **ETL (extract, transform, load) pipeline** built with **PySpark** that cleans two public datasets from
Costa Rica, loads them into **PostgreSQL**, and analyzes them with **SQL** to look for relationships between
crime and employment by district.

University project for *IC4302 - Bases de Datos II* at the Costa Rican Institute of Technology (TEC), June 2025.
The full walkthrough, code and documentation are in [`Proyecto3.ipynb`](Proyecto3.ipynb) (in Spanish).

## What it does

1. **Extract:** reads two Excel datasets from 2011: about 47,000 crime records from the **OIJ** (Organismo de
   Investigación Judicial) and employment indicators by district from the **INEC** census.
2. **Transform:** the datasets name districts differently, so they could not be joined. PySpark functions
   normalize the names (spaces, case, accents and `ñ`), and an **anti-join** finds the districts that still
   don't match. After cleaning, only 23 districts remain unmatched.
3. **Load:** writes the cleaned DataFrames into PostgreSQL through JDBC.
4. **Analyze:** SQL queries over PostgreSQL, charted with Matplotlib:
   - the 10 districts with the most crimes vs. their employment rate (`LEFT JOIN` of two aggregated subqueries)
   - crimes by day of the week in the district with the most crimes (`WITH` CTE + `EXTRACT(DOW ...)`)
   - the most frequent crime types in that district (window function `RANK() OVER (...)`)
   - crimes by sex and by nationality (`GROUP BY` aggregations)

## Example query

```sql
WITH distrito_top AS (
    SELECT distrito
    FROM DatosOIJ
    GROUP BY distrito
    ORDER BY COUNT(*) DESC
    LIMIT 1
)
SELECT EXTRACT(DOW FROM "Fecha") AS dia_semana, COUNT(*) AS delitos
FROM DatosOIJ
WHERE distrito = (SELECT distrito FROM distrito_top)
GROUP BY dia_semana
ORDER BY dia_semana;
```

## Run it

Requirements: Python 3, Java (for Spark), a local PostgreSQL database named `Proyecto3`, and the
[PostgreSQL JDBC driver](https://jdbc.postgresql.org/download/) (`postgresql-42.7.3.jar`).

```bash
pip install -r requirements.txt
export POSTGRESQL_PASSWORD=your_password
export JDBC_DRIVER_JAR=/path/to/postgresql-42.7.3.jar
jupyter notebook Proyecto3.ipynb
```

Credentials are read from environment variables, never stored in the code.

## Stack

Python · PySpark · Spark SQL · PostgreSQL · JDBC · pandas · Matplotlib · Jupyter

## Team

Angie Esquivel López · Diego Zamora Bustos · Jose Adrián Herrera Salas · Sebastian Quesada Córdoba

## Data sources

Public 2011 datasets from the INEC (Costa Rica's National Institute of Statistics and Censuses) and the OIJ
(Judicial Investigation Agency), provided by the course professor.
