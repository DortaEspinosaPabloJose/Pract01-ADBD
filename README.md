# Ejercicios de ADBD práctica 1\.

#### Pablo José Dorta Espinosa

En este documento se realiza cada ejercicio de la práctica, separado tarea donde primero se presenta el enunciado y luego un bloque de código con el comando y la salida de este copiados y pegados tal y como estaba en la terminal a la hora de realizar el ejercicio, a veces con espacios verticales añadidos para claridad. 

1. Creación de la base de datos  
   1. Crear una base de datos llamada biblioteca.

```sql
postgres=# CREATE DATABASE biblioteca;
CREATE DATABASE
```

2. Creación de usuarios  
   1. Crear dos usuarios:  
      1. admin\_biblio con permisos de administrador sobre la base de datos.  
      2. usuario\_biblio con permisos solo de lectura.  
   2. Crear un rol llamado lectores con permisos únicamente de consulta sobre todas las tablas de la base de datos.  
   3. Asignar el usuario usuario\_biblio a este rol.  
   4. Consultar las tablas del sistema para listar todos los usuarios creados (pg\_roles).  
   5. Cambiar la contraseña del usuario usuario\_biblio.  
   6. Configurar permisos de tal forma que el usuario usuario\_biblio no pueda eliminar registros en ninguna tabla.

```sql
postgres=# CREATE USER admin_biblio;
CREATE ROLE
postgres=# ALTER DATABASE biblioteca OWNER TO admin_biblio; 
ALTER DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE biblioteca TO admin_biblio;
GRANT
postgres=#GRANT ALL ON SCHEMA public TO admin_biblio; 
GRANT

postgres=# CREATE ROLE lectores;
CREATE ROLE
postgres=# GRANT pg_read_all_data TO lectores;
GRANT ROLE
postgres=# GRANT CONNECT ON DATABASE biblioteca TO lectores;
GRANT

postgres=# SELECT rolname FROM pg_roles WHERE rolname NOT LIKE 'pg_%';
    rolname
----------------
 postgres
 mydb_admin
 admin_biblio
 usuario_biblio
 lectores

postgres=# ALTER USER usuario_biblio WITH PASSWORD 'nueva_clave_123';
ALTER ROLE
```

De aquí en adelante, se accede a la base de datos biblioteca con el usuario admin\_biblio:

```sql
postgres=# ALTER USER admin_biblio WITH PASSWORD 'clave_admin_123';
ALTER ROLE
postgres=# exit
usuario@ubuntu:~$ psql -U admin_biblio -h localhost -d biblioteca
```

3. Creación de tablas  
   1. Crear las siguientes tablas con sus respectivas claves primarias:  
      1. autores(id\_autor, nombre, nacionalidad)  
      2. libros(id\_libro, titulo, año\_publicacion, id\_autor)  
      3. prestamos(id\_prestamo, id\_libro, fecha\_prestamo, fecha\_devolucion, usuario\_prestatario)  
   2. Establecer las claves foráneas correspondientes.

```sql
biblioteca=# CREATE TABLE autores (
    id_autor     SERIAL PRIMARY KEY,
    nombre       VARCHAR(100) NOT NULL,
    nacionalidad VARCHAR(50)
);
CREATE TABLE

biblioteca=# CREATE TABLE libros (
    id_libro        SERIAL PRIMARY KEY,
    titulo          VARCHAR(200) NOT NULL,
    año_publicacion INT,
    id_autor        INT NOT NULL REFERENCES autores(id_autor)
);
CREATE TABLE

biblioteca=# CREATE TABLE prestamos (
    id_prestamo         SERIAL PRIMARY KEY,
    id_libro            INT NOT NULL REFERENCES libros(id_libro) ON DELETE CASCADE,
    fecha_prestamo      DATE NOT NULL,
    fecha_devolucion    DATE,
    usuario_prestatario VARCHAR(100) NOT NULL
);
CREATE TABLE
```

4. Inserción de datos  
   1. Insertar al menos 5 autores, 8 libros y 5 préstamos de ejemplo.

```sql
biblioteca=# INSERT INTO autores (nombre, nacionalidad) VALUES
('Gabriel García Márquez', 'Colombiana'),
('Miguel de Cervantes', 'Española'),
('Jorge Luis Borges', 'Argentina'),
('Isabel Allende', 'Chilena'),
('Benito Pérez Galdós', 'Española');
INSERT 0 5
biblioteca=# SELECT * FROM autores;
 id_autor |         nombre         | nacionalidad
----------+------------------------+--------------
        1 | Gabriel García Márquez | Colombiana
        2 | Miguel de Cervantes    | Española
        3 | Jorge Luis Borges      | Argentina
        4 | Isabel Allende         | Chilena
        5 | Benito Pérez Galdós    | Española
(5 rows)

biblioteca=# INSERT INTO libros (titulo, año_publicacion, id_autor) VALUES
('Cien años de soledad', 1967, 1),
('El amor en los tiempos del cólera', 1985, 1),
('Don Quijote de la Mancha', 1605, 2),
('Novelas ejemplares', 1613, 2),
('Ficciones', 1944, 3),
('El Aleph', 1949, 3),
('La casa de los espíritus', 1982, 4),
('Fortunata y Jacinta', 1887, 5);
INSERT 0 8
biblioteca=# SELECT * FROM libros;
 id_libro |              titulo               | año_publicacion | id_autor
----------+-----------------------------------+-----------------+----------
        1 | Cien años de soledad              |            1967 |        1
        2 | El amor en los tiempos del cólera |            1985 |        1
        3 | Don Quijote de la Mancha          |            1605 |        2
        4 | Novelas ejemplares                |            1613 |        2
        5 | Ficciones                         |            1944 |        3
        6 | El Aleph                          |            1949 |        3
        7 | La casa de los espíritus          |            1982 |        4
        8 | Fortunata y Jacinta               |            1887 |        5
(8 rows)

biblioteca=# INSERT INTO prestamos (id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario) VALUES
(1, '2026-09-01', NULL,         'Ana López'),
(3, '2026-09-03', '2026-09-10', 'Carlos Pérez'),
(1, '2026-08-20', '2026-09-01', 'Carlos Pérez'),
(5, '2026-09-10', NULL,         'Lucía Gómez'),
(7, '2026-09-15', '2026-09-22', 'Ana López'),
(1, '2026-07-01', '2026-07-15', 'Lucía Gómez'),
(3, '2026-08-01', '2026-08-12', 'Ana López'),
(6, '2026-09-20', NULL,         'Pedro Ruiz');
INSERT 0 8
biblioteca=# SELECT * FROM prestamos;
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           1 |        1 | 2026-09-01     |                  | Ana López
           2 |        3 | 2026-09-03     | 2026-09-10       | Carlos Pérez
           3 |        1 | 2026-08-20     | 2026-09-01       | Carlos Pérez
           4 |        5 | 2026-09-10     |                  | Lucía Gómez
           5 |        7 | 2026-09-15     | 2026-09-22       | Ana López
           6 |        1 | 2026-07-01     | 2026-07-15       | Lucía Gómez
           7 |        3 | 2026-08-01     | 2026-08-12       | Ana López
           8 |        6 | 2026-09-20     |                  | Pedro Ruiz
(8 rows)
```

5. Consultas básicas  
   1. Listar todos los libros con su autor correspondiente.  
   2. Mostrar los préstamos que aún no tienen fecha de devolución.  
   3. Obtener los autores que tienen más de un libro registrado.

```sql
biblioteca=# SELECT nombre,titulo FROM autores NATURAL JOIN libros;
         nombre         |              titulo
------------------------+-----------------------------------
 Gabriel García Márquez | Cien años de soledad
 Gabriel García Márquez | El amor en los tiempos del cólera
 Miguel de Cervantes    | Don Quijote de la Mancha
 Miguel de Cervantes    | Novelas ejemplares
 Jorge Luis Borges      | Ficciones
 Jorge Luis Borges      | El Aleph
 Isabel Allende         | La casa de los espíritus
 Benito Pérez Galdós    | Fortunata y Jacinta

biblioteca=# SELECT id_prestamo,titulo FROM libros NATURAL JOIN prestamos WHERE fecha_devolucion IS NULL;
 id_prestamo |        titulo
-------------+----------------------
           1 | Cien años de soledad
           4 | Ficciones
           8 | El Aleph
(3 rows)

biblioteca=# SELECT a.nombre, COUNT(*) AS num_libros
FROM autores a JOIN libros l ON l.id_autor = a.id_autor
GROUP BY a.id_autor, a.nombre
HAVING COUNT(*) > 1;
         nombre         | num_libros
------------------------+------------
 Jorge Luis Borges      |          2
 Miguel de Cervantes    |          2
 Gabriel García Márquez |          2
(3 rows)
```

6. Consultas con agregación  
   1. Calcular el número total de préstamos realizados.  
   2. Obtener el número de libros prestados por cada usuario.

```sql
biblioteca=> SELECT COUNT(*) AS total_prestamos FROM prestamos;
 total_prestamos
-----------------
               8

biblioteca=> SELECT usuario_prestatario, COUNT(*) AS total_prestamos
FROM prestamos
GROUP BY usuario_prestatario;
 usuario_prestatario | total_prestamos
---------------------+-----------------
 Pedro Ruiz          |               1
 Lucía Gómez         |               2
 Carlos Pérez        |               2
 Ana López           |               3
(4 rows)
```

7. Modificación de datos  
   1. Actualizar la fecha de devolución de un préstamo pendiente.  
   2. Eliminar un libro y comprobar el efecto en la tabla de préstamos (usar ON DELETE CASCADE o justificar el comportamiento).

```sql
biblioteca=> UPDATE prestamos SET fecha_devolucion = CURRENT_DATE WHERE id_prestamo = 1;
UPDATE 1
biblioteca=> SELECT * FROM prestamos WHERE id_prestamo = 1;
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           1 |        1 | 2026-09-01     | 2026-09-28       | Ana López
(1 row)

biblioteca=> SELECT * FROM prestamos WHERE id_libro = 3;
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
           2 |        3 | 2026-09-03     | 2026-09-10       | Carlos Pérez
           7 |        3 | 2026-08-01     | 2026-08-12       | Ana López
(2 rows)

DELETE FROM libros WHERE id_libro = 3;
DELETE 1

SELECT * FROM prestamos WHERE id_libro = 3;
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario
-------------+----------+----------------+------------------+---------------------
(0 rows)
```

Se observa que la selección pasa de tener 2 filas a 0 filas porque la clave *prestamos.id\_libro* está definida con *ON DELETE CASCADE*, así que al eliminar el libro se borran automáticamente sus préstamos.

8. Creación de vistas  
   1. Crear una vista llamada vista\_libros\_prestados que muestre: título del libro, autor y nombre del prestatario.  
   2. Conceder permisos de consulta sobre esta vista únicamente a usuario\_biblio.

```sql
biblioteca=> CREATE VIEW vista_libros_prestados AS
SELECT l.titulo,
       a.nombre AS autor,
       p.usuario_prestatario AS prestatario
FROM prestamos p
JOIN libros l  ON l.id_libro = p.id_libro
JOIN autores a ON a.id_autor = l.id_autor;
CREATE VIEW

biblioteca=> GRANT SELECT ON vista_libros_prestados TO usuario_biblio;
GRANT
```

9. Funciones y consultas avanzadas  
   1. Crear una función que reciba el nombre de un autor y devuelva todos los libros escritos por él.  
   2. Crear una consulta que devuelva los tres libros más prestados.

```sql
biblioteca=>  CREATE OR REPLACE FUNCTION libros_por_autor(p_nombre VARCHAR)
RETURNS TABLE (titulo VARCHAR) AS $$
    SELECT titulo
    FROM libros
    NATURAL JOIN autores
    WHERE nombre ILIKE '%' || p_nombre || '%';
$$ LANGUAGE sql;
CREATE FUNCTION
biblioteca=> SELECT libros_por_autor('Jorge Luis Borges');
 libros_por_autor
------------------
 Ficciones
 El Aleph
(2 rows)

biblioteca=> SELECT titulo, COUNT(*) AS veces_prestado
FROM prestamos NATURAL JOIN libros
GROUP BY id_libro, titulo
ORDER BY veces_prestado DESC
LIMIT 3;
        titulo        | veces_prestado
----------------------+----------------
 Cien años de soledad |              3
 Ficciones            |              1
 El Aleph             |              1
(3 rows)
```

10. Exportación e importación de datos  
    1. Exportar el contenido de la tabla libros a un archivo CSV.  
    2. Importar datos adicionales de autores desde un archivo CSV externo.

```sql
biblioteca=> \copy libros TO '~/libros.csv' WITH (FORMAT csv, HEADER)
COPY 7

biblioteca=> \copy autores (nombre, nacionalidad) FROM '~/autores_nuevos.csv' WITH (FORMAT csv, HEADER)
COPY 3
biblioteca=> SELECT * from autores;
 id_autor |         nombre         | nacionalidad
----------+------------------------+--------------
        1 | Gabriel García Márquez | Colombiana
        2 | Miguel de Cervantes    | Española
        3 | Jorge Luis Borges      | Argentina
        4 | Isabel Allende         | Chilena
        5 | Benito Pérez Galdós    | Española
        6 | Julio Cortázar         | Argentina
        7 | Mario Vargas Llosa     | Peruana
        8 | Rosalía de Castro      | Española
(8 rows)
```

