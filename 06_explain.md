# Planes de consulta o de etapas en MongoDB

- Consultas diversas y que cambiarán con el paso del tiempo
- Varios índices

Importante casuistica para determinar qué índice se utilizará cuando llegue una consulta.

El algoritmo de mongo es que decide utilizar un índice u otro en función de una evaluación
(calcula que operación le resulta más rápida) y esa información la registra en el denominado
plan caché.

Cuando posteriormente vuelve a recibir consultas con la misma forma (campos que intervienen y su orden)
emplea los datos del plan caché para volver a utilizar la misma operación.

Según diseñemos los índices Mongo utilizará estos en mayor o menor medida, por lo tanto debemos tener
presente una serie de consideraciones. Para ayudarnos a saber de antemano si un índice será usado o
no, Mongo nos proporciona el método explain().

## Método explain()

Sintaxis

db.<coleccion>.find(<forma-consulta>).explain(<modo-verbosidad>)

Verbosidad

- queryPlanner (no es necesaria pasarla por defecto). Devuelve un objeto con la info del plan ganador.
- executionStats. Ídem al anterior pero ejecuta la consulta y devuelve un objeto con la info del plan ganador y sus estadísticas.
- allPlansExecution. Ídem al anterior añadiendo en el objeto los planes no ejecutados.

Set de datos

Ejecutamos con JavaScript desde la shell

```
let runners = [];
let names = ['Laura','Juan','Fernando','María','Carlos','Lucía','David'];
let surnames = ['Fernández','Etxevarría','Nadal','Novo','Sánchez','López','García'];
let dniWords = ['A','B','C','D','P','X'];
for (i = 0; i < 1000000; i++) {
    runners.push({
        _id: i,
        name: names[Math.floor(Math.random() * names.length)],
        surname1: surnames[Math.floor(Math.random() * surnames.length)],
        surname2: surnames[Math.floor(Math.random() * surnames.length)],
        age: Math.floor(Math.random() * 100),
        dni: Math.floor(Math.random() * 1e8) + dniWords[Math.floor(Math.random() * dniWords.length)]
    })
}

use maraton
db.runners.insert(runners)
```

## Etapas más importantes del Plan de ejecución de la consulta

- COLLSPAN. Escanear documentos en la colección a partir de los datos de la consulta.
- IXSCAN. Escanear documentos en el índice a partir de los datos de la consulta.
- FETCH. Recupera los documentos de la colección a partir de la lectura de los índices.
- SORT. Ordena los documentos seleccionados en memoria (evitar).

## Uso del método explain()

Si hacemos una consulta y no tenemos índices, evidentemente tendrá que escanear la colección con la etapa
COLLSCAN.

```
db.runners.find({dni: "9464990C"}).explain('allPlansExecution')
```

Si creamos un índice en ese mismo campo y repetimos la consulta comprobaremos como se utiliza el índice.

```
db.runners.createIndex({dni: 1},{unique: true});
db.runners.find({dni: "9464990C"}).explain('allPlansExecution')
```

Ahora el plan ganador utiliza las etapas IXSCAN y FETCH, empleando solamente 3ms mientras que en el ejemplo anterior tardaba más de 1000ms.

## Uso de varios índices simples

```
db.runners.createIndex({age: 1})
db.runners.createIndex({surname1: 1})
```

En una consulta que tenga campos que estén en diferentes índices, MongoDB usará el que más rápido ejecute la consulta
de acuerdo a la evaluación de planes que realiza (que luego almacenada en la cache plan).

```
db.runners.find({surname1: 'Novo', age: {$gte: 18}}).explain('allPlansExecution')
```

Mongo puede realizar intersección de índices a partir de ciertas consultas.

```
db.runners.find({
    $or: [
        {surname1: 'Novo', age: {$gte: 18}},
        {age: 45}
    ]
}).explain('allPlansExecution')
```

## Uso de índices múltiples

En este tipo de índices condiciona el orden de los campos empleados para que sea utilizado.

```
db.runners.dropIndexes()
db.runners.createIndex({age: 1, surname1: 1})
```

### Consultas con todos los campos del índice

MongoDB usará el índice.

```
db.runners.find({surname1: 'Novo', age: {$gte: 18}}).explain('allPlansExecution')
```

### Consultas que no tengan todos los campos del índice

Para a que MongoDB le resulte óptimo el uso del índice los campos de la consulta
deberán ser prefijo.

```
db.runners.find({age: 45}).explain('allPlansExecution')  // Va a usar el índice porque age es prefijo
```

```
db.runners.find({surname1: 'Nadal'}).explain('allPlansExecution')  // No va a usar el índice porque surname1 no es prefijo
```

Para profundizar, un índice con tres campos.


```
db.runners.dropIndexes()
db.runners.createIndex({age: 1, surname1: 1, name: 1})
```

Diferentes consultas para comprobar los prefijos

```
db.runners.find({surname1: 'Nadal', age: 40}).explain('allPlansExecution')  // Usará el índice
db.runners.find({name: 'Laura', surname1: 'Nadal', age: 40}).explain('allPlansExecution')  // Usará el índice
db.runners.find({name: 'Laura', surname1: 'Nadal'}).explain('allPlansExecution')  // No usará el índice porque no es prefijo
db.runners.find({age: {$gte: 40}, name: 'Laura'}).explain('allPlansExecution')  // No es prefijo pero a veces lo puede usar
db.runners.find({age: 60, name: 'Laura'}).explain('allPlansExecution')  // No es prefijo pero a veces lo puede usar
```

Prefijo a los efectos de índices es:

db.foo.createIndex({a: 1, b: 1, c: 1, d: 1})

Las consultas prefijo de ese indice serían (no exhaustivo):

{a: <valor>}
{a: <valor>, b: <valor>}
{b: <valor>, a: <valor>}
{a: <valor>, b: <valor>, c: <valor>}
... idem anterior modificando el orden
{a: <valor>, b: <valor>, c: <valor>, d: <valor>}
... idem anterior modificando el orden

No serían prefijo:

{b: <valor>}
{c: <valor>}
{d: <valor>}
{b: <valor>, c: <valor>}
{c: <valor>, d: <valor>}
{b: <valor>, c: <valor>, d: <valor>}
...