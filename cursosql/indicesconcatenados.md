<link rel="stylesheet" type="text/css" href="estilos.css">

### Índices multicolumna ###

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

La columna *type* en el plan de ejecución nos confirma el acceso logarítmico a través del índice compuesto. Es importante comprender que la segunda columna del índice *id_s* no está ordenada, depende completamente de la primera columna que es la que mantiene el orden. 

<div class="img-centrada">
    <img src="imagenes/indicecompuesto.png" /><br/>
    <strong>Figura 4.1. Índice multicolumna.</strong>
</div>

