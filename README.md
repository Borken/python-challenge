# Juego de Tornos
With this yml, an automation build has been configured to run an application that integrates three microservices in one host.

The microservices referenced, created and fetched are:

* Places: https://github.com/Borken/places_microservice
* Characters: https://github.com/Borken/characters_microservice
* Juego de Tornos: https://github.com/Borken/juego_de_tornos_microservice

## Requirements
Having [Docker](https://docs.docker.com/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.

## Installation

1. Build and run the project: `docker-compose up --build -d`
2. Init characters database:

   `docker-compose exec characters_api flask db init`
   
   `docker-compose exec characters_api flask db migrate`
   
   `docker-compose exec characters_api flask db upgrade`
3. Init places database:

   `docker-compose exec places_api flask db init`
   
   `docker-compose exec places_api flask db migrate`
   
   `docker-compose exec places_api flask db upgrade`
   
After these steps you should be able to access a swagger ui through the url http://0.0.0.0:8080/api/ui

## Optional

Both databases are empty, but you can fill them with some examples running:

`docker-compose exec places_api python utils/populate_database.py`

And:

`docker-compose exec characters_api python utils/populate_database.py`

## Next steps:

* Security: 

  Add an nginx container to the build to use as a reverse proxy, with https, certificates...
  
  Use env variables or something similar to store the database creds instead of hardcoding them in the configurations.
  
  Use oauth or similar to secure communications between microservices. Primarily to avoid unwanted modifications, deletions or creations.

* Testing:

  Improve testing to test correct format returns in requests.
  
  Use mock libraries to simulate places and characters api returns in the juego de tornos microservice (Now it needs the apis running and with valida data).

* CI:

  Add continuous integration to launch tests, execute migrations...
## Project Original Instructions

La Compañía Plateada S.A. necesita hacer un estudio de cuanta gente vive en cada región de Juego de Tornos. Para ello, hay que construir un proyecto basado en microservicios que permita consultar y dar de alta datos sobre personajes y lugares de Poniente a través de APIs.
Su idea es la siguiente:
- Un microservicio para los Personajes (CRUD).
- Otro microservicio para los Lugares (CRUD).
- Otro microservicio que sirva de punto de entrada para los usuarios, teniendo que consultar los otros microservicios para componer su output (BaaS).

Los campos de las entidades serán:
- **Personaje:** debe tener nombre, lugar al que pertenece y si es el rey de ese lugar. Y si está vivo o muerto (si no lo sabemos, lo daremos por muerto).
- **Lugar:** debe tener un nombre.


**¿Qué restricciones nos ponen?** 
- Todos lo personajes deben pertenecer a un lugar, salvo que estén muertos, por lo que solo se podrían dar de alta personajes si ya existen lugares dados de alta.
- Un lugar solamente puede tener un personaje que sea rey y esté vivo.
- Las bases de datos están separadas, por lo que para hacer un Join de Lugares y Personas hay que utilizar el tercer microservicio para componer la salida.


Un compañero de La Compañía Plateada S.A. empezó el modelado de las APIs pero se ha cogido la baja por paternidad y te han pedido que continues con el trabajo y con el modelado:


## Lugar Microservices
El servicio "lugar" debe consultar en una base de datos los lugares y devolverlos, por ejemplo:

place-microservice:8081/v1/places/
```json
[
    {
        "id": 1,
        "name": "Descaro del Rey"
    },
    {
        "id": 2,
        "name": "Más allá del Zumo"
    },
    {
        "id": 3,
        "name": "Veranolandia"
    }
]
```

## Personas Microservices
Otro servicio debe consultar en otra base de datos las personas que viven en Juego de Tornos  y devolverlos, por ejemplo:

people-microservice:8082/v1/people/
```json
[
    {
        "id": 1,
        "name": "Sensei Lamister",
        "isAlive": true,
        "placeId": 1
        
    },
    {
        "id": 2,
        "name": "Aiba Stack",
        "isAlive": true,
        "placeId": 3
    },
    {
        "id": 3,
        "name": "Joaquín Nevado",
        "isAlive": true,
        "placeId": 3
    },
    {
        "id": 4,
        "name": "Dedo Gordo",
        "isAlive": false,
        "placeId": null
    },
    {
        "id": 5,
        "name": "Juanito Lamister",
        "isAlive": true,
        "placeId": 1
    },
    {
        "id": 6,
        "name": "Nerf Stack",
        "isAlive": true,
        "placeId": 3
    }
]
```


## Juego de Tornos Microservices
Por último, el tercer servicio, dado un id de lugar que se le pase, deberá exponer una api que, consultando las otras dos anteriores, componga una respusta en su api tal que:

got-microservice:8083/v1/places/
```json
[
    {
        "id": 1,
        "name": "Descaro del Rey",
        "people": [
            {
                "id": 1,
                "name": "Sensei Lamister",
                "isAlive": true
            }, {
                "id": 5,
                "name": "Juanito Lamister",
                "isAlive": true
            }
        ]
    },
    {
        "id": 2,
        "name": "Más allá del Zumo",
        "people": [
            {
                "id": 3,
                "name": "Juaquín Nevado",
                "isAlive": true
            }
        ]
    },
    {
        "id": 3,
        "name": "Veranolandia",
        "people": [
            {
                "id": 2,
                "name": "Aiba Stack",
                "isAlive": true
            }, {
                "id": 6,
                "name": "Nerf Stack",
                "isAlive": true
            }
        ]
    }
]
```

# Guidelines:
- Usar Flask para contruir los servicios.
- Usar flask_restplus o connexion para construir las APIs y exponer un Swagger.
- Usar SQLAlchemy (sobre PostgreSQL o Mysql) para construir la BBDD.
- Usar Alembic para crear la base de datos.
- Poblar la base de datos con datos de prueba.
- Documentar cómo arrancar el código.
- Test unitarios.

## Plus:
- Dockerizar los 3 servicios.
