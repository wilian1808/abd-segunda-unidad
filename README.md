# PREGUNTAS SEGUNDA UNIDAD


## PRIMERA ACTIVIDAD
Implementar un procedimiento almacenado que inserte un pedido en base de datos *Northwind*

- Esta actividad esta desarrollada en **Mysql**.

```sql
USE northwind;

CREATE OR REPLACE PROCEDURE create_new_order(
    IN customer_id VARCHAR (5),
    IN employee_id INT (11),
    IN ship_via INT (11),
    IN freightt DECIMAL(19,4),
    IN product_id INT (11),
    IN quantityy INT (11)
)
BEGIN
    SELECT 
        @ship_name := CompanyName, 
        @ship_address := Address, 
        @ship_city := City, 
        @ship_postal_code := PostalCode,
        @ship_country := Country 
    FROM customers WHERE CustomerID = customer_id;
                                              |
    INSERT INTO orders (
        CustomerID, 
        EmployeeID, 
        OrderDate, 
        RequiredDate, 
        ShippedDate, 
        ShipVia, 
        Freight, 
        ShipName, 
        ShipAddress, 
        ShipCity, 
        ShipPostalCode, 
        ShipCountry
    ) VALUES (
        customer_id,
        employee_id,
        CURRENT_TIME,
        CURRENT_TIME,
        CURRENT_TIME,
        ship_via,
        freightt,
        @ship_name,
        @ship_address,
        @ship_city,
        @ship_postal_code,
        @ship_country
    );

    SELECT @order_id := OrderID FROM orders ORDER BY OrderID DESC LIMIT 1;
    SELECT @unit_price := UnitPrice FROM products WHERE ProductID = product_id;

    INSERT INTO order_details (
        OrderID,
        ProductID,
        UnitPrice,
        Quantity
    ) VALUES (
        @order_id,
        product_id,
        @unit_price,
        quantityy
    );
END
```

> Para comprobar que funciona insertamos un nuevo pedido usando nuestro procedimiento almacenado al cual devemos pasar como parametro los siguientes datos: *id del cliente*, *id del envio*, *el número de carga*, *el id del producto* y la *cantidad del producto*.

> Despues procedemos realizar las consultas para comprobar que si se creo el nuevo pedido y tambien se crearon los detalles del pedido.

```sql
CALL create_new_order("ALFKI", 5, 2, 20.2222, 44, 22);

SELECT 
	OrderID, 
	CustomerID, 
	EmployeeID, 
	OrderDate, 
	ShipVia, 
	Freight, 
	ShipName, 
	ShipAddress, 
	ShipCity, 
	ShipPostalCode 
FROM orders ORDER BY OrderID DESC LIMIT 1;

SELECT * FROM order_details ORDER BY OrderID DESC LIMIT 1;
```

**Resutaldo** de la consulta a la tabla de los pedidos *orders*.

| OrderID | CustomerID | EmployeeID | OrderDate           | ShipVia | Freight | ShipName            | ShipAddress   | ShipCity | ShipPostalCode |
| ------- | ---------- | ---------- | ------------------- | ------- | ------- | ------------------- | ------------- | -------- | -------------- |
|   11080 | ALFKI      |          5 | 2020-10-15 20:57:48 |       2 | 20.2222 | Alfreds Futterkiste | Obere Str. 57 | Berlin   | 12209          |

**Resultado**

| OrderID | ProductID | UnitPrice | Quantity | Discount |
| ------- | --------- | --------- | -------- | -------- |
|   11080 |        44 |   19.4500 |       22 |        0 |

---

## SEGUNDA ACTIVIDAD
Implementar una transacción en la base de datos *AdventureWorks* y explque.

- Esta actividad esta desarrollada en **SQLServer**.

> Primero usaremos la base de datos y procederemos a crear nuestra *transaccion* dentro de un procedimiento almacenado, lo que haces este procedimiento es recibiri cuatro parametros los cuales son: *PersonType*, *FirstName*, *MiddleName*, *LastName* luego procederemos a definir nuestra transacción para ello declaramos dos variables con **declare** los cuales son *IDBusiness* y *RowGuid*.

> Luego procedemos a ejecutar las consultas que se realizaran dentro de la transacción, el primero obtiene un indice unico apoyandose de la funcion *NEWID()* y lo guarda en la variable *@RowGuid*, el segundo insertar o crea un nuevo negocio con el *RowGuid*, el tercero obtenemos el *BusinessEntityID* de la tabla del mismo nombre y lo guardamos en  *@IDBusiness*, el cuarto insertamos o creamos una nueva persona haciendo uso de los parametros que recibe el procedimiento almacenado y tambien de la variable *@IDBusiness*.

> Si existe algun tipo de error sera evaluado con la ayuda de *IF* si no ocurrio ningun error pintaremos en pantalla el mensaje *negocio y persona creado con exito* y procedera a cerrar la transaccion si por el contrario ocurre un error pintara en pantalla el mensaje *error al crear el negocio y la persona* y procedera a anular todo lo que se realizo dentro la transaccion con ayuda de *Rollbakc tran*.

```sql
USE AdventureWorks2019;

CREATE PROCEDURE AddNewPersonBusiness (
    @PersonType CHAR(100),
    @FirstName VARCHAR(100),
    @MiddleName VARCHAR(100),
    @LastName VARCHAR(100)) 
AS
BEGIN TRAN ANPB
	DECLARE @IDBusiness INTEGER
	DECLARE @RowGuid UNIQUEIDENTIFIER

	SELECT @RowGuid =  NEWID()
	INSERT INTO Person.BusinessEntity (rowguid) values (@RowGuid)
	SELECT @IDBusiness = BusinessEntityID FROM Person.BusinessEntity WHERE rowguid = @RowGuid
	INSERT INTO Person.Person (BusinessEntityID, PersonType, FirstName, MiddleName, LastName)
		VALUES (@IDBusiness, @PersonType, @FirstName, @MiddleName, @LastName);
	
    IF @@ERROR = 0
	BEGIN
		PRINT 'NEGOCIO Y PERSONA CREADO CON EXITO'
		COMMIT TRAN ANPB
	END
	ELSE
	BEGIN
		PRINT 'ERROR AL CREAR EL NEGOCIO Y LA PERSONA'
		ROLLBACK TRAN ANPB
	END
GO
```

> Procederemos a usar el procedimiento que contiene la transacción, y tambien realizamos una consulta para ver que se creo en negocio y la persona.

```sql
EXEC AddNewPersonBusiness 'EM', 'Mario', 'Juan', 'Pedro'
GO

SELECT TOP 1 * FROM Person.BusinessEntity ORDER BY BusinessEntityID DESC;

SELECT TOP 1 
    BusinessEntityID, CONCAT(FirstName, ' ', MiddleName, ' ', LastName) AS fullname 
    FROM Person.Person ORDER BY BusinessEntityID DESC;
```

**Resutaldo** del uso de la transacción:

|              response             |
| --------------------------------- |
|NEGOCIO Y PERSONA CREADO CON EXITO | 


**Resultado de las consultas**

| BusinessEntityID |                rowguid               |       ModifiedDate      |
| ---------------- | ------------------------------------ | ----------------------- |
|      20784       | 74FB2A7D-6694-4DD1-8C07-364D917688F6 | 2020-10-14 16:50:58.093 |

| BusinessEntityID |     fullname     |
| ---------------- | ---------------- |
|      20784       | Mario Juan Pedro |

---

## TERCERA ACTIVIDAD
Implemente un trigger en la base de datos de su preferencia y explique.

- Esta actividad esta desarrollada en **SQlServer**.

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