# Proyecto_NoSQL_Cassandra_L
Proyecto de base de datos NoSQL con Cassandra
## **Dataset: Laptop Price Dataset (Kaggle)**

**Materia:** Modelos de Datos  
**Proyecto:** Base de Datos NoSQL – Cassandra  
**Profesor:** M. en C. Luis R. Basto  
**Integrantes:**  
- Lira Llama Ximena Estefania  
- Xool Lopez Sandy Linzay
- Valen (escribe tu nombre completo vale)  
**Fecha de entrega:** 2 diciembre 2025

# b.**Descripción del DataSet:**
El dataset **“Laptop Price Dataset”** contiene información detallada sobre computadoras portátiles disponibles en el mercado. Cada registro corresponde a un modelo de laptop con características como marca, procesador, RAM, almacenamiento, tarjeta gráfica, sistema operativo, peso y precio.
**Fuente:** Kaggle – Laptop Price Dataset (mkechinov)

# c.**Diccionario de datos:**
Company: Texto : Marca del fabricante de la laptot
Product: Texto :Nombre o modelo del equipo
TypeName: Texto : Categoría (si es notebook, gaming, étc.)
Inches: Decimal : Tamaño de la pantalla en pulgadas
ScreenResolution: Texto : Resolución de la pantalla
Cpu: Texto : Información del procesador
Ram: Texto : Cantidad de memoria RAM
Memory: Texto : Tipo y capacidad de almacenamiento
Gpu: Texto : Targeta gráfica
OpSys: Texto : Sistema operativo que maneja
Weight: Texto : Peso del equipo
Price_euros: Decimal : Precio en euros

# d.**Modelado NoSQL en Cassandra:**
Cassandra utiliza un modelo **orientado a columnas**, donde el diseño depende de las consultas que se realizarán.
Creamos tres tablas principales:
## Tabla 1: laptops_by_company  
Consulta principal: *Obtener laptops por marca*

```sql
CREATE TABLE laptops_by_company (
    company TEXT,
    product TEXT,
    typeName TEXT,
    inches DECIMAL,
    screenResolution TEXT,
    cpu TEXT,
    ram TEXT,
    memory TEXT,
    gpu TEXT,
    opSys TEXT,
    weight TEXT,
    price_euros DECIMAL,
    PRIMARY KEY (company, product)
);
```
## Tabla 2: laptops_by_cpu  
Consulta principal: *Obtener laptops según procesador*
```sql
CREATE TABLE laptops_by_cpu (
    cpu TEXT,
    product TEXT,
    company TEXT,
    ram TEXT,
    memory TEXT,
    gpu TEXT,
    price_euros DECIMAL,
    PRIMARY KEY (cpu, product)
);
```
## Tabla 3: laptops_by_priceRange  
Consulta principal: *Obtener laptops por rango de precio*
```sql
CREATE TABLE laptops_by_price_range (
    price_range TEXT,
    product TEXT,
    company TEXT,
    cpu TEXT,
    ram TEXT,
    gpu TEXT,
    price_euros DECIMAL,
    PRIMARY KEY (price_range, product)
);
```
# e.**Herramientas usadas:**
Cassandra: Base de datos NoSQL columnar
cqlsh: Cliente CLI para ejecutar comandos CQL
cassandra driver: Interseción de datos desde Phyton hacia Cassandra
GitHub & GitHub Desktop: Control de versione sy repositorio
Python 3 + Pandas: Para la lectura del CSV

# f.**Importanción de datos:**
1.-Creamos el Keyspace:
```Sql
CREATE KEYSPACE laptopsdb
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 1
};
```
2.-Creamos las tablas (anteriormente mencionadas)
3.-Script de importación en Python:
```python
from cassandra.cluster import Cluster
import pandas as pd

cluster = Cluster(['127.0.0.1'])
session = cluster.connect('laptopsdb')

df = pd.read_csv('laptops.csv')

for _, row in df.iterrows():
    price = row['Price_euros']
    if price < 700:
        price_range = 'Low'
    elif price <= 1200:
        price_range = 'Medium'
    else:
        price_range = 'High'

    session.execute("""
        INSERT INTO laptops_by_company (company, product, typeName, inches, screenResolution, cpu, ram, memory, gpu, opSys, weight, price_euros)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
    """, (row['Company'], row['Product'], row['TypeName'], row['Inches'], row['ScreenResolution'], row['Cpu'], row['Ram'], row['Memory'], row['Gpu'], row['OpSys'], row['Weight'], row['Price_euros']))

    session.execute("""
        INSERT INTO laptops_by_cpu (cpu, product, company, ram, memory, gpu, price_euros)
        VALUES (%s, %s, %s, %s, %s, %s, %s)
    """, (row['Cpu'], row['Product'], row['Company'], row['Ram'], row['Memory'], row['Gpu'], row['Price_euros']))

    session.execute("""
        INSERT INTO laptops_by_price_range (price_range, product, company, cpu, ram, gpu, price_euros)
        VALUES (%s, %s, %s, %s, %s, %s, %s)
    """, (price_range, row['Product'], row['Company'], row['Cpu'], row['Ram'], row['Gpu'], row['Price_euros']))
```
# g.**Sentencias para cada una de las operaciones CRUD:**
1.-CRUD:
```SQL
INSERT INTO laptops_by_company (company, product, typeName, inches, screenResolution, cpu, ram, memory, gpu, opSys, weight, price_euros)
VALUES ('Dell', 'XPS 14', 'Ultrabook', 14.0, '1920x1080', 'Intel i7', '16GB', '512GB SSD', 'Intel Iris', 'Windows 11', '1.3kg', 1500);

INSERT INTO laptops_by_cpu (cpu, product, company, ram, memory, gpu, price_euros)
VALUES ('Intel i5', 'HP ProBook 450', 'HP', '8GB', '256GB SSD', 'Intel HD', 650);

INSERT INTO laptops_by_price_range (price_range, product, company, cpu, ram, gpu, price_euros)
VALUES ('Low', 'Acer Aspire 1', 'Acer', 'Intel Celeron', '4GB', '64GB Flash', 'Intel HD', 299);

INSERT INTO laptops_by_company (...) VALUES (...);
INSERT INTO laptops_by_cpu (...) VALUES (...);
```
    1.1_READ:
```SQL
    SELECT * FROM laptops_by_company WHERE company = 'Dell';
    SELECT * FROM laptops_by_cpu WHERE cpu = 'Intel i7';
    SELECT * FROM laptops_by_price_range WHERE price_range = 'Low';
    SELECT product, price_euros FROM laptops_by_company WHERE company = 'Lenovo';
    SELECT * FROM laptops_by_company WHERE company = 'Apple';
```
    1.2_UPDATE:
```SQL
    UPDATE laptops_by_company SET ram = '32GB' WHERE company = 'Dell' AND product = 'XPS 14';
    UPDATE laptops_by_cpu SET price_euros = 1300 WHERE cpu = 'Intel i7' AND product = 'XPS 14';
    UPDATE laptops_by_price_range SET price_euros = 699 WHERE price_range = 'Low' AND product = 'HP Stream';
    UPDATE laptops_by_company SET inches = 15.6 WHERE company = 'HP' AND product = 'Pavilion 15';
    UPDATE laptops_by_company SET weight = '1.1kg' WHERE company = 'Apple' AND product = 'MacBook Air';
```
    1.3_DELETE:
```SQL
    DELETE FROM laptops_by_company WHERE company = 'Dell' AND product = 'XPS 14';
    DELETE FROM laptops_by_cpu WHERE cpu = 'Intel i5' AND product = 'HP ProBook 450';
    DELETE FROM laptops_by_price_range WHERE price_range = 'High' AND product = 'MacBook Pro 16';
    DELETE FROM laptops_by_company WHERE company = 'Acer' AND product = 'Aspire 3';
    DELETE FROM laptops_by_cpu WHERE cpu = 'AMD Ryzen 5' AND product = 'Lenovo Legion Y520';
```