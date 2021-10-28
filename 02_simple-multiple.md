# Índice simple

```
use gimnasio

db.clientes.createIndex({apellidos: -1})
```

Los índices se pueden construir por un campo de un subdocumento, usando la notación del punto

```
db.clientes.createIndex({"direccion.localidad": 1})
```

# Índice compuesto

```
db.clientes.createIndex({apellidos: 1, nombre: -1}) // Influye el órden y sentido de los campos
```

