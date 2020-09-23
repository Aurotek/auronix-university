<link rel="stylesheet" type="text/css" href="estilos.css">
<div class="encabezado">
    <div class="h-izq">
        <h1 class="titulo-h1">Auronix University | <em>Curso SQL</em></h1>
    </div>
    <div class="h-der">
        <a href="indiceslentos.html"><img src="imagenes/previous.png"/></a>
        <a href="../"><img src="imagenes/home.png"/></a>
        <a href="#"><img src="imagenes/next.png"/></a>
    </div>   
</div>
 

# La claúsula where #

{:.justificado}
El filtro `where` define las condiciones de búsqueda en una enunciado *SQL*, esto es importante porque define si la BD usará funcionalmente el índice y determina el rendimiento de la consulta, generalmente una consulta lenta se debe a la mala programación de este filtro. 

{:.justificado}
Dentro de los operadores usados en un filtro where, el operador de *igualdad* es el más trivial y a menudo el más usado, sin embargo, es muy común que se combine con condiciones múltiples generando vulnerabilidad. 

Analicemos el comportamiento de la claúsula *where* con un ejemplo. Considere la tabla empleados con la siguiente definición:

```SQL
    CREATE TABLE empleados (
    id NUMBER NOT NULL,
    nombre VARCHAR(60) NOT NULL,
    apellidos VARCHAR(60) NOT NULL,
    fecha_nacimiento DATE NOT NULL,
    telefono  VARCHAR(20) NOT NULL,
    direccion CHAR(255) NOT NULL
    CONSTRAINT empleados_id PRIMARY KEY (id)
    );
```

{:.justificado}
La BD crea automáticamente un índice sobre la llave primaria, el *id*, nota que en el BTree no habrá valores repetidos, esto quiere decir que si buscamos un empleado por coincidencia exacta sobre su campo llave, la búsqueda será muy rápida. Hagámos un ejemplo para entender mejor. Considera una consulta SQL para obtener el nombre y apellidos del empleado con **id=28**. 
```SQL
    Select nombre, apellidos from empleados where id=28;
```
{:.justificado}
En este ejemplo el filtro *where* hace uso del índice primario ejecutando una operación *INDEX UNIQUE SCAN*, no necesita recorrer la lista de nodos hoja (operación *INDEX RANGE SCAN*) porque no habrán llaves repetidas, por lo tanto, solo se recorre el árbol hasta llegar a la única hoja con el valor buscado, después se obtiene el registro a través de *TABLE ACCESS BY ROWID*. El costo en tiempo de la consulta es logarítmico más el costo de obtener el registro.

Si observamos el plan de ejecución podremos constatar que la hipotésis planteada sobre el uso del índice es correcta.

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
                <td>PRIMARY</td>
                <td>PRIMARY</td>
                <td>5</td>
                <td>const</td>
                <td>1</td>
                <td>100.0</td>
                <td>NULL</td>
            </tr>
    </table>    
    <p>En MySql el tipo <code>const</code> equivale a <em>INDEX UNIQUE SCAN</em></p>
</div>
<br/>

<div class="sugerencia">
    <img src="imagenes/test.png">
    <a href="http://sqlfiddle.com/#!9/06c8ca/1/0" target="_blank">Prueba la ejecución de la consulta ejemplo haciendo click aquí.</a>    
</div>
<br/>

{:.justificado}
Para ver el plan ejecución después de probar el link de arriba, haz click sobre el hipervínculo *View Execution Plan*.


<div class="img-centrada">
    <img src="imagenes/planejecucion1.png" /><br/>
    <strong>Figura 3.1. Ver el plan de ejecución en el portal SQL Fiddle.</strong>
</div>

<style>
    *{
        box-sizing:border-box !important;
    }
</style>