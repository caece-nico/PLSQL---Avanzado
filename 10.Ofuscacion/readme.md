# Ofuscacion.

1. [Introducción](Introduccion)
2. [Utilidad WRAP](Utilidad WRAP)
3. [dbms_ddl.](dbmd_dll.)

## 1. Introduccion

La ofuscación ayuda a crear pseudocodigo que ayuda a ocultar el código.

Hay  dos formas de hacerlo.

a. utilidad wrap
b. paquete dbms_ddl

POdemos ocultar.

1. codigo de procedimientos
2. funciones y paquetes
3. types

No se puede ofuscar triggers

```
Este codigo no se encrripta, esto quiere decir que no hay forma de revertirlo, si perdemos el codigo fuente no se puede volver atras.
```

## 2. Utilidad WRAP

Es la forma mas directa.

* debemos crear un directorio con un fichero .sql que contiene el procedure que queremos ocultar.

* Nos logeamos en plsql ora/ora vamos al directorio donde está el fichero

```
plsql > wrap mi_sql.sql

Esto nos crea un nuevo fichero ofuscado.

plsql > wrap iname=mi_sql.sql oname=mi_ofus_sql.sql
```


__Como se integra con oracle?__

```
ora/ora

sqlplus > @mi_sql.sql

esto lo crea dentro mi oracle (no ofuscado)

col text format a40 (para formatear)

select name, line, text from user_source where name = 'PROC1';

sqlplus > @mi_ofus_sql.plb

compilamos el codigo ofuscado (no lo ponemos ver y tampoco lo podemos recuperar)

```


## 3. DBMS_DDL

Otro metodo es usando la utilidad de BD __dbms_ddl.create_wrapped__

```sql
begin
    dbms_ddl.wrapper('
    CREATE ORA REPLACE PROCEDURE PROC1
    IS
        NUM NUMBER;
    BEGIN
        NULL;
    END;'
    );

SELECT LINE, TEXT FROM USER_SOURCE WHERE NAME LIKE '%PROC1%';


```

__como lo podemos pasar con una variable?__

```sql
declare
codigo varchar2(1000):='
CREATE OR REPLACE PROCEDURE PROC2
IS 
    BEGIN
        NULL;
    END;';

BEGIN
    DBMS_DDL.WRAPPED(CODIGO);
END;

--- CODIGO MAS GRANDE, HAY QUE USAR UN TIPO TABLA. DBMS_SQL.VARCHAR2

DECLARE
    CODIGO DBMS_SQL.VARCHAR2A;

begin
    codigo(1):='create or replace proc3';
    codigo(2) := 'is begin null; end;';

    dbms_ddl.wrapped(codigo,1,codigo.count);
end;
```