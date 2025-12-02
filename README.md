# Proyecto_NoSQL_Cassandra_L
Proyecto de base de datos NoSQL con Cassandra
## **Dataset: Laptop Price Dataset (Kaggle)**

**Materia:** Modelos de Datos  
**Proyecto:** Base de Datos NoSQL – Cassandra  
**Profesor:** M. en C. Luis R. Basto  
**Integrantes:**  
- Lira Llama Ximena Estefania  
- Xool Lopez Sandy Linzay
- Canul Chacon Karol Valentina  
**Fecha de entrega:** 2 diciembre 2025

# b.**Descripción del DataSet:**
El dataset **“Laptop Price Dataset”** contiene información detallada sobre computadoras portátiles disponibles en el mercado. Cada registro corresponde a un modelo de laptop con características como marca, procesador, RAM, almacenamiento, tarjeta gráfica, sistema operativo, peso y precio.
**Fuente:** Kaggle – Laptop Price Dataset (mkechinov)

# c.**Diccionario de datos:**
 **Columna            Tipo      Descripción** 
 id                   int       Identificador único            
 brand                text      Marca de la laptop             
 name                 text      Modelo comercial               
 price                int       Precio en moneda local         
 spec_rating          float     Calificación del rendimiento   
 processor            text      Nombre del procesador          
 cpu                  text      Tipo de arquitectura y núcleos 
 ram                  text      Memoria RAM disponible         
 ram_type             text      Tipo de memoria RAM            
 rom                  text      Almacenamiento (capacidad)     
 rom_type             text      Tipo de almacenamiento         
 gpu                  text      Procesador gráfico             
 display_size         float     Tamaño de pantalla             
 resolution_width     int       Resolución horizontal          
 resolution_height    int       Resolución vertical            
 os                   text      Sistema operativo              
 warranty             int       Garantía del producto    
 

# d.**Modelado NoSQL en Cassandra:**
Cassandra utiliza un modelo **orientado a columnas**. Algunas caracteristicas por el cual cassandra es apta para este diseño es que no exite una relacion estricta entre las laptops ademas las consultas son independientes por ID, marca o caracteristica y Cassandra escala horizontalmente sin afectar el rendimiento. 

# d.**Herramientas Utilizadas:**                          
**Windows + WSL Ubuntu**: Ejecutar servicios Linux             
**Apache Cassandra 5.0.6**: Base de datos NoSQL                  
**cqlsh**:Cliente para enviar comandos CQL      
**VSCode / Git**:Documentación y control de versiones 

# f.**Importanción de datos:**
1: Mover el dataset a WSL
mkdir ~/data
cp /mnt/c/Users/Downloads/data.csv ~/data/data.csv
2:Quitar la columna Unnamed:0
cd ~/data
cut -d',' -f2- data.csv > data_clean.csv
3: Crear Keypace 
CREATE KEYSPACE laptops_db 
WITH replication = {'class':'SimpleStrategy','replication_factor':1};
USE laptops_db;
4: Crear la tabla:
CREATE TABLE laptops (
  id int PRIMARY KEY,
  brand text,
  name text,
  price int,
  spec_rating float,
  processor text,
  cpu text,
  ram text,
  ram_type text,
  rom text,
  rom_type text,
  gpu text,
  display_size float,
  resolution_width int,
  resolution_height int,
  os text,
  warranty int
);
5: Importar el archivo
COPY laptops FROM '/home/USER/data/data_clean.csv' WITH HEADER = TRUE;

# g.**Operaciones CRUD:**
**CREATE**
INSERT INTO laptops (id, brand, name, price) VALUES (999, 'HP', 'Test Model', 50000);
INSERT INTO laptops (id, brand, name) VALUES (998, 'Dell', 'Inspiron');
INSERT INTO laptops (id, brand, name, os) VALUES (997, 'Acer', 'Predator', 'Windows 11');
INSERT INTO laptops (id, brand, price) VALUES (996, 'Lenovo', 32990);
INSERT INTO laptops (id, brand, warranty) VALUES (995, 'MSI', 1);
**READ**
SELECT * FROM laptops LIMIT 20;
SELECT brand, price FROM laptops WHERE id=1;
SELECT * FROM laptops WHERE brand='HP' ALLOW FILTERING;
SELECT * FROM laptops WHERE price < 30000 ALLOW FILTERING;
SELECT name, gpu FROM laptops WHERE id=20;
**UPDATE**
UPDATE laptops SET price=59990 WHERE id=10;
UPDATE laptops SET warranty=2 WHERE id=50;
UPDATE laptops SET os='Ubuntu' WHERE id=3;
UPDATE laptops SET ram='16GB' WHERE id=100;
UPDATE laptops SET brand='Samsung' WHERE id=200;
**DELETE**
DELETE FROM laptops WHERE id=999;
DELETE brand FROM laptops WHERE id=10;
DELETE price FROM laptops WHERE id=200;
DELETE ram FROM laptops WHERE id=123;
DELETE FROM laptops WHERE id=500;



