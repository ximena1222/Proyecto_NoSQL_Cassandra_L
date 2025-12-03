# Proyecto_NoSQL_Cassandra
Proyecto de base de datos NoSQL con Cassandra

# **Dataset: Laptop Price Dataset (Kaggle)**

**Materia:** Modelado de Datos  
**Proyecto:** Base de Datos NoSQL (Cassandra)  
**Profesor:** M. en C. Luis R. Basto  
**Integrantes:**  
- Lira Llama Ximena Estefania  
- Xool Lopez Sandy Linzay
- Canul Chacon Karol Valentina  
**Fecha de entrega:** 2 diciembre 2025

# **Descripción del DataSet:**
El dataset **“Laptop Price Dataset”** contiene información detallada sobre computadoras portátiles disponibles en el mercado, cada registro corresponde a un modelo de laptop con características como marca, procesador, RAM, almacenamiento, tarjeta gráfica, sistema operativo, peso y precio. Este dataset es adecuado para bases como cassandra ya que debido a la estructura que maneja(tabular), la ausencia de relaciones complejas y la posibilidad que hay de consultar con diferentes criterios (por marca, procesador, precio).
**Fuente:** Kaggle – Laptop Price Dataset (mkechinov)

# **Diccionario de datos:**
Brand Texto : Marca del fabricante de la laptop

Name: Texto :Nombre o modelo del equipo

Price: Decimal : Precio en euros

spec_rating: Puntuación general del dispositivo, un valor numérico que resume la calidad o desempeño según el sitio que creó el dataset.

processor: Nombre y generación del procesador

Cpu: Texto : Información del procesador

Ram: Texto : Cantidad de memoria RAM

Ram:type: Tipo de RAM: DDR4, DDR5, LPDDR5, etc.

ROM: Almacenamiento interno de la laptop

ROM_type: Tipo de almacenamiento: SSD, HDD, eMMC, etc.

Gpu: Texto : Targeta gráfica

display_size: float: Tamaño de la pantalla en pulgadas.

resolution_width: int : Resolución HORIZONTAl de la pantalla 

resolution_width: int: Resolución vertical de la pantalla 

OS: tEXTO: Sistema operativo preinstalado

Warranty: INT : Garantía en años.


# **Modelado NoSQL en Cassandra:**
Cassandra utiliza un modelo **orientado a columnas**, donde el diseño depende especificamente de las consultas que se realizarán.
Por lo que creamos tres tablas principales:
# Tabla 1: laptops_by_company
Consulta principal: *Obtener laptops por marca*
PK: company
CK: product

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
# Tabla 2: laptops_by_cpu  
Consulta principal: *Obtener laptops según procesador*
PK: cpu
CK: product
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
# Tabla 3: laptops_full (Todas las laptops)

```sql
CREATE TABLE laptops_full (
             ...     brand TEXT,
             ...     name TEXT,
             ...     price DECIMAL,
             ...     spec_rating DECIMAL,
             ...     processor TEXT,
             ...     cpu_architecture TEXT,
             ...     ram TEXT,
             ...     ram_type TEXT,
             ...     rom TEXT,
             ...     rom_type TEXT,
             ...     gpu TEXT,
             ...     display_size DECIMAL,
             ...     resolution_width INT,
             ...     resolution_height INT,
             ...     os TEXT,
             ...     warranty TEXT,
             ...     PRIMARY KEY (brand, name)
             ... );
```
Clasifica los precios pór categorías:
LOW: (<700€)
MEDIUM:  (700-1200€)
HIGH:  (>1200€)

# **Herramientas usadas:**
Cassandra: Base de datos NoSQL columnar

cqlsh: Cliente CLI para ejecutar comandos CQL

cassandra driver: Interseción de datos desde Phyton hacia Cassandra

GitHub & GitHub Desktop: Control de versione sy repositorio

# **Importanción de datos:**
Primero descargamos el dataset
Desde Kaggle: Laptop Price Dataset-mkechinov
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
# **Sentencias para cada una de las operaciones CRUD:**
1.-CREATE(insert):
```SQL
INSERT INTO laptops_by_company (company, product, typeName, inches, screenResolution, cpu, ram, memory, gpu, opSys, weight, price_euros)
VALUES ('Dell', 'XPS 14', 'Ultrabook', 14.0, '1920x1080', 'Intel i7', '16GB', '512GB SSD', 'Intel Iris', 'Windows 11', '1.3kg', 1500);

INSERT INTO laptops_by_cpu (cpu, product, company, ram, memory, gpu, price_euros)
VALUES ('Intel i5', 'HP ProBook 450', 'HP', '8GB', '256GB SSD', 'Intel HD', 650);

INSERT INTO laptops_by_price_range (price_range, product, company, cpu, ram, gpu, price_euros)
VALUES ('Low', 'Acer Aspire 1', 'Acer', 'Intel Celeron', '4GB', '64GB Flash', 'Intel HD', 299);

INSERT INTO laptops_full (brand, name, price, spec_rating, processor, cpu_architecture, ram, ram_type, rom, rom_type, gpu, display_size, resolution_width, resolution_height, os, warranty)
```
    1.1_READ:
```SQL
    __SELECT * FROM laptops_by_company WHERE company = 'Dell';__
    SELECT * FROM laptops_by_cpu WHERE cpu = 'Intel i7';
    SELECT * FROM laptops_by_price_range WHERE price_range = 'Low';
    SELECT product, price_euros FROM laptops_by_company WHERE company = 'Lenovo';
    SELECT * FROM laptops_by_company WHERE company = 'Apple';
```
    1.2_UPDATE:
```SQL
    __UPDATE laptops_by_company SET ram = '32GB' WHERE company = 'Dell' AND product = 'XPS 14';__
    UPDATE laptops_by_cpu SET price_euros = 1300 WHERE cpu = 'Intel i7' AND product = 'XPS 14';
    UPDATE laptops_by_price_range SET price_euros = 699 WHERE price_range = 'Low' AND product = 'HP Stream';
    UPDATE laptops_by_company SET inches = 15.6 WHERE company = 'HP' AND product = 'Pavilion 15';
    UPDATE laptops_by_company SET weight = '1.1kg' WHERE company = 'Apple' AND product = 'MacBook Air';
```
    1.3_DELETE:
```SQL
    __DELETE FROM laptops_by_company WHERE company = 'Dell' AND product = 'XPS 14';__
    DELETE FROM laptops_by_cpu WHERE cpu = 'Intel i5' AND product = 'HP ProBook 450';
    DELETE FROM laptops_by_price_range WHERE price_range = 'High' AND product = 'MacBook Pro 16';
    DELETE FROM laptops_by_company WHERE company = 'Acer' AND product = 'Aspire 3';
    DELETE FROM laptops_by_cpu WHERE cpu = 'AMD Ryzen 5' AND product = 'Lenovo Legion Y520';
```


# Aplicación Web CRUD
Inventario de Laptops con Cassandra

Este proyecto implementa una aplicación web simple en Flask conectada a una base de datos Apache Cassandra, que permite realizar operaciones CRUD (Create, Read, Update, Delete) sobre un dataset de laptops.

La aplicación se integra en el mismo repositorio donde se almacena el dataset (data.csv) y ofrece una interfaz gráfica para gestionar el inventario.

## Sentencias CRUD en la Base de Datos
A continuación se definen y describen 5 ejemplos de sentencias para cada operación CRUD en Cassandra CQL:

### CREATE (insertar registros)
```SQL
INSERT INTO laptops (company, product, cpu, ram, price_euros)
VALUES ('Dell', 'Inspiron 15', 'Intel i5', '8GB', 650);

INSERT INTO laptops (company, product, cpu, ram, price_euros)
VALUES ('HP', 'Pavilion', 'AMD Ryzen 7', '16GB', 850);

INSERT INTO laptops (company, product, cpu, ram, price_euros)
VALUES ('Lenovo', 'ThinkPad X1', 'Intel i7', '16GB', 1200);

INSERT INTO laptops (company, product, cpu, ram, price_euros)
VALUES ('Apple', 'MacBook Air', 'M1', '8GB', 1100);

INSERT INTO laptops (company, product, cpu, ram, price_euros)
VALUES ('Asus', 'ZenBook', 'Intel i5', '16GB', 900);
```
### READ (Consultar registros)
```SQL
SELECT * FROM laptops WHERE company = 'Dell';
SELECT product, price_euros FROM laptops WHERE company = 'HP';

SELECT * FROM laptops WHERE ram = '16GB';
SELECT company, product FROM laptops WHERE price_euros < 1000;

SELECT * FROM laptops WHERE cpu = 'Intel i7';
```
### UPDATE (Modificar Registros)
```SQL
UPDATE laptops SET ram = '32GB' WHERE company = 'Lenovo' AND product = 'ThinkPad X1';

UPDATE laptops SET price_euros = 700 WHERE company = 'Dell' AND product = 'Inspiron 15';

UPDATE laptops SET cpu = 'Intel i9' WHERE company = 'HP' AND product = 'Pavilion';

UPDATE laptops SET ram = '16GB' WHERE company = 'Apple' AND product = 'MacBook Air';

UPDATE laptops SET price_euros = 950 WHERE company = 'Asus' AND product = 'ZenBook';
```
### DELETE (Elimina Registros)
```SQL
DELETE FROM laptops WHERE company = 'Dell' AND product = 'Inspiron 15';

DELETE FROM laptops WHERE company = 'HP' AND product = 'Pavilion';

DELETE FROM laptops WHERE company = 'Lenovo' AND product = 'ThinkPad X1';

DELETE FROM laptops WHERE company = 'Apple' AND product = 'MacBook Air';

DELETE FROM laptops WHERE company = 'Asus' AND product = 'ZenBook';
```

## Operaciones Realizadas 
La aplicación web implementa las siguientes funcionalidades: 

- Inicio (/): Página principal con acceso al inventario.
- Inventario (/inventory): Visualización completa de todas las laptops en una tabla.
- Crear (/create): Formulario para agregar nuevas laptops.
- Leer (/read): Búsqueda de laptops filtradas por compañía.
- Actualizar (/update): Modificación de la memoria RAM de una laptop específica.
- Eliminar (/delete): Eliminación de laptops por compañía y producto.

## Lenguaje y herramientas utilizadas

- Lenguaje principal: Python 3
- Framework Web: Flask 3.0.0
- Base de Datos: Apache Cassandra 3.28.0
- Manejo de datos: Pandas (para cargar y procesar el dataset data.csv)
- Frontend: HTML5, CSS3 (plantillas Jinja2 y estilos personalizados)

## Ejecución 
1. Clonar el repositorio:
   ```bash
   git clone https://github.com/tu_usuario/inventario-laptops.git
    cd inventario-laptops
    ```
2. instalar independencias
   ```bash
   pip install -r requirements.txt
    ```
3. Iniciar la aplicacion Flask
    ```bash
   flask run
    ```
4. Acceder al navegador
    ```bash
    http://127.0.0.1:5000
    ```
## Conclusión
El proyecto demuestra cómo integrar un dataset real en una base de datos distribuida (Cassandra) y gestionarlo mediante una aplicación web en Flask.  
La implementación de las operaciones CRUD garantiza que el sistema sea funcional y extensible, y el diseño modular permite futuras mejoras como autenticación de usuarios, exportación de datos y generación de estadísticas.
