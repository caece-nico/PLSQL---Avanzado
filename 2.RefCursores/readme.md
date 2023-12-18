# Introducción a SYSREF CURSOR

1. [Introducción](Introduccion)
2. [Creacion_de_cursor_REF](Creacion de un cursor REF)
3. [Bucles_en_cursores_ref](Bucles en cursores ref)
4. [ref_cursor_con_tipos](Ref cursor con tipos)
5. [ref_cursors_y_funciones](ref cursores y funciones)
6. [Compartir_cursores](Compartir Cursores)
7. [sysref cursor](sys ref cursor)

## Introduccion

Se llaman cursores variables o REF CURSORS.
Son referencias o punteros a otros cursores, no tienen datos sino que apuntan a otros cursores.



__Es una variable del tipo Cursor__

__VENTAJAS__ Al ser variables y dinamicos pueden apuntar primero a una consulta y luego a otra.

## Creacion de un cursor REF

+ creacion de un cursor normal.

```sql
DECLARE
    CURSOR C1 is SELECT nombre FROM mi_tabla;
BEGIN
    NULL;
END;    
```
+ Declaracion de un cursor REF

```sql
SET SERVEROUTPUT on

DECLARE

    TYPE CURSOR_VARIABLE IS REF CURSOR;
    V1 CURSOR_VARIABLE;
    X1 CURSOR_VARIABLE; 
    ...
    ...
    ...

    V_EMPLEADOS EMPLOYEE%ROWTYPE;
    v_DEPARTAMENTO DEPARTMENT%ROWTYPE;

BEGIN

    OPEN V1 FOR SELECT * FROM EMPLOYEE;

    FETCH V1 INTO V_EMPLEADOs;
    dbms_output.put_line(V_EMPLEADOS.salary);
    CLOSE V1;

    OPEN V1 FOR SELECT * FROM DEPARTMENT;

    FETCH V1 INTO v_DEPARTAMENTO;
    dbms_output.put_line(V_EMPLEADOS.DEPARTMENT_NAME);
    CLOSE V1;
```

+ Acá se vé una ventaja de los REF CURSOR porque una vez que termino con una query puede reutilizarlo y seguir con la próxima, pero debo tener en cuenta lo que devuelve. Esto es porque son __CURSORES DINAMICOS__


**IMPORTANTE** Notar que las variables de tipo cursor no están tipadas lo que quiere decir que pueden recibir cualquier tipo de datos.

## Bucles en cursores REF.

```sql
DECLARE

    TYPE CURSOR_VARIABLE is REF CURSOR;
    V1 CURSOR_VARIABLE;

    V_DEPARTMENT departamento%rowtype;


BEGIN

    OPEN V1 for SELECT * FROM DEPARTMENT;

    FETCH V1 INTO V_DEPARTEMT;
    WHILE V1%found loop
        DBMS_OUTPUT.PUT_LINE(v1.department_name);
        fetch V1 into V_DEPARTMENT;
    END LOOP;

END;

```


+ Mas adelante veremos como hacer llamadas a __funciones__ o __porcedures__ y los __sysref cursores__. Estos últimos se usan mucho con las __colecciones__.


## Ref cursor con tipos.

+ Podemos crear un cursor ref cursor con un tipo especifico.

```sql
set serveroutput on;

DECLARE
    TYPE CURSOR_REF IS REF CURSOR RETURN EMPLOYEE%ROWTYPE;

    V1 CURSOR_REF;
    V_DEPARTAMENTOS DEPARTMENTS%ROWTYPE;
BEGIN

    OPEN V1 FOR SELECT * FROM DEPARTMENTS;

    FETCH  V1 INTO V_DEPARTAMENTOS;
    WHILE V1%FOUND LOOP
        DBMS_OUTPUT.PUT_LINE(V1.DEPARTMENT_NAME);
        FETCH  V1 INTO V_DEPARTAMENTOS;
    END LOOP;

    CLOSE V1;

END;

```

+ Se utiliza como restricción para no poner cualquier cosa dentro de un cursor.

## Ref cursores y funciones.

+ Podemos utilizarlos como cualquier otra variable dentro de una función.

```sql
CREATE OR REPLACE PACKAGE PAQ1
AS
    type c_variable is ref cursor;
    function devolver_datos (c1 IN OUT c_variable, x NUMBER) return varchar2;

 END;   

 CREATE OR REPLACE PACKAGE BODY PAQ1 AS 
    FUNCTION devolver_datos (C1 IN OUT C_VARIABLE, X NUMBER) RETURN varchar2
    IS
        DEPARTAMENTOS DEPARTENTS%ROWTYPE;
        EMPLEADOS EMPLOYEE%ROWTYPE;

    BEGIN

        IF X = 1 THEN

            OPEN C1 FOR SELECT * FROM EMPLOYEES;
            FETCH C1 INTO EMPLEADOS;
            RETURN EMPLEADOS.FIRST_NAME;

        ELSE

            OPEN C1 FOR SELECT * FROM DEPARTMENTS;
            FETCH C1 INTO DEPARTAMENTOS;
            RETURN DEPARTAMENTOS.DEPARTMENT_NAME;

        END IF;
END;       
```

__IMPORTANTE__ Notar que no cerramos el cursor dentro de la función.

**EJECUCION**

```sql
SET SERVEROUTPUT on;
DECLARE

    DATOS PAQ1.C_VARIABLE;

BEGIN

    DBMS_OUTPUT.PUT_LINE((PAQ1.DEVOLVER_DATOS(DATOS, 1)));

END;

```

## Compartir Cursores

```sql
SET SERVEROUTPUT ON
DECLARE

type cursor_type is ref cursor;
v1 cursor_type;
v2 cursor_type;

V_EMPLEADOS EMPLOYEE%ROWTYPE;

BEGIN

    OPEN  V1 FOR SELECT * FROM EMPLOYEES;
    FETCH V1 INTO V_EMPLEADOS;
    DBMS_OUTPUT.PUT_LINE(V1.EMPLOYEE_NAME);

    V2 := V1; -- v2 va a apuntar a ala misma posicion que v1, como luego le hacemos un fetch apuntará a la siguiente fila, al empleado siguiente.

    FETCH V2 INTO V_EMPLEADOS;
    DBMS_OUTPUT.PUT_LINE(V2.EMPLOYEE_NAME);

    CLOSE V1;

    CLOSE V2; -- No hace falta cerrarlo

END;
```


## Sysref cursor.

+ Es un cursor o variable ya predefinida que es del tipo __refcursor__ integrado dentro del paquete estandar.
+ Si es __REFCURSOR__ primero teniamos que hacer el TYPE y luego declarar el cursor.
+ Con el __sysref cursor__ ya está recreado en la BD.


```sql

DECLARE
-- El sys_refcursor no sirve cuando tenemos un return 
V1 SYS_REFCURSOR;

v_EMPLEADOS EMPLOYEE%ROWTYPE;

BEGIN

    OPEN V1 FOR SELECT * FROM EMPLOYEE;

    FETCH V1 INTO V_EMPLEADO;
    DBMS_OUTPUT.PUT_LINE(V1.EMPLOYEE_NAME);

    CLOSE V1;

END;


```

**IMPORTANTE** El sys_refcursor solo se puede usar para declarar un cursor del tipo __ref cursor__ no se puede usar si queremos usar una clausula RETURN.

