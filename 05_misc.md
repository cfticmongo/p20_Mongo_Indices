## Wildcard index

db.<coleccion>.createIndex({"<campo>.$**": 1})

Ampliar el índice a todos los campos del subdocumento en el que se construye

Set de datos

```
use fabrica

db.productos.insert({
    nombre: 'Turbina F-40',
    caracteristicas: {
        tuercas: 'F11',
        chapa: 'c21'
    }
})

db.productos.createIndex({"caracteristicas.$**": 1})

```

Ahora una consulta por cualquiera de los campos del subdocumento usará el índice

```
db.productos.find({"caracteristicas.chapa": "c21"})
```

Y además si se modifica cualquier campo de ese subdocumento o tenemos otros documentos con otros campos
también estarán en el índice

```
db.productos.insert({
    nombre: 'Turbina F-40V',
    caracteristicas: {
        tuercas: 'F11',
        chapa: 'c21',
        colector: 'F-R-12' 
    }
})
```

```
db.productos.find({"caracteristicas.colector": 'F-R-12'}) // También usará el índice para este nuevo campo
```

## Índices TTL

Su funcionalidad es eliminar documentos de una colección por un campo que contenga una marca de tiempo. Su
uso, improbable, estará restringido a logs de aplicaciones, pequeños documentos temporales que no sean
necesario almacenar pasado un tiempo.

- Cantidad de tiempo desde que se crea el documento hasta que se elimina.
- Indicar una marca de tiempo en la que se eliminará el documento.

db.<coleccion>.createIndex({<campo-fecha>: 1 | -1}, {expireAfterSeconds: <segundos> | 0})

```
use app

db.logs.createIndex({startSession: 1}, {expireAfterSeconds: 60}) // Eliminará los docs 60 segundos después de su creación
```

Si insertamos un documento, con el campo startSession, se eliminará pasado ese tiempo desde su inserción

```
db.logs.insert({nickname: 'pepe20', startSession: new Date()})
```

```
db.logs.createIndex({deleteAt: 1}, {expireAfterSeconds: 0}) //Eliminará los docs en la fecha que tenga cada uno de ellos

```                                                         // en el campo deleteAt

En el documento deberá llevar el campo deleteAt con la fecha en la que se eliminará

```
db.logs.insert({nickname: 'juan', visits: 123, deleteAt: new Date(2021, 9, 29, 18, 31)}) // lo eliminará en la marca
                                                        // de tiempo pasada
```

## Índices únicos ¡Ojo certificación!

db.<coleccion>.createIndex({<campo>: 1 | -1, ..},{unique: true})

```
use clinica

db.pacientes.createIndex({dni: 1}, {unique: true})

db.pacientes.createIndex({numeroSS: 1}, {unique: true}) // Podemos tener varios índices únicos

```

Operaciones de escritura

```
db.pacientes.insert({nombre: 'Sara', dni: '55235638T'})
db.pacientes.insert({nombre: 'Luis', dni: '55235638T'}) // Error 11000
```
Si el índice se intenta construir sobre una colección con documentos con valores duplicados en
ese campo devolverá error

```
db.pacientes.dropIndexes()

db.pacientes.insert({nombre: 'Luis', dni: '55235638T'})
db.pacientes.createIndex({dni: 1}, {unique: true}) // Error

MongoServerError: Index build failed:

```

Se pueden usar índices únicos múltiples

```
use shop

db.articulos.createIndex({marca: 1, nombre: 1}, {unique: true})

db.articulos.insert({marca: 'Nike', nombre: 'Sudadera básica'})
db.articulos.insert({marca: 'Adidas', nombre: 'Sudadera básica'})
db.articulos.insert({marca: 'Nike', nombre: 'Sudadera básica'}) // Dará error

```

Si tenemos un índice único solo podremos tener en esa colección un documento
que no contenga el campo (ya que a los efectos de índice Mongo asigna el valor null a ese documento en ese campo)

```
use clinica

db.pacientes.createIndex({dni: 1}, {unique: true})

db.pacientes.insert({nombre: 'Laura'}) // El primer documento sin el campo dni lo permitirá
db.pacientes.insert({nombre: 'Carlos'}) // Error

```