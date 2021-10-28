# Introducción a Índices

Índices se construyen a nivel de colección

- Ventajas. Velocidad y eficiencia en las consultas.
- Inconvenientes. Ocupan espacio en memoria y hacen más lentas las operaciones de escritura.

Tipología 

MongoDB utiliza B-Tree

Comandos 

## Crear índices createIndex()

Sintaxis
db.<colección>.createIndex({<campo>: 1 | -1 | <valor>, <campo>: 1 | -1 | <valor>, ...}, <opciones>)

```
use gimnasio

db.clientes.createIndex({apellidos: 1}) // 1 ascendente y -1 descendente

```

'apellidos_1'  // Devuelve el nombre del índice

Si quisieramos darle un nombre se puede pasar como opción

```
db.clientes.createIndex({nombre: 1}, {name: 'nombre-ascendente'})

```

## Visualización de índices getIndexes()

```
db.clientes.getIndexes() // Devuelve los índices de la colección

```

## Eliminar un índice dropIndex()

```
db.clientes.dropIndex('nombre-ascendente') 
```

## Eliminar todos los índices dropIndexes()

```
db.clientes.dropIndexes() // Elimina todos los índices menos el de _id 
```

