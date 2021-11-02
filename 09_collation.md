# Colación en índices

La colación se puede establecer a nivel de índice.

```
use maraton
db.runners.dropIndexes()
db.runners.createIndex({surname1: 1},{collation: {locale: 'es', strength: 1}})
```

## Operación con idéntica colación que la del índice

Usará el índice

```
db.runners.find({surname1: 'lopez'}).collation({locale: 'es', strength: 1}) // Usará el índice
```

## Operación con diferente colación que la del índice

No usará el índice

```
db.runners.find({surname1: 'lopez'}).collation({locale: 'en', strength: 1}) // No usará el índice
```

## Cuando la colección disponga de colación

Si la operación tiene colación idéntica a la de la colección o no se indica usará el índice, si la
colación es diferente a la de la colección no usará el índice

```
db.createCollection('jueces', {collation: {locale: 'es', strength: 1}})
db.jueces.createIndex({apellidos: 1})
```

Si la consulta no tiene colación usará el índice

```
db.jueces.find({apellidos: 'Novo'}) // Usará el índice
```

Si la consulta tiene colación y es idéntica a la de la colección usará el índice

```
db.jueces.find({apellidos: 'Novo'}).collation({locale: 'es', strength: 1}) // Usará el índice
```

Si la consulta tiene colación pero es diferente a la de la colección no usará el índice

```
db.jueces.find({apellidos: 'Novo'}).collation({locale: 'en', strength: 1}) // No usará el índice
```