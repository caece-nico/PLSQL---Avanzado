# Cache PLSQL

1. [Introduccion](Introduccion)
2. [Result Cache](Result Cache)
    + [Result Cache Mode](Result Cache Mode)
    + [Paquete DBMS_RESULT_CACHE](Paquete dbms_result_cache)
3. [HINT result_cache](HINT result_cache)
4. [Cache en funciones](Cache en funciones)


## Introduccion.

2. Result Cache.

```
Es una opción de la BD Oracle que permiten almacenar un resultado en memoria.
Funciona para funciones o consultas con poco cambios.
```

### Cómo funciona?

Oracle tiene una zona que se llama __SGA__ System Global Area. Es un lugar de momeria que almacena los datos que se consultan de forma frecuente. Es una tarea del administrador determinar el espacio o capacidad de la misma.

**RESULT_CACHE_MAX_SIZE** Incrementar este valor le puede quitar recursos a otras cosas.

### Ejemplo

+ 1. Crear una conexion admin para poder setear el valor de max_cache

```sql
show parameter cache
```

|parametro|valor|observacioens|
|---------|------|-------------|
|result_chache_max_result| 10|
|result_cache_max_size|1020|  es lo que usamos para guardar en la cache|
|result_cache_mode|MANUAL|
|result_cache_remote_expiration|0|

```sql
show SGA
```

Muestra el total de lo que tenemos disponible para Oracle. Es importante tener en cuenta este valor antes de setear la cache.

```sql
select *
from v$parameter --son vistas del sistema.
where name like '%chache;
```

# Incrementar la CACHE.

```sql
ALTER SYSTEM SET result_cache_max_size=100M;
```

## Result cache mode

```
La cache tiene dos modos, puede ser __MANUAL__ o __FORCE__

Si es manual le debemos indicar que lo guarde en cache, si es force lo intenta hacer de forma automatica.
Lo ideal es que no sea force.
```

```sql
ALTER SYSTEM SET result_cache_mode='MANUAL';
```


## DBMS_RESULT_CACHE

```sql
--para saber si la cache está activa  o no hacemos:
set serveroutput on;
begin
    dbms_output.put_line(dbms_result_cache.status);
end;

-- Si necesitamos ver mas detalle hacemos

set serveroutput on;

begin
    dbms_result_cache.memory_report;
    dbms_result_cache.flush; -- este limpia la cache, vacia todo.
end;

begin
    dbms_result_chache.memory_reporte; -- muestra como está la memoria en un momento dado.
end;
```

3. ## Hint result_cache.

```
para usar result cache en una query debemos usar un HINT, que no es mas que una sentencia que se escribe en el select ente /*+ RESULT_CACHE */
```

```sql
select /*+ result_cache*/ avg(salary)
from employee;


--Si acá hacemos un explain plan vemos que va a usar la memoria.
```

#### Cómo saber si estamos usando la CACHE?

```sql
select *
from v$result_cache_statistics;

-- Para ver una zona de cache especifica y poder ver la query que la creo.

select *
from v$result_cache_object
where cache_id = 'id_de_la_cache_del_excecution_plan'


```

4. ## Cache en funciones

```
Es útil guardar el resultado de una funcion en la cache para no tener que volver a ejecutar  una funcion si sabemos que no cambia muy seguido.
```

```sql
CREATE OR REPLACE FUNCTION mi_funcion_cache(DEPARTAMENTO_ID) 
RETURN NUMBER
RESULT_CACHE RELIES_ON (EMPLOYEE) -- con esto le sigo que lo guarde en cache. 
-- Relied indica que depende de la tabla employee , si cambia que invalide la cache.
IS
    NUM_EMPL number;
BEGIN
    select count(*) into num_empl from employee where deparmet_id = DEPARTAMENTO_ID;
    return num_empl;
END;

```

Usamos la funcion.
------------------

```sql
select nombre, mi_funcion_cache(departamento_id) from departamentos;
```