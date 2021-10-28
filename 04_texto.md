# Índices de textos

Búsqueda de palabras en grandes volúmenes de textos.

Sintaxis
db.<coleccion>.createIndex({<campo>: "text"}) // creación
{$text: {$search: <terminos-busqueda>}} // uso del índice

```
use biblioteca

db.libros.createIndex({autor: "text"})

db.libros.find({$text: {$search: 'Ende'}}) // No se espefica el campo en la consulta porque solo puede haber un índice
                                            // de texto por colección

```
{ _id: ObjectId("61797fd74267241ee0aed7aa"),
  titulo: 'Momo',
  autor: 'Michael Ende', // Contiene la palabra Ende
  valoraciones: [ 'muy bien', 'regular', 'correcto', 'bien', 'muy bien' ] }
{ _id: ObjectId("617972798648ca8f924f81da"),
  autor: 'Michael Ende', // Contiene la palabra Ende
  titulo: 'La Historia Interminable',
  precio: 30 }

Limitación a un índice de texto por colección

```
db.libros.createIndex({titulo: "text"}) // Error
```
Si podemos crear indices compuestos de tipo texto

```
db.libros.dropIndexes()

db.libros.createIndex({autor: 'text', titulo: 'text'})

```

Busqueda de una palabra en todos los campos del índice compuesto

```
db.libros.find({$text: {$search: 'Ende'}})

db.libros.insert({titulo: 'Biografía de Michael Ende', autor: 'John Doe'})

db.libros.find({$text: {$search: 'Ende'}}) // La consulta busca tanto en el campo autor como en título

```

{ _id: ObjectId("61797fd74267241ee0aed7aa"),
  titulo: 'Momo',
  autor: 'Michael Ende',
  valoraciones: [ 'muy bien', 'regular', 'correcto', 'bien', 'muy bien' ] }
{ _id: ObjectId("617972798648ca8f924f81da"),
  autor: 'Michael Ende',
  titulo: 'La Historia Interminable',
  precio: 30 }
{ _id: ObjectId("617ae8624267241ee0aed7b8"),
  titulo: 'Biografía de Michael Ende',
  autor: 'John Doe' }

## Wildcard text index
Para buscar palabras en todos los campos. Si se amplian los campos tras la construcción del índice, los textos de
los nuevos campos también serán indexados.

```
db.libros.dropIndexes()

db.libros.createIndex({"$**": "text"})

```

## Casos de uso

Set de datos

```
db.libros.insert([
    {titulo: 'París era una Fiesta', autor: 'Ernest Hemingway'},
    {titulo: 'París, La Guía Completa', autor: 'vv.aa.'},
    {titulo: 'Silicon Valley, la era de internet', autor: 'John Doe'},
])

db.libros.dropIndexes()

db.libros.createIndex({"titulo": "text"})
```

### Búsqueda de una palabra

```
db.libros.find({$text: {$search: 'pAris'}}) // Por defecto es case-insensitive y diacritic-insensitive
```

{ _id: ObjectId("617aea334267241ee0aed7ba"),
  titulo: 'París, La Guía Completa',
  autor: 'vv.aa.' }
{ _id: ObjectId("617aea334267241ee0aed7b9"),
  titulo: 'París era una Fiesta',
  autor: 'Ernest Hemingway' }


```
db.libros.find({$text: {$search: 'Pa'}}) // No encuentra fragmentos de palabras
```
{} // No emite error pero si no contiene esa palabra no devuelve nada


```
db.libros.find({$text: {$search: /Pa/}}) // Solo permite el tipo string
```
MongoServerError: "$search" had the wrong type. Expected string, found regex

### Búsqueda de varias palabras (OR inclusivo)

```
db.libros.find({$text: {$search: 'paris era'}})
db.libros.find({$text: {$search: 'era paris'}})  // El orden de las palabras no influye
```
{ _id: ObjectId("617aea334267241ee0aed7b9"),
  titulo: 'París era una Fiesta',
  autor: 'Ernest Hemingway' }
{ _id: ObjectId("617aea334267241ee0aed7bb"),
  titulo: 'Silicon Valley, la era de internet',
  autor: 'John Doe' }
{ _id: ObjectId("617aea334267241ee0aed7ba"),
  titulo: 'París, La Guía Completa',
  autor: 'vv.aa.' }

### Búsqueda de varias palabras (AND)

```
db.libros.find({$text: {$search: '"era" "paris"'}}) // entrecomillar las palabras
```

{ _id: ObjectId("617aea334267241ee0aed7b9"),
  titulo: 'París era una Fiesta',
  autor: 'Ernest Hemingway' }

### Búsqueda de una frase exacta

```
db.libros.find({$text: {$search: '"una fiesta"'}}) // entrecomillar la frase
```

{ _id: ObjectId("617aea334267241ee0aed7b9"),
  titulo: 'París era una Fiesta',
  autor: 'Ernest Hemingway' }

### Busqueda con exclusión de palabras ¡Ojo certificación!

```
db.libros.find({$text: {$search: 'Paris -guia'}}) // Las palabras excluidas preceder con -
```

{ _id: ObjectId("617aea334267241ee0aed7b9"),
  titulo: 'París era una Fiesta',
  autor: 'Ernest Hemingway' }

### Omite las denominadas stop words (artículos, preposiciones, etc...)

Por defecto las stop words que omite son las del lenguaje inglés

```
db.libros.insert({titulo: 'The Second World War', autor: 'vv.aa.'})

db.libros.find({$text: {$search: 'The'}}) // No devuelve nada porque este artículo no está en el índice
```

Se puede establecer el lenguaje en la creación del índice

```
db.libros.dropIndexes()

db.libros.createIndex({"titulo": "text"},{ default_language: "spanish" })

db.libros.find({$text: {$search: 'La'}}) // No devolverá nada

```

### Busqueda por stemmed words

Permite búsquedas por las raices de las palabras

```
db.libros.dropIndexes()

db.libros.createIndex({"titulo": "text"})

db.libros.insert([
    {titulo: 'Agile Consultants', autor: 'vv.aa.'},
    {titulo: 'Consulting for Global Markets', autor: 'John Doe'}
])

db.libros.find({$text: {$search: 'consult'}})

```

{ _id: ObjectId("617af1414267241ee0aed7bd"),
  titulo: 'Agile Consultants',
  autor: 'vv.aa.' }
{ _id: ObjectId("617af1414267241ee0aed7be"),
  titulo: 'Consulting for Global Markets',
  autor: 'John Doe' }

Probamos en castellano

```
db.libros.dropIndexes()

db.libros.createIndex({"titulo": "text"},{ default_language: "spanish" })

db.libros.insert([
    {titulo: 'Economía de Guerra', autor: 'vv.aa.'},
    {titulo: 'Economice su hogar', autor: 'John Doe'}
])


db.libros.find({$text: {$search: 'econom'}}) // Fail en castellano (inconsistente) 

```