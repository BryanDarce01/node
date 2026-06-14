# Optimización de consultas en MongoDB: Los índices (indexes).

## Cuando hacemos `userSchema.index(...)`

``` javascript

userSchema.index({ email: 1 });

```

le decimos a MongoDB:

- "Crea un índice para el campo email."

## 1. Sin índice

Si tenemos una colección con 1 millón de usuarios:

``` javascript

[
  { email: 'a@gmail.com' },
  { email: 'b@gmail.com' },
  { email: 'c@gmail.com' },
  ...
]

```

Y hacemos:

``` javascript

User.findOne({ email: 'b@gmail.com' });

```

MongoDB tendría que revisar:

```

Usuario 1
Usuario 2
Usuario 3
Usuario 4
...

```

hasta encontrar el correcto.

A esto se le llama:

```

COLLSCAN (Collection Scan)

```

- Escanear toda la colección.

## 2. Con índice

Si hacemos:

``` javascript

userSchema.index({ email: 1 });

```

MongoDB crea una estructura parecida a:

```

a@gmail.com -> documento A
b@gmail.com -> documento B
c@gmail.com -> documento C

```

Entonces cuando buscamos:

``` javascript

User.findOne({ email: 'b@gmail.com' });

```

- MongoDB va directamente al registro.

- Mucho más rápido.

### ¿Qué significa el `1`?

``` javascript

userSchema.index({ email: 1 });

```

El `1` significa:

- Orden ascendente

### También existe:

``` javascript

userSchema.index({ email: -1 });

```

que significa:

- Orden descendente

## 3. Índices compuestos

También podemos hacer:

``` javascript

userSchema.index({
  role: 1,
  createdAt: -1
});

```

MongoDB indexará ambos campos.

Útil para consultas como:

``` javascript

User.find({ role: 'admin' })
    .sort({ createdAt: -1 });

```

## 4. ¿Cuándo crear índices?

Campos que vayamos a consultar frecuentemente:

``` javascript

email
slug
username
role
createdAt

```

# ¿Qué hace `.explain()`?

`.explain()` permite ver cómo MongoDB ejecuta una consulta.

Ejemplo:

``` javascript

User.find({ email: 'john@gmail.com' }).explain();

```

MongoDB devuelve información de ejecución.

### Sin índice

Sale algo parecido a:

``` javascript

{
  stage: "COLLSCAN"
}

```

Significa:

```

Collection Scan

```

- MongoDB revisó toda la colección.

- Mala señal para colecciones grandes.

### Con índice

Después de:

``` javascript

userSchema.index({ email: 1 });

```

podríamos ver:

``` javascript

{
  stage: "IXSCAN"
}

```

Significa:

```

Index Scan

```

- MongoDB usó el índice.

- Buena señal.

## Los campos más importantes de `explain()`

### executionStats

Muchas veces se usa:

``` javascript

User.find({ email: 'john@gmail.com' })
    .explain('executionStats');

```

### totalDocsExamined

``` javascript

totalDocsExamined: 1000000

```

- MongoDB revisó 1 millón de documentos.

- Muy malo.

### totalKeysExamined

``` javascript

totalKeysExamined: 1

```

- MongoDB revisó solo una entrada del índice.

- Muy bueno.

## Importante en Mongoose

Si definimos:

``` javascript

email: {
  type: String,
  unique: true
}

```

Mongoose crea automáticamente un índice único:

``` javascript

{
  email: 1
}

```

Por eso muchas veces no vamos a ver:

``` javascript

userSchema.index({ email: 1 });

```

porque el índice ya existe gracias al:

``` javascript

unique: true

```

# Resumen

- `userSchema.index({ email: 1 })` crea un índice para acelerar búsquedas.

- `1` = orden ascendente, `-1` = descendente.

- Sin índice MongoDB hace `COLLSCAN`(escanea toda la colección).

- Con índice MongoDB hace `IXSCAN` (usa el índice).

- `.explain()` muestra cómo MongoDB ejecutó la consulta.

- Lo más importante de `explain()` suele ser:

    - `stage` (`COLLSCAN` o `IXSCAN`)

    - `totalDocsExamined`

    - `executionTimeMillis`

Muchos problemas de rendimiento se solucionan simplemente creando los índices correctos.