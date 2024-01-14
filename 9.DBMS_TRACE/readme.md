# DBMS_TRACE

1. [Introducción](Introduccion)
2. [Crear_Traza](Crear_Traza)
3. [Tipos de traza](Tipos de traza)
4. [Activar una traza](Activar una traza)
5. [Analisis de la traza](Analisis de la traza)


1. ## Introducción.

```
La finalidad es poder hacer el seguimiento de un programa de pl/sql.
1. Activar la Traza
2. Determinar el nivel de traza.
3. Parar la traza e interpretar el resultado.
```

2. ## Crear Traza

Primero debemos crear las tablas, estas ya existen dentro de un directorio llamadao dbms --> admin.
generalmente está en -> __cd product/18c/dbhomeXE/rdbms/admin/tracetab.sql__

```sql
-- estas tablas se crean como usuario sys.
--plsql_trace_events
--plsql_tarce_runs
```

### Crear un procedure que sea traceable.

```sql
CREATE PROCEDURE proc1
is
    v number:=0;
BEGIN
    dbms_output.put_line(v);

    select count(*)
    into v
    from empleados;

    dbms_output.put_line(v);
end;
/

ALTER PROCEDURE proc1 COMPILE DEBUG; -- para poder tracear.

SELECT *
FROM user_plsql_object_settings
where NAME = 'PROC1'; -- está en modo debug.

ALTER PROCEDURE proc1 COMPILE; -- para dejar de trasear.
```

3. ## Tipos de traza.

Existen distintos niveles de tipos de tarza con valores constantes.


|constante|valor|
|---------|------|
|calls|
|TRACE_ALL_CALLS|1|
|TRACE_ENABLED_CALLS|2|
|exceptions|
|TRACE_ALL_EXCEPTIONS|4|
|TRACE_ENABLED_EXCEPTIONS|8|
|sql|
|TRACE_ALL_SQL|32|
|TRACE_ENABLED_SQL|64|
|lines|
|TRACE_ALL_LINES|128|
|TRACE_ENABLED_LINES|256|

## 4.  Activar una traza

1. modifico el procedure en modo debug.
2. activo la traza.
3. ejecuto el procedure.
4. detengo la traza.

```sql
EXECUTE DBMS_TRACE.SET_PLSQL_TRACE(
    DBMS_TRACE.TRACE_ENABLED_SQL +
    DBMS_TRACE.TRACE_ALL_CALLS
);

-- eN ESTE EJEMPLO ACTIVAMOS DOS TRAZAS, LO HACEMOS USANDO EL SIGNO +. TAMBIEN PODEMOS USAR EL VALOR NUMERICO, PERO HACERLO POR REFERENCIA ES MEJOR.

ALTER PROCEDURE PROC1 COMPILE DEBUG;

execute proc1;

-- EJECUTAMOS LA TRAZA DE TODO LO QUE VENGA DEBAJO.

EXECUTE DBMS_TRACE.CLEAR_PLSQL_TRACE;

```

## 5. Analisis de la traza.

Como las tablas de trazas están en sys, debemos logearnos como admin para poder verlas.

```sql
GRANT SELECT ON PLSQL_TRACE_RUNS to PUBLIC; -- para poder verlas desde otros usuarios.

select *
from sys.plsql_trace_events
where event_unit = 'PROC1';
```

