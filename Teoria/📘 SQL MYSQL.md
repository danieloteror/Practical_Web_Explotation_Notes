## 📚 **Índice**

- [[#🛢️ Comandos básicos MariaDB / MySQL]]
- [[#🧱 1. Conceptos Básicos]]
- [[#🟦 2. SELECT – La base de todo]]
- [[#🟧 3. WHERE – Filtrando Datos]]
- [[#🟩 4. ORDER BY – Ordenar resultados]]
- [[#🟪 5. LIMIT / OFFSET – Paginación]]
- [[#🟨 6. Funciones de Agregación]]
- [[#🔷 7. GROUP BY – Agrupar Datos]]
- [[#🟥 8. JOINs – Combinando Tablas]]
- [[#🟫 9. Subconsultas (Subqueries)]]
- [[#🟦 10. UNION / UNION ALL]]
- [[#🔺 11. Manipulación de Datos (DML)]]
- [[#🧰 12. Creación de Tablas (DDL)]]
- [[#🟣 13. Índices]]
- [[#🔐 14. Transacciones (TCL)]]
- [[#🧠 15. Resumen para repaso rápido]]
- [[#🧠 ¿Para qué sirven las subconsultas?]]
- [[#9. Comandos por terminal]]


---

# 🛢️ **Comandos básicos MariaDB / MySQL**

```bash
systemctl status mariadb
sudo systemctl start mariadb
mysql -u root -p
```

---

# 🧱 **1. Conceptos Básicos**

### ¿Qué es SQL?

- Lenguaje para **consultar, manipular y administrar** bases de datos relacionales.
    
- Basado en sentencias declarativas: dices **qué quieres**, no cómo obtenerlo.
    

### Tipos de comandos

|Tipo|Descripción|Ejemplos|
|---|---|---|
|**DQL**|Consulta de datos|`SELECT`|
|**DML**|Manipulación|`INSERT`, `UPDATE`, `DELETE`|
|**DDL**|Definición|`CREATE`, `ALTER`, `DROP`|
|**TCL**|Control de transacciones|`COMMIT`, `ROLLBACK`|
|**DCL**|Control de permisos|`GRANT`, `REVOKE`|

---

# 🟦 **2. SELECT – La base de todo**

## Sintaxis mínima

```sql
SELECT columna1, columna2
FROM tabla;
```

## Seleccionar todas las columnas

```sql
SELECT * FROM tabla;
```

## Alias

```sql
SELECT columna AS alias
FROM tabla;
```

---

# 🟧 **3. WHERE – Filtrando Datos**

```sql
SELECT *
FROM tabla
WHERE condicion;
```

### Operadores útiles

|Tipo|Ejemplo|
|---|---|
|Comparación|=, !=, >, <, >=, <=|
|Rango|`BETWEEN 10 AND 20`|
|Listas|`IN ('A','B','C')`|
|Patrón|`LIKE 'A%'`|
|Nulls|`IS NULL`, `IS NOT NULL`|

---

# 🟩 **4. ORDER BY – Ordenar resultados**

```sql
SELECT *
FROM tabla
ORDER BY columna DESC;
```

---

# 🟪 **5. LIMIT / OFFSET – Paginación**

```sql
SELECT *
FROM tabla
LIMIT 10 OFFSET 20;
```

---

# 🟨 **6. Funciones de Agregación**

|Función|Descripción|
|---|---|
|COUNT()|Cuenta registros|
|SUM()|Suma valores|
|AVG()|Promedio|
|MIN/MAX|Mínimo / Máximo|

```sql
SELECT COUNT(*), AVG(salario)
FROM empleados;
```

---

# 🔷 **7. GROUP BY – Agrupar Datos**

```sql
SELECT departamento, AVG(salario)
FROM empleados
GROUP BY departamento;
```

### HAVING – filtro tras agrupar

```sql
SELECT departamento, COUNT(*)
FROM empleados
GROUP BY departamento
HAVING COUNT(*) > 5;
```

---

# 🟥 **8. JOINs – Combinando Tablas**

## Tabla mental

|JOIN|Qué devuelve|
|---|---|
|INNER|Coincidencias en ambas tablas|
|LEFT|Todos los de izquierda + coincidencias|
|RIGHT|Todos los de derecha + coincidencias|
|FULL|Todo (depende motor)|
|CROSS|Producto cartesiano|

### Ejemplo

```sql
SELECT e.nombre, d.nombre AS departamento
FROM empleados e
INNER JOIN departamentos d
ON e.id_departamento = d.id;
```

---

# 🟫 **9. Subconsultas (Subqueries)**

### En SELECT

```sql
SELECT nombre,
       (SELECT AVG(salario) FROM empleados)
FROM empleados;
```

### En WHERE

```sql
SELECT *
FROM empleados
WHERE salario > (
    SELECT AVG(salario) FROM empleados
);
```

---

# 🟦 **10. UNION / UNION ALL**

## UNION

```sql
SELECT nombre FROM tabla1
UNION
SELECT nombre FROM tabla2;
```

## UNION ALL

```sql
SELECT nombre FROM tabla1
UNION ALL
SELECT nombre FROM tabla2;
```

---

# 🔺 **11. Manipulación de Datos (DML)**

## INSERT

```sql
INSERT INTO tabla (col1,col2)
VALUES ('A',10);
```

## UPDATE

```sql
UPDATE tabla
SET col1 = 'Nuevo'
WHERE id = 5;
```

## DELETE

```sql
DELETE FROM tabla
WHERE id = 5;
```

---

# 🧰 **12. Creación de Tablas (DDL)**

## CREATE TABLE

```sql
CREATE TABLE usuarios (
    id INT PRIMARY KEY,
    nombre VARCHAR(50),
    edad INT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## ALTER TABLE

```sql
ALTER TABLE usuarios
ADD COLUMN email VARCHAR(50);
```

## DROP TABLE

```sql
DROP TABLE usuarios;
```

---

# 🟣 **13. Índices**

```sql
CREATE INDEX idx_nombre ON usuarios(nombre);
```

---

# 🔐 **14. Transacciones (TCL)**

```sql
START TRANSACTION;

UPDATE cuentas SET saldo = saldo - 100 WHERE id = 1;
UPDATE cuentas SET saldo = saldo + 100 WHERE id = 2;

COMMIT; -- o ROLLBACK
```

---

# 🧠 **15. Resumen para repaso rápido**

- SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
    
- JOINs = combinar tablas
    
- Subqueries = lógica dependiente
    
- UNION = combinar tablas verticalmente
    
- DML = datos / DDL = estructura
    

---

# 🧠 **¿Para qué sirven las subconsultas?**

### 1. Filtrar usando valores calculados

```sql
SELECT *
FROM empleados
WHERE salario > (SELECT AVG(salario) FROM empleados);
```

### 2. Reemplazar JOIN simples

```sql
SELECT nombre
FROM empleados
WHERE id_departamento IN (
    SELECT id
    FROM departamentos
    WHERE nombre = 'Ventas'
);
```

### 3. Comparaciones con otras tablas

```sql
SELECT *
FROM productos
WHERE precio > (SELECT MAX(precio) FROM competencia);
```

### 4. Columnas calculadas

```sql
SELECT nombre,
       (SELECT COUNT(*) FROM ventas v WHERE v.id_cliente = c.id)
FROM clientes c;
```

### 5. Tablas virtuales

```sql
SELECT *
FROM (
    SELECT id,salario FROM empleados WHERE activo=1
) AS emp
WHERE salario > 2000;
```

---

# **9. Comandos por terminal**

```
show databases;
use nombre_database;
show tables;
describe user;
```

---

Si quieres **añado:**

🔥 cheatsheet visual  
🔥 sección de SQLi para pentesting  
🔥 funciones avanzadas (WINDOW FUNCTIONS)  
🔥 práctica OSCP con ejemplos reales

¿Quieres extenderla?