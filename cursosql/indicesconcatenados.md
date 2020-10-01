<link rel="stylesheet" type="text/css" href="estilos.css">
<div class="encabezado">
    <div class="h-izq">
        <h1 class="titulo-h1">Auronix University | <em>Curso SQL</em></h1>
    </div>
    <div class="h-der">
        <a href="filtrowhere.html"><img src="imagenes/previous.png"/></a>
        <a href="../"><img src="imagenes/home.png"/></a>
        <a href="#"><img src="imagenes/next.png"/></a>
    </div>   
</div>
 
# Índices multicolumna #

{:.justificado}
Un índice concatenado o multicolumna es la unión de dos o más campos que a menudo se emplean para representar un registro de la tabla, con fines didácticos supongamos que en nuestra empresa existen diferentes sucursales y se necesita identificar de forma única a un empleado usando su id y el id de la sucursal donde labora, podemos definir el campo llave de la tabla empleados de la forma `id_empleado,id_sucursal`, a esta unión de dos datos atómicos para representar a una entidad se le conoce como índice multicolumna. Consideremos la siguiente tabla como caso de estudio:

```SQL
   CREATE TABLE empleados (
   id INT NOT NULL,
   id_s INT NOT NULL,
   nombre VARCHAR(60) NOT NULL,
   apellidos VARCHAR(60) NOT NULL,
   fecha_nacimiento DATE,
   tel  VARCHAR(20) NOT NULL,
   direccion CHAR(255),
   UNIQUE KEY id_empleado_sucursal(id,id_s)
);
```

{:.justificado}
En teoria tenemos un acceso de tipo *INDEX UNIQUE SCAN* cuando se use el índice compuesto porque se asegura la unicidad. Analicémos que ocurre con el plan de ejecución si ejecutamos la siguiente consulta:

```SQL
    SELECT nombre, apellidos FROM empleados where id=123 and id_s=20;
```

<div class="ejercicio execution-plan">
    <strong>Plan de ejecución MySql</strong><br/><br/>
    <table class="">
            <tr>
                <th>id</th>
                <th>select_type</th>
                <th>table</th>
                <th>type</th>
                <th>possible_keys</th>
                <th>key</th>
                <th>key_len</th>
                <th>ref</th>
                <th>rows</th>
                <th>filtered</th>
                <th>Extra</th>
            </tr>
            <tr>
                <td>1</td>
                <td>SIMPLE</td>
                <td>empleados</td>
                <td><strong><em style='color:blue;'>const</em></strong></td>
                <td>id_empleado_sucursal</td>
                <td>id_empleado_sucursal</td>
                <td>8</td>
                <td>const,const</td>
                <td>1</td>
                <td></td>
                <td></td>
            </tr>
    </table>    
</div>
<br/>

{:.justificado}
La columna *type* en el plan de ejecución nos confirma el acceso logarítmico a través del índice compuesto. Es importante comprender que la segunda columna del índice *id_s* no está ordenada, depende completamente de la primera columna que es la que mantiene el orden. 

<div class="img-centrada">
    <img src="imagenes/indicecompuesto.png" /><br/>
    <strong>Figura 4.1. Índice multicolumna.</strong>
</div>

{:.justificado}
La *figura 4.1* muestra una aproximación de la estructura que tiene el índice, nota que aunque pareciera que las dos columnas son independientes en realidad ambas forman la llave, estas dos columnas que forman el índice compuesto tienen una relación de dependencia, es decir, la segunda columna depende de la primera, si se tiene que mover una llave, se mueven ambas columnas. Observemos la entrada `123, 27` en la lista de nodos hoja, esta llave representa a todo el nodo (porque es la mayor del conjunto respecto a la primera columna), por lo tanto, se inserta en el nodo padre manteniendo el orden ascendente respecto a la columna *id*.

{:.justificado}
Debido a la relación de dependencia que tiene la segunda columna respecto a la primera, las búsquedas que involucren únicamente el campo *id* harán uso del índice mientras que aquellas que solo busquen por el campo *id_s* ignoran el índice. Aclaremos con dos ejemplos:

```SQL
    SELECT nombre, apellidos FROM empleados where id=123;
```

<div class="ejercicio execution-plan">
    <strong>Plan de ejecución MySql</strong><br/><br/>
    <table class="">
            <tr>
                <th>id</th>
                <th>select_type</th>
                <th>table</th>
                <th>type</th>
                <th>possible_keys</th>
                <th>key</th>
                <th>key_len</th>
                <th>ref</th>
                <th>rows</th>
                <th>filtered</th>
                <th>Extra</th>
            </tr>
            <tr>
                <td>1</td>
                <td>SIMPLE</td>
                <td>empleados</td>
                <td><strong><em style='color:blue;'>ref</em></strong></td>
                <td>id_empleado_sucursal</td>
                <td>id_empleado_sucursal</td>
                <td>4</td>
                <td>const</td>
                <td>1</td>
                <td></td>
                <td></td>
            </tr>
    </table>    
</div>
<br/>

<div class="resumen">
    <img src="imagenes/idea.png">
    Cuando la columna <em>type</em> del plan de ejecución muestra <strong><em>ref</em></strong> significa que se usó una comparación de igualda exacta <code>=</code> en el predicado. Si despliega <strong><em>range</em></strong> quiere decir que se usó un operador de rango en el predicado <code>>,<,>=,<=, between</code>, en ambos casos se ejecuta una operación <em>INDEX RANGE SCAN</em>.
</div>
<br/>

{:.justificado}
El plan de ejecución muestra que la consulta por el campo *id* usó el índice para encontrar los registros, ¿Que ocurre si buscamos únicamente por el campo *id_s*?.

```SQL
    SELECT nombre, apellidos FROM empleados where id_s=20;
```

<div class="ejercicio execution-plan">
    <strong>Plan de ejecución MySql</strong><br/><br/>
    <table class="">
            <tr>
                <th>id</th>
                <th>select_type</th>
                <th>table</th>
                <th>type</th>
                <th>possible_keys</th>
                <th>key</th>
                <th>key_len</th>
                <th>ref</th>
                <th>rows</th>
                <th>filtered</th>
                <th>Extra</th>
            </tr>
            <tr>
                <td>1</td>
                <td>SIMPLE</td>
                <td>empleados</td>
                <td><strong><em style='color:blue;'>ALL</em></strong></td>
                <td></td>
                <td></td>
                <td></td>
                <td></td>
                <td>1000</td>
                <td></td>
                <td>Using where</td>
            </tr>
    </table>    
</div>
<br/>
{:.justificado}
Como estamos búscando por la segunda columna del índice concatenado y esta en realidad no tiene un orden, el manejador de la BD descarta el índice y ejecuta una operación *FULL ACCESS TABLE*. La solución es crear un índice sobre el campo *id_s*, si es que realmente se necesita ejecutar búsquedas de este tipo.

<div class="resumen">
    <img src="imagenes/idea.png">
    Cuando se tiene un índice compuesto, se debe poner especial atención a los filtros que involucran los campos del índice. El motor de la BD usará el índice compuesto cuando reciba consultas que respeten el orden de prefijos especificado en el índice. Por ejemplo si existe un índice compuesto (campo1, campo2, campo3) se usará el indice en casos como:
    <ul>
        <li><em>campo1=valor1</em></li>
        <li><em>campo1=valor1 and campo2=valor2</em></li>
        <li><em>campo1=valor1 and campo2=valor2 and campo3=valor3</em></li>
    </ul>
    El índice será ignorado en consultas con filtros como:
    <br/>
    <ul>
        <li><em>campo2=valor2</em></li>
        <li><em>campo2=valor2 and campo3=valor3</em></li>
        <li><em>campo1=valor1 or campo2=valor2</em></li>
    </ul>
</div>
<br/>


<style>
    *{
        box-sizing:border-box !important;
    }
   
</style>


