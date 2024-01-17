# Triggers de Esquema y de Base de Datos.

1. [Introduccion](Introducción)
2. [Tipo DDL](Tipo DDL)
3. [Tipo DB](Tipo DB)
4. [Tipo System](Type System)



## 1. Introducción.

```
Son triggers que se disparan a nivel de ESQUEMA y de BD.
1. Esquema, se disparan cuando se produce un cambio a nivel de esuqema, como alter, drop, create.
2. Base de Datos, se usan para aditoria de erroes de DB, logeo de usuarios, etc.
```

## 2. Tipo DDL

Ocurren a nivel de Esquema de la DB.


|tipo|descripcion|
|-----|----------|
|alter|Cada vez que se modifica un objeto|
|audit|Cuando se recolectan estadisticas|
|drop|al borrar un objeto|
|create|Al crear un objeto|
|comment|Al agregar un comentario|
|revoke-grant|Al otorgar o negar permisos|
|DDL| todos los anteriores|



```sql
CREATE OR REPLACE TRIGGER DDL1
ALFER/BEFORE grant or revoke
on schema ORA
begin
    DO SMT;
end;
```
* importante: __Que difenrencia hay entre un trigger de tipo BEFORE y AFTER?__

Los de tipo __BEFORE__ nos permiten abortar el proceso si no queremos que termine, los de tipo __AFTER__ no, ya se hicieron.

Otro ejemplo

```sql
CREATE TABLE control_log
(
    evento varchar2(2500);
    fecha date dafault sysdate;
);


create or replace trigger borrar_objeto
after drop
on schema
begin

    INSERT into control_log('Hemos borrado un objeto');
    commit;

end;

/
```

__Cómo recuperar informacion o atributos relacionados al objeto que se vió afectado__

|tipo|desc|
|-----|----|
|ora_sysevent|evento que dispara el trigger|
|ora_login_user|Usario que ha hecho el login|
|ora_instance_num|id de la instancia de bd|
|ora_database_name|nombre de la db|
|ora_dict_obj_type|tipo de obj que ha disparado el evento|
|ora_dict_obj_name|nombre del objeto|
|ora_dict_obj_owner|propietario del obj|


```sql
CREATE OR REPLACE TIRGGER borrar_objeto
AFTER DROP
on SCHEMA
bEGIN
    INSERT INTO CONTROL_LOG values
    ('Se ha borrado' || ORA_DICT_OBJ_NAME || ' del tipo '|| ORA_DICT_OBJ_TYPE);
    commit;
end;
```



## 3. Tipo DB

SOn triggers que estan a nivel de la BD, es necesario tener permisos de administrador.

```sql
create or replace trigger borrar_objeto
after drop
on databse
begin
    insert into control_log values
    ('Este es un trigger a nival de bd');
    commit;
end;
```

El primer trigger solo sirve para el usuario que lo crea pero el segundo abarca a todos los uaurios dentro de una bd que hayan hecho de __DROP__

## 4. Tipo System

Son triggers que se disparan no sobre objetos de la db sino sobre acciones que ocurren a nivel del motor de ORACLE.

|tipo|descp|
|----|------|
|AFTER STARTUP|CUando se abre la BD|
|BEFORE SHOUTDOWN|Antes de cerrar la bd|
|AFTER SERVERERROR| cuando se produce un error de la db, como no data found, etc|
|AFTER LOGON| CUando un usuario se logea|
|BEFORE LOGOFF| Cuando se desconecta|
|AFTER SUSPEND| Cuando se suspende una transaccion|


```sql
create table control_log
(
    usuario varchar2(2500),
    ip varchar2(20),
    fecha date
);

create or replace trigger loging
after logon
on database
begin
    insert into loging values
    (
        ora_loging_user, ora_ip_address, sysdate
    );
    commit;
end;

```

__Funciones importantes para logear errores del system con triggers__

|tipo|desc|
|----|----|
|ora_sever_error|codigo de error|
|ora_server_error_depth|son mas de un error|
|ora_server_error_msg( position in binary_integer)|textos de los errores|
|ora_sql_text (sql_text out ora_name_list_t)|que comandos sql han fallado|
|ora_is_servererror||
|ora_space_error_info||


__EJEMPLO__

```sql
create or replace trigger logeo
after servererror
on database
declare
    sql_text ora_name_list_t; -- ya viene creada por oracle
    mensaje varchar2(5000);
    comando varchar2(2500);
begin
    for x in 1..ora_server_error_depth loop
        mensaje := mensaje || ora_server_error(x);
    end loop

        for i in 1..ora_sql_text(sql_text) loop
            comando := comando || sql_text(i);
        end loop;

    --insertamos las variables.
    --commit;
end ;

```
