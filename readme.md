# Introducción a PLSQL - Avanzado

1. [Dependencias](Dependencias)
    1. [Objetos invalidos](Objetos Invalidos)
    2. [UTLDTREE](UTLDTREE)
    3. [DBMS_UTILITY](DBMS_UTILITY)
    3. [DBMS_UTILITY.COMPLIE_SCHEMA](DBMS_UTILITY.COMPLIE_SCHEMA)


## Dependencias

Las dependencias indican como se relacionan los objetos entre si y como un cambio en uno puede afectar el funcionamiento de otros.

Podemos ver las dependencias usando las tablas

a. USER_DEPENDENCES


```sql
select *
from user_dependence
where name = 'EMP_DETAILS_VIEW'
```

_Las relaciones del tipo foreign key no se tienen en cuenta en esta tabla_

### Tambien lo podemos hacer en el otro sentido, ver que objetos dependen de otro objeto.

```sql
select *
from USER_DEPENDENCIES
where REFERENCED_NAME = 'MI_TABLA
```

## Objetos Invalidos

Hay una vista que nos permite ver que objetos son ivalidos.

```sql
select *
from USER_OBJECTS
```

_Nos intereza la columna status que puede tener los valores *VALID* o *INVALID*_

### Ejemplo para invalidar un objeto

```sql
CREATE TABLE prueba (C1 NUMBER, C2 NUMBER);

SELECT * FROM USER_OBJECTS WHERE NAME = 'prueba';

CREATE VIEW PRUEBA_VW as SELECT C1 FROM prueba;

ALTER TABLE prueba MODIFY C1  VARCHAR(2000);

--esto invalida la vista pero no da error, sino que es una precaución de ORACLE, cuando la ejecuto se valida de forma automatica.

SELECT *
FROM PRUEBA_VW;
```

_Si por ejemplo eliminamos la columna C1 es estado será invalido pero al intentar ejecutar la vista no podrá solucionarlo porque la columna ya no existe._

### Ejemplo de dependencias secundarias

```sql
CREATE PROCEDURE proc1 is
begin
    null;
    xxx; -- invalidado
end;

CREATE PROCEDURE proc2 is
begin
    NULL;
end;

SELECT *
from USER_DEPENDENCIES
WHERE name = 'PROC2'; -- deende de proc1

SELECT *
from USER_OBJECTS
where OBJECT_NAME like 'proc%'; 

--ambos objetos están invalidos.
--Proc1 porque tiene un error de sintaxis
--Proc2 porque depende de Proc1
```


_Cuando sulucione **proc1** este pasa a valido pero quedará invalido **proc2**. Esto se puede solucionar solo o lo podemos hacer notros **Esto es lo ideal**_

```sql
ALTER PROCEDURE proc2 COMPILE;
```

```sql
ALTER FUNCTION mi_funcion COMPILE;
```

### Cómo compilar paquetes invalidos?

Es un poco distinto a como se hace con Funciones y Procedures.

```sql
CREATE OR REPLACE PACKAGE PAQ1 is
    FUNCTION fdx1 RETURN NUMBBER;
    PROCEDURE P1;
    PROCEDURE P2;
END;

CREATE OR REPLACE PACKAGE BODY PAQ1 IS
    FUNCTION fdx1 RETURN NUMBER 
    IS
        BEGIN
            NULL;
            RETURN 1;
        END;

    PROCEDURE P1 IS
    BEGIN
        NULL;
    END;

    PROCEDURE P_PRIVADO IS
    IS
        BEGIN
            NULL;
        END;

    END;
    
SELECT * FROM USER_OBJECTS WHERE OBJECT_NAME LIKE 'PAQ1%';
```

_¿Que pasa con este ejemplo?_

Este ejemplo declara un Header con una funcion y dos  procedures. 
Mientras que el Body tiene una función , un procedure (que existe en el header) y un procedure privado (No debe estar en el Header).

**EL PROBLEMA**

El problema es que el procedure del header, P2, no está en el body y este es algo que __invalida el BODY pero no el header__.

```sql
ALTER PACKAGE PAQ1 COMPILE;
ALTER PACKAGE PAQ1 COMPILE BODY;
```


## UTLDTREE - Estructura de las referencias.

_Cómo puedo hacer para ver la estructura de las dependencias en los casos de **Dependencias Indirectas**_

Por ejemplo tengo la tabla __PRUEBA__ y una vista de esa tabla __PRUEBA_X__ y con esa vista creo un procedimiento __PROC1__ 
Este __PROC_1__ tambien depende de la tabla __PRUEBA__ de firma indirecta, pero la tabla __USER_DEPENDENCIES__ no me lo muestra.

```sql
CREATE TABLE PRUEBA (COL1 NUMBER, COL2 NUMBER);

CREATE OR REPLACE VIEW PRUEBA_X as 
SELECT * FROM PRUEBA;

CReATE OR REPLACE PROCEDURE PROC1 is
v_index NUMBER;
    begin
        select count(1)
        INTO v_index
        from PRUEBA_X;
    end;
```

**Estructura Jerarquica**

1. Se puede hacer usando querys Jerarquicas (START WITH)
2. Usando el procedimiento.


## DBMS_UTILITY 

Este metodo tiene varias utilidades para usar con dependencias.

### Devuelve una estructura tipo arbol.

```sql
SET SERVEROUTPUT on;
EXECUTE DBMS_UTILITY.GET_DEPENDENCIES('TABLE','HR','PRUEBA');
```

+ TABLE -> es el tipo de objeto
+ HR -> es el esquema
+ PRUEBA -> nombre del objeto

### Valida un objeto si puede.

```sql
EXECUTE DBMS_UTILITY.VALIDATE('HR','PRUEBA', Tipo)
```

+ Tipos posibles.



| tipo | referencia  |
|------|-------------|
|1|TABLA/procedure/TYPE|
|2|BODY|
|3|TRIGGER|
|4|INDEX|
|5|CLUSTER|
|6|LOB|
|7|DIRECTORY|
|8|QUEUE|


__PORQUE HACEMOS ESTO__

Dentro de un bloque PLSQL no se puede usar ALTER, DROP o CREATE
Pero como DBMS_UTILITY es un metodo se puede usar en un BEGIN END;


## DBMS_UTILITY.COMPLIE_SCHEMA

Podemos compilar todo el SCHEMMA.

```sql
DBMS_UTILITY.COMPLIE_SCHEMA('HR', COMPILE_ALL=> FALSE, REUSE_SETTINGS=FALSE);
```

COMPILE_ALL -> Pasando False solo compila lo que es invalido.