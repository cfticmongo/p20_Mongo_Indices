# Consultas totalmente cubiertas

Estas consultas covered queries son las más eficientes y siempre que podamos debemos
crear índices para que se puedan llevar a cabo.

Son muy eficientes porque todos los datos se obtienen del índice y no necesitan escanear
los documentos en la colección.

Para que se produzcan las covered queries se tienen que dar las siguientes circunstancias: ¡Ojo certificación!

- Todos los campos del documento de consulta deben pertenecer al mismo índice.
- Todos los campos que se devuelvan tienen que estar en ese mismo índice.

```
db.runners.dropIndexes()
db.runners.createIndex({age: -1, surname1: 1})
```

```
db.runners.find({surname1: 'Nadal', age:{$gte: 60}},{surname1: 1, age: 1, _id: 0})
```