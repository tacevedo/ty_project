# TY_PROJECT

## M贸dulo 3: Prueba Unitarias
  Conocer y manejar los conceptos b谩sicos de pruebas unitarias de software

### Unidad 4: Trabajar con el Framework de pruebas y las utilidades

馃憖 Instrucciones 馃憖 

Pre-requisitos

- Postman o Insomnia
- DB Browser for SQLite

Conozcamos el proyecto que vamos a probar

Vamos a instalar dependencias del proyecto, montar base de datos y poblar base de datos con informaci贸n de prueba (fixtures )
  `npm install`
  `npm run sequelize db:migrate`
  `npm run sequelize db:seed:all`

Debemos poner mucha atenci贸n en los 2 煤ltimos comandos. Con ellos definiremos un estado inicial del sistema.
Cuando implementamos metodolog铆as 谩giles es vital contar con informaci贸n que permita poder mantener estados replicables del sistema para que los desarrolladores puedan realizar intervenciones con seguridad.

Podremos hacer uso de esta informaci贸n para realizar pruebas de software y simplificar el proceso de generar distintos estados para simular escenarios, como lo hicimos anteriormente con las pruebas unitarias.

Antes de entrar en las pruebas, primero discutiremos como inicializar un estado del sistema.

### Fixtures

Seg煤n Wikipedia:
"Un fixture es un dispositivo de sujeci贸n, posicionamiento, localizaci贸n y/o soporte, ya sea al inicio, durante y/o al final de una operaci贸n de ensamble, maquinado, soldadura, inspecci贸n o alg煤n otro proceso industrial"

y en Inform谩tica

"Es un estado del sistema que permite que lo configuremos con un comportamiento repetible en el tiempo. De esta manera podemos realizar cambios que nos permitan validar si estos afectar谩n a las funcionalidades ya existentes y evitar que salgan a producci贸n errores inesperados"

### Interactuar con el sistema existente

Vamos a correr el comando `npm start` e interactuar con los endpoints a trav茅s de Postman

Debemos analizar las respuestas y comprender el concepto de Fixtures realizando pruebas manuales entendiendo que significa preparar el sistema

  - GET `/api/v1/artists`
  - POST `/api/v1/artists`
  - PUT `/api/v1/artists/:id`
  - DELETE `/api/v1/artists/:id`

Al probar este 煤ltimo Endopoint vemos que tiene un comportamiento inesperado.
Solucionar el problema en el endpoint para eliminar artista

Podemos debatir sobre los niveles de prueba y entender porque las pruebas de integraci贸n son tan importantes.

### Pruebas de Integraci贸n

Las pruebas de Integraci贸n se abstraen m谩s del c贸digo y se centran m谩s en los requerimientos de negocio sin considerar secciones espec铆ficas del c贸digo. Por lo tanto no tendremos que generar esas dificiles construcciones de mocks para que nos permita corroborar si los endpoints cumplen con los requerimientos del negocio.
Si bien ganamos en simplicidad de c贸digo, debemos sortear algunos obst谩culos aplicando los conceptos ya conocidos sobre pruebas de software, como el principio FIRST, AAA y funciones como `beforeEach`, `afterEach`.

En este caso espec铆fico los obst谩culos son:
  - Interactuar con la BBDD (Base de datos)
  - Levantar el servidor Express
  - Realizar peticiones a las rutas definidas

Vamos a sortear todas estas problem谩ticas utilizando el poder de varias librer铆as

## Integrar el Framework de pruebas y utilidades

1. Instalar jest, jest-cli y supertest

  `npm i jest jest-cli supertest --save-dev`

2. Correr el siguiente comando y analizaremos cada una de las preguntas que nos ir谩 haciendo

  `npx jest --init`

3. Modificar el archivo `src/config/config.json`

```javascript
  ...
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "test.database.sqlite3",
    "dialect": "sqlite",
    "logging": false
  },
  ...
```

antes de seguir avanzando agreguemos a `.gitignore` el archivo `test.database.sqlite3`

4. Crear una carpeta en la ra铆z del repositorio llamada `tests` y en su interior crearemos un archivo llamado `artists.test.js` con el siguiente contenido:

```javascript
const server = require('../src/server')

describe('Artists Endpoint',() => {
  it('works', () => {
    expect(true).toEqual(true)
  })
})
```

5. Ahora ejecutaremos el comando `npm run test`

Al ejecutar esto veremos un error. Esto nos dar谩 pie a discutir un tema muy importante respecto del c贸digo que escribimos.

### Las fronteras de las pruebas de integraci贸n

Cuando queremos ejecutar pruebas de integraci贸n debemos preguntarnos cu谩les son las dependencias que tiene el sistema bajo las pruebas.
En este caso es una aplicaci贸n Backend que tiene como objetivo responder a ciertos endpoints. Para poner en producci贸n la aplicaci贸n ejecutaremos la funci贸n derivada de la librer铆a Express `app.listen()`. 
驴Debemos ejecutar esto en nuestras pruebas?
驴Existir谩 alguna forma de realizar peticiones a la aplicaci贸n sin dejarla corriendo de forma constante?

Las pruebas de integraci贸n tiene como objetivo ejecutar el c贸digo fuente haciendo interactuar las unidades b谩sicas del sistema. Es nuestro deber como programadores conocer cuales son los componentes que lo conforman y entender el proceso relacionado a su ejecuci贸n en producci贸n en funci贸n de los requerimientos de negocio.

En nuestro caso es necesario dividir las responsabilidades del c贸digo contenido en el archivo `server.js` de manera de seperar las configuraci贸n de la Aplicaci贸n y su ejecuci贸n.

Crearemos un archivo dentro de la carpeta `src` llamado `app.js` con el siguiente contenido desde el archivo `server.js`


```javascript
const express = require('express')

// refactorizar
const artistsController = require('./controllers/artists.controller')

const router = express.Router()

router.get('/api/v1/artists', artistsController.getAllArtists)
router.post('/api/v1/artists', artistsController.saveArtist)
router.put('/api/v1/artists/:id', artistsController.updateArtist)
router.delete('/api/v1/artists/:id', artistsController.removeArtist)

const app = express()
app.use(express.json())
app.use(router)

module.exports = app
```

y el archivo `server.js` debe quedar as铆:

```javascript
const app = require('./app')
const port = 3000

app.listen(port, () => {
  console.log(`App server listening on port ${port}`)
})

```

y finalmente vamos a cambiar nuestra prueba para solo requerir `app`.

```javascript
const app = require('../src/app')

describe('Artists Endpoint',() => {
  it('works', () => {
    expect(true).toEqual(true)
  })
})
```

###聽Crear una prueba que interactue con la Base de datos

Para poder interactuar con la base de datos utilizaremos Sequelize. El m茅todo m谩s importante ser谩 el siguiente:

```javascript
const Models = require('../src/models')

await Models.sequelize.sync({ force: true })
```
Esto es equivalente al comando `npm run sequelize db:migrate`. Por lo tanto con 茅l prodremos crear la base de datos con las tablas actualizadas a la 煤ltima versi贸n.

En las pruebas de Integraci贸n vamos a trabajar con la siguiente API
https://sequelize.org/master/class/lib/model.js~Model.html

Ahora analicemos el siguiente c贸digo y luego lo agregaremos a nuestro archivo `artists.test.js`

```javascript
const supertest = require('supertest')

const app = require('../src/app')
const Models = require('../src/models')

// Crea tu propios fixtures con tus artistas favoritos
const artistsFixtures = [
  {
    id: 1,
    name: 'Luis Alberto Spinetta',
    description: 'Guitarrista, Cantante y Compositor Argentino considerado como uno de los m谩s innovadores debido a su uso de elementos del Jazz en m煤sica popular',
    code: 'L_A_SPINETTA_GUITAR',
    image: 'https://res.cloudinary.com/boolean-spa/image/upload/v1627536010/boolean-fullstack-js/L_A_SPINETTA_GUITAR_jpg_dwboej.png'
  },
  {
    id: 2,
    name: 'Chet Baker',
    description: 'Fue un trompetista, cantante y m煤sico de jazz estadounidense. Exponente del estilo cool. Baker fue apodado popularmente como el James Dean del jazz dado a su aspecto bien parecido',
    code: 'CHET_BAKER_TRUMPET',
    image: 'https://res.cloudinary.com/boolean-spa/image/upload/v1627536010/boolean-fullstack-js/CHET_BAKER_TRUMPET_ssbyt5.jpg'
  },
  {
    id: 3,
    name: 'Tom Misch',
    description: 'Thomas Abraham Misch es un m煤sico y productor ingl茅s. Comenz贸 a lanzar m煤sica en SoundCloud en 2012 y lanz贸 su 谩lbum de estudio debut Geography en 2018',
    code: 'TOM_MISCH_GUITAR',
    image: 'https://res.cloudinary.com/boolean-spa/image/upload/v1627536009/boolean-fullstack-js/TOM_MISCH_GUITAR_r456nt.jpg'
  }
]

describe('/api/artists', () => {

  // Configurar que pasara al inicio de la bater铆a de pruebas y al finalizar

  beforeEach(async () => {
    // Creamos la base de datos vac铆a con las tablas necesarias
    await Models.sequelize.sync({ force: true })
  })

  afterAll(async () => {
    // En toda base de datos es importante cerrar la conexi贸n
    await Models.sequelize.close()
  })

  it('returns an list of artists', async () => {
    await Models.Artist.bulkCreate(artistsFixtures)
    
    const response = await supertest(app)
      .get('/api/v1/artists')
      .expect(200)
    expect(response.body).toMatchObject(artistsFixtures)
  })

  it('returns 500 when the database throws error', async () => {
    // Podemos ser ingeniosos y generar un escenario que haga fallar a la base de datos
    await Models.Artist.drop()

    const response = await supertest(app)
      .get('/api/v1/artists')
      .expect(500)
    expect(response.body).toMatchObject({ message: 'SQLITE_ERROR: no such table: Artists' })
  })
})
```


Seguiremos escribiendo las dem谩s pruebas. Para la prueba del endpoint POST podemos utilizar el siguienete ejemplo:

```javascript
const newArtist = {
  name: "Herbie Hancock",
  description: "Es un pianista, tecladista y compositor estadounidense de jazz. A excepci贸n del free jazz, ha tocado pr谩cticamente todos los estilos jazz铆sticos surgidos tras el bebop: hard bop, fusi贸n, jazz modal, jazz funk, jazz electr贸nico, etc.",
  code: "HERBIE_HANCOCK_KEYBOARD",
  image: "https://res.cloudinary.com/boolean-spa/image/upload/v1627537633/boolean-fullstack-js/HERBIE_HANCOCK_KEYBOARD_n6b2fv.jpg"
}
```


Antes de terminar el laboratorio, crear un nuevo script en el archivo `package.json` llamado "test:ci" que contenga el siguiente c贸digo

```javascript
  ...
  "scripts": {
      ...
      "test:ci": "jest --coverage --runInBand"
      ...
   }
   ...
```
Podemos probar que el comando qued贸 bien configurado corriendo `npm run test:ci` y corroborar que todo salga OK.

隆脡XITO!
