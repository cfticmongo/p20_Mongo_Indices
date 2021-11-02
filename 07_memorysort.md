# Ordenación en memoria

Para las consultas ordenadas MongoDB permite evitar tener que ordenar el resultado en memoria
si diseñamos eficientemente los índices.

Para evitar el uso de ordenación en memoria, el sentido de ordenación de la consulta debe coincidir
con el sentido de ordenación del índice o su inverso (en todos los campos).

## Índices simples (el orden no influye)

```
db.runners.dropIndexes()
db.runners.createIndex({age: 1})
```

En este caso para cualquier consulta ordenada por ese campo, no tendrá que ordenar
en memoria con independencia del sentido de ordenación.

```
db.runners.find({age: {$lt: 50}}).sort({age: 1}).explain('allPlansExecution') // Ok, no ordenará en memoria
db.runners.find({age: {$lt: 50}}).sort({age: -1}).explain('allPlansExecution') // Ok, no ordenará en memoria
```

## Índices compuestos (el orden influye)

```
db.runners.dropIndexes()
db.runners.createIndex({age: -1, surname1: 1})
```

Las consultas con el mismo orden que el índice o el inverso en todos los campos tampoco ordenarán en memoria.

```
db.runners.find({}).sort({age: -1, surname1: 1}).explain('allPlansExecution') // Ok, no ordenará en memoria
db.runners.find({}).sort({age: 1, surname1: -1}).explain('allPlansExecution') // Ok, no ordenará en memoria
db.runners.find({}).sort({age: -1, surname1: -1}).explain('allPlansExecution') // Bad, tendrá que ordenar en memoria
db.runners.find({}).sort({age: 1, surname1: 1}).explain('allPlansExecution') // Bad, tendrá que ordenar en memoria
```

¿Qué ocurre si no es prefijo en la ordenación?

```
db.runners.find({age: {$gte: 98}}).sort({surname1: 1}).explain('allPlansExecution') // Bad, tendrá que ordenar en memoria porque /// los campos del documento de ordenación no son prefijo del índice
```

Importante ¡Ojo certificación! Si la consulta en su documento de ordenación tiene campos que no son prefijo del
índice pero en el documento de consulta estan los campos que le faltan para ser prefijo con una condición de
igualdad no tendrá que utilizar ordenación en memoria.

```
db.runners.find({age: 50}).sort({surname1: 1}).explain('allPlansExecution') //  Ok, no ordenará en memoria
```