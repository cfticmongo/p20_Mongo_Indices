# Índices multikey

Los índices multikey son aquellos que se construyen sobre campos que pudieran
tener valores de tipo array

Sintaxis
No tiene una sintaxis específica
db.<colección>.createIndex({<array>: 1 | -1})

Set de datos

```
use tienda2

db.articulos.insert({
    nombre: 'Sudadera FTR34',
    marca: 'Nike',
    stock: [
        {color:'azul', talla: 's', cantidad: 2},
        {color:'azul', talla: 'l', cantidad: 12},
    ],
    categorias: ['mujer','ropa']
})
```

1º Premisa podemos tener varios índices simples multikey ¡Ojo certificación!

```
db.articulos.createIndex({stock: 1})

db.articulos.createIndex({categorias: 1})
```

2ª Premisa en el caso de índices múltiples, solo uno de los campos puede ser array ¡Ojo certificación!

```
db.articulos.createIndex({categorias: 1, stock: 1}) // Error
```

MongoServerError: Index build failed: 

```
db.articulos.createIndex({marca: 1, categorias: 1}) // OK
```

'marca_1_categorias_1'

## Limitaciones a la escritura con el uso de índices multikey ¡Ojo certificación!

Como MongoDB es schemaless provoca que si tenemos un índice multikey compuesto, si insertaramos un
documento con un campo con valor array y ese campo esté en el índice y no sea el de arrays no dejará
ejecutar la inserción para evitar incumplir la premisa anterior

Si hacemos la siguiente inserción

```
db.articulos.insert({
    nombre: 'Zapatillas tal',
    marca: 'Nike',
    stock: [
        {color:'azul', talla: '40', cantidad: 2},
        {color:'azul', talla: '4l', cantidad: 12},
    ],
    categorias: ['hombre','calzado']
})
```
Ok, se insertará

En cambio con esta otra inserción

```
db.articulos.insert({
    nombre: 'Zapatillas lorem...',
    marca: ['nike','importación'], // Error porque este campo ya no podrá almacenar arrays
    stock: [
        {color:'azul', talla: '40', cantidad: 22},
        {color:'azul', talla: '4l', cantidad: 12},
    ],
    categorias: ['hombre','calzado']
})
```
MongoBulkWriteError: cannot index parallel arrays [categorias] [marca]