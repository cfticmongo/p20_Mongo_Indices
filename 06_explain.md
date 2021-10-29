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
for (i = 0; i < 2000000; i++) {
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