## PREGUNTAS SEGUNDA UNIDAD


#### PRIMERA ACTIVIDAD
Implementar un procedimiento almacenado que inserte un pedido en base de datos *Northwind*

```sql

```

#### SEGUNDA ACTIVIDAD
Implementar una transacción en la base de datos *AdventureWorks* y explque.

```sql

```

#### TERCERA ACTIVIDAD
Implemente un trigger en la base de datos de su preferencia y explique.


> Primero crearemos la base de datos **Escuela** y procederemos a usarla, tambien crearemos la tabla llamada **Estudiantes** con sus respectivas columnas, cada estudiante tendra una columna llamada *Nota* y *Estado* el *Estado* esta definido por defecto como desaprobado y la *Nota* por defecto es 0, luego de esto procedemos a insertar datos en nuestra tabla **Estudiantes**.
```sql
CREATE DATABASE Escuela;

USE Escuela;

CREATE TABLE Estudiantes (
	IdEstudiante INTEGER NOT NULL IDENTITY(1, 1),
	Username VARCHAR(100) NOT NULL,
	Nota INTEGER NOT NULL DEFAULT 0,
	Estado VARCHAR(50) NOT NULL DEFAULT 'desaprobado',
	CONSTRAINT Pk_Estudiantes PRIMARY KEY (IdEstudiante),
	CONSTRAINT Ck_Estudiantes CHECK (Nota >= 0 AND Nota <= 20)
);

INSERT INTO Estudiantes(Username) VALUES ('mario'), ('juan'), ('pepe'), ('oscar'), ('pedro');

```

> Ahora procederemos a crear nuestro *trigger* lo que este *trigger* realizara sera que si el alumno consigue una nota aprobatoria osea que su nota sea mayor que 11 nuestro *trigger* cambiara el estado de *desaprobado* a *aprobado*, ahora digamos que pusimos una nota aprovatoria por error entonces al momento de volver a actualizar la nota del estudiante tambien volvera a actualizar el estado poneindo *aprobado* si la nota es aprovatoria o desaprobado si la nota es menor a 11.
> 
> Creamos el *trigger* llamado **EstadoEstudiante** para la tabla **Estudiantes** luego veresmo si se realizo algun tipo de *actualizacion* en la columna de la *Nota* si ocurrio una actualización nuestro *trigger* procedera a cambiar el *Estado del estudiante* deacuerdo a si su nota es aprobatoria o desprobatoria y terminamos.

```sql
CREATE TRIGGER EstadoEstudiante
    ON Estudiantes
AFTER UPDATE AS
    IF UPDATE(Nota)
BEGIN
    UPDATE Estudiantes SET Estado = 'aprobado' WHERE Nota >= 11;
    UPDATE Estudiantes SET Estado = 'desprobado' WHERE Nota < 11;
END;
```
> Primero veremos el contenido de nuestra tabla **Estudiantes** para luego realizar una actualización de las notas de dos estudiantes mediante *update* y para ver que nuestro *trigger* se ejecuto correctamente volveremos a consultar el contenido de la tabla **Estudiantes** y veresmo que el estado cambio automaticamente de acuerdo a la nota que le asignamos al estudiante.

```sql
SELECT * FROM Estudiantes;

UPDATE Estudiantes SET Nota = 16 WHERE IdEstudiante = 2;
UPDATE Estudiantes SET Nota = 12 WHERE IdEstudiante = 4;

SELECT * FROM Estudiantes;
```

**Resultado** de la primera consulta.

|IdEstudiante | Username  | Nota |    Estado   |
| ----------- | --------- | ---- | ----------- | 
|      1      |   mario   |  0   | desaprobado |
|      2      |   juan	  |  0   | desaprobado |
|      3      |   pepe	  |  0   | desaprobado |
|      4      |   oscar   |  0   | desaprobado |
|      5      |   pedro   |  0   | desaprobado |

**Resultado** despues de realizar el cambio de nota y que nuestro *trigger* haya realizado su trabajo.

|IdEstudiante | Username  | Nota |    Estado   |
| ----------- | --------- | ---- | ----------- | 
|      1      |   mario   |  0   | desaprobado |
|      2      |   juan	  |  16  | aprobado    |
|      3      |   pepe	  |  0   | desaprobado |
|      4      |   oscar   |  12  | aprobado    |
|      5      |   pedro   |  0   | desaprobado |