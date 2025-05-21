# Practica Power Apps
## Introducción
El objetivo de esta práctica es la creación de una aplicación mediante para el departamento de recursos humanos utilizando el entorno de desarrollo de Microsoft Power Apps.
## Entorno
Utilizaremos excel para crear varias tablas relacionadas con el departamento de recursos humanos.

![brr](Imagenes/tabla.png)

Desde Power Apps crearemos una App de lienzo.

![brr](Imagenes/lienzo.png)

## Cargar la base de datos
Para introducir la base de datos creada en nuestro excel seleccionamos el icono de datos en el menú izquierdo, agregar datos, Excel Online.

![brr](Imagenes/datos.png)

Desde ahi buscaremos en nuestro OneDrive donde tengamos guardado el excel que hemos hecho antes y seleccionamos las tablas.

![brr](Imagenes/tablas.png)

## Creación de la app
Desde la vista de arbol en el menú izquierdo seleccionamos Nueva pantalla.

![brr](Imagenes/pantalla.png)

Con la pantalla creada le añadimos los datos de nuestra tabla utilizando una galería.

![brr](Imagenes/galeria.png)

Seleccionando la galería vacía se abrirá un pequeño sub-menú desde el cual añadiremos la tabla exacta que queramos usar en esa galería.

![brr](Imagenes/dgal.png)

Para mostrar los campos que nos interesen, desde el mismo sub-menú seleccionamos "Campos".

![brr](Imagenes/campos.png)

Desde las propiedades de la galería podemos modificar todos los aspectos relacionados con este.

![brr](Imagenes/propiedades.png)

Y por ultimo desde las propiedades, si seleccionamos la pestaña "Avanzado" se nos permitirá modificar cosas mediante codigo.

![brr](Imagenes/avanzado.png)

Así es como funciona Power Apps, en este caso explicado con una galería pero vendría a ser lo mismo para todo lo que añadamos a la aplicación.

## Codigo utilizado
Aqui haré un resumen del codigo utilizado por la aplicación.

* ### App

En el apartado OnStart de la aplicación establecemos 2 variables que utilizará la pestaña "Calendario"

```
Set(varMes; Month(Today()));;
Set(varAño; Year(Today()));;
```

* ### Botones

Hay varios tipos de botones, pueden ser para guardar datos:

```
SubmitForm(Form4)
```

Para borrar datos:

```
Remove(Empleados; Gallery1.Selected)
```

Para navegar por diferentes pantallas:

```
Navigate(scrcalendario)
```

O para crear nuevos datos:

```
NewForm(Form4)
```

* ### Galerías

Las galerias tienen asignadas variables en el apartado OnSelect para que se muestren los datos que seleccionas de ellas en el formulario.

```
Set(varRegistroSeleccionado;ThisItem);;
```

* ### Formularios

Los formularios tienen asignada la variable de su correspondiente galería en el apartado Item para mostrar los datos de esta.

```
varEvaluacionSeleccionada
```

* ### Pantalla calendario

La pantalla calendario la explico a parte de forma más detallada porque es más compleja.

Lo que vemos como calendario en si es una galería configurada de la siguiente manera:

* TemplateFill
```
If(
    ThisItem.delmes = false;
    RGBA(240; 240; 240; 1);
If(
    !IsBlank(varEvento) &&
    Day(varEvento.FechaInicio) = ThisItem.Value &&
    Month(varEvento.FechaInicio) = varMes &&
    Year(varEvento.FechaInicio) = varAño &&
    ThisItem.delmes;
    RGBA(255; 230; 200; 1);
    RGBA(255; 255; 255; 1)
)
)
```

Esto lo que hace es que al seleccionar un evento, se marque en otro color el día en el calendario y también se marque en un color más oscuro los días que no pertenecen al mes actual.

* Items

```
coll_mes
```

Esto añade una colección creada por nosotros para que tenga los datos que tendría un calendario.

* Para los textos en los que se ve el mes y el año son 2 etiquetas que en el campo texto tienen configuradas las 2 variables que iniciamos en la aplicación respectivamente:

```
varAño
```

```
varMes
```

* Creamos un botón para especifar los datos de la colección usada para el calendario y en el apartado OnSelect añadimos:

```
Set(varPrimerDiaMes;Date(varAño;varMes;1));;
Set(varUltimoDiaMes;Date(varAño;varMes+1;0));;
Set(varUltimoDiaMesAnterior;varPrimerDiaMes-1);;
Set(varPrimerDiaSemana;If(Weekday(varPrimerDiaMes)-1=0;7;Weekday(varPrimerDiaMes)-1));;
Set(varPrimerDiaAnteriorMes;Day(varUltimoDiaMesAnterior-varPrimerDiaSemana+2));;
Set(varUltimoDiaSemana;If(Weekday(varUltimoDiaMes)-1=0;7;Weekday(varUltimoDiaMes)-1));;


Clear(coll_mes);;
If(varPrimerDiaSemana<>1; ForAll(Sequence(Day(varUltimoDiaMesAnterior)-varPrimerDiaAnteriorMes+1;varPrimerDiaAnteriorMes);
Collect(coll_mes;{Value:Value;delmes: false })));;
ForAll(Sequence(Day(varUltimoDiaMes));
Collect(coll_mes;{Value:Value;delmes:true}));;
ForAll(Sequence(7-varUltimoDiaSemana);
Collect(coll_mes;{Value:Value;delmes:false}));;
```

Este botón no queremos que se vea ni haga nada al pulsarse asi que en el apartado Visible escribimos false.

* En la propia pantalla en el apartado OnVisible seleccionamos el botón que contiene la colección.

```
Select(Button1)
```

* Para el botón que cambia al mes siguiente añadimos lo siguiente al OnSelect:

```
Set(varMes;varMes+1);;
If(varMes>12;Set(varMes;1);;Set(varAño;varAño+1));;
Select(Button1)
```

* Para el botón que cambia al mes anterior añadimos lo siguiente al OnSelect:

```
Set(varMes;varMes-1);;
If(varMes<1;Set(varMes;12);;Set(varAño;varAño-1));;
Select(Button1)
```

* En la segunda galería, que contiene la lista de eventos, le escribimos una nueva variable en el OnSelect

```
Set(varEvento; ThisItem)
```

## Aplicación
Mi aplicación son 4 pantallas conectadas entre si por un menú compartido en la parte superior.

### Empleados

Desde esta pantalla tenemos acceso a toda la lista de empleados, los cuales podemos seleccionar y ver o modificar sus datos asi como borrarlos, además pulsando el icono + el formulario se vaciará pudiendo añadir los datos para un nuevo empleado.

![brr](Imagenes/empleados.png)

### Candidatos

La ventana candidatos funciona igual que la de empleados con la diferencia de que aqui se ven los datos de la gente a la que se entrevista.

![brr](Imagenes/candidatos.png)

### Evaluaciones

La pantalla de evaluaciones permite añadir evaluaciones de desempeño y evaluaciones finales en caso de baja del empleado para tener referencias sobre ellos.

![brr](Imagenes/evaluaciones.png)

### Calendario

En la pantalla calendario podemos ver y añadir eventos, saliendo remarcado  el día en concreto que ocupa el evento.

![brr](Imagenes/calendario.png)

![brr](Imagenes/nuevo.png)

## Enlace de la app
https://apps.powerapps.com/play/e/default-669222ee-d5e5-4539-987e-a3015ab6df31/a/c50b64b8-0ced-4bc8-9c60-e79cf8059697?tenantId=669222ee-d5e5-4539-987e-a3015ab6df31&hint=fea03a3e-5bf2-4d80-a38a-3ee0e0ec7fb3&sourcetime=1747810157842