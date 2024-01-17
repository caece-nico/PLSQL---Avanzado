# Colecciones

1. [Introduccion](Introducción)
2. [Tipos_de_Collections](Tipos de collections)
    1. [Asociativos](Asociativos)
    2. []

## Introducción.


Las colecciones se diferencian de los Records porque solo almacenan __una fila de distinto tipos__.
En cambio las colecciones almacenan objetos del mismo tipo y pueden ser mas de uno.

Tipos:

+ Arrays Asociativos
+ Nested Tables
+ Varrays

## Tipos de Collections

### 1. Asociativos (index by Tables).

Son colecciones con dos columnas,  __una clave primaria__ de tipo entera o cadena y  __un valor__ que puede ser numerico o cadena.

#### Ejemplo.

```sql
TYPE [nombre] IS TABLE OF [tipo columns] INDEX BY [PLS_INTEGER | BINARY_INTEGER | VARCHAR2(X)
] 
```

```sql
TYPE tt_departamentos is table of DEPARTMENTS.DEPARTMENT_NAME%type
index by PLS_INTEGER;

TYPE tt_empleados is table of EMPLOYEES%ROWTYPE
index by PLS_INTEGER;

t_empleado tt_empleado
t_departamentos tt_departamentos
```

__IMPORTANTE__ En el primer ejemplo creamos una tipo tabla de una columna mientras que en el segunda ejemplo es de rowtype (todo el registro)

```
Estos arrays al ser dinamicos no tienen tamaño porque se van creando a medida que se necesitan las posiciones. __Podemos tener el indice 1,2,6 pero no los 3,4,5__
```

### Ejemplo

```sql
t_departamentos(1):='Sistemas';
t_departamentos(2) := 'Ingenieria';

select * 
into t_empleado
from employees;

dbms_output.put_line(t_departamentos(1));
dbms_output.put_line(t_empleados(1).FIRST_NAME);
```

Cómo no tenemos un tamaño concreto, existe un conjunto de metodos para poder movernos por los arrays.

|metodo|descripción|
|------|-----------|
|exists(n)|Determina si existe algo en determinada posicion (n)
|first|devuelve el primer indice|
|last|devuelve el ultimo indice|
|count|devuelve la cantidad de elementos|
|prior(n) |devuelve el indice anterior a n|
|next(n)|devuelve el indice posterior a n|
|delete|borra todo|
|delete(n)|Borra el indice n|
|delete(m,n)|borra del indice m a n|

```sql
IF t_empleados.exists(3) THEN
    dbms_output.put_line(t_empleados(3).first_name)
ELSE
    dbms_output.put_line('No existe el elemento');
```

### Ejemplo de creacion de un ARRAY ASOCIATIVO.

```sql
DECLARE
    TYPE tt_departamento is table of deparments.department_name%type index by pls_integer; -- es un escalar

    t_departamento  tt_departamento; -- instanciacion.

    TYPE tt_nombre is table of employees.FIRST_NAME%TYPE index by varchar2(2);

    t_nombre tt_nombre;

    TYPE tt_empleados is table of employees%ROWTYPE
    index by pls_integer; -- es una tabla, del tipo complejo

    t_empleados tt_empleados;

BEGIN
    t_departamento(1):='Informatica';
    t_departamento(2):='Sistemas';
    t_departamento(50):='Ingenieria';
    --RECORDAR que 1,2,50 no son POSICIONES son CLAVES que asocian un VALOR -> CLAVE/VALOR


    t_nombre('NI'):='Nicolas';
    --<con este ejemplo queda claro que no estamos hablando de posiciones sino de CLAVE VALOR>

    t_empleados(11).first_name := 'Nicolas Leali';
    t_empleados(11).last_name ;= 'Leali':

END;
```

```
Es muy importante recordar que estos arrays al ser asociativos no tienen una posicion, sino que es una clave que asocia un valor es CLAVE/VALOR
Estos ARRAYS son del tipo SPARSE que quiere decir que las claves no necesariamente estan seguidas.
```

### Cargar un array con valores de una tabla existente

+ **Método Uno**

```sql
DECLARE

    TYPE tt_departamento is table of departments.department_name%type index by pls_integer;

    cursor mi_cur is select department_name from departments;

    x pls_integer := 1;

    t_deparmento tt_departamento;

BEGIN

    FOR dep1 in mi_cur loop
         t_departamento(x) := dep1.department_name;
         x := x +1 ;
    END LOOP;

    dbms_output.put_line(t_departamento.count());
END;
```

+ **Método Dos**


```sql
DECLARE

    TYPE tt_departamento is table of DEPARTMENTS%ROWTYPE index by pls_integer;

    x PLS_INTEGER := 1:

    t_departamento tt_departamento;

    CURSOR mi_cursor is SELECT * from departments;

BEGIN



END;

```