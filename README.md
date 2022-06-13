# SD-Homeworks3

## Integrantes: Ze Hui Fu y Joaquín Fernández.

**Instrucciones y uso**

1) Clonar el repositorio.
    ```bash
    https://github.com/Joacker/SD-Homeworks2.git
    ```
2) Iniciar contenedores.
    ```bash
    docker-compose up --build
    ```
3) Crear un paciente con una receta nula en la siguiente ruta:
    ```url
    http://localhost:3000/create [POST]
    ```
    ```json
    {
        "rut":"1234",
        "nombre":"javiera",
        "apellido":"salas",
        "email":"1234",
        "fecha_nacimiento":"123"
    }
    ```
    Como respuesta retornará el ID de receta.
    ```json
    {
    "ID_Receta": "0606852d-ec4c-4dab-9ca1-25fc67143584"
    }
    ```
4) Editar la receta en la siguiente ruta:
    ```url
    http://localhost:3000/edit [POST]
    ```
    ```json
    {
        "id":"0606852d-ec4c-4dab-9ca1-25fc67143584",
        "comentario": "ayayaya",
        "farmacos":"mariajuanas",
        "doctor": "simi"
    }
    ```
5) Verificar si se actualizó la receta del paciente en la siguiente ruta:
    ```url
    http://localhost:3000/getRecipes [GET]
    ```
    Retorna todas las recetas con sus respectivos pacientes.
    ```json
    {
        "id": "0606852d-ec4c-4dab-9ca1-25fc67143584",
        "comentario": "ayayaya",
        "doctor": "simi",
        "farmacos": "mariajuanas",
        "id_paciente": "acd906e9-fb1f-4cf4-bf97-af87112b1a35"
    }
    ```
6) Eliminar la receta en la siguiente ruta:
    ```url
    http://localhost:3000/delete [POST]
    ```
    ```json
    {
    "id": "0606852d-ec4c-4dab-9ca1-25fc67143584"
    }
    ```
    Para verificar que la operación se haya eliminado correctamente, realizar punto 5.

----
**Preguntas**
----
- Explique la arquitectura que Cassandra maneja. Cuando se crea el clúster ¿Cómo los nodos se conectan? ¿Qué ocurre cuando un cliente realiza una petición a uno de los nodos? ¿Qué ocurre cuando uno de los nodos se desconecta? ¿La red generada entre los nodos siempre es eficiente? ¿Existe balanceo de carga?

Casandra está construida con un sistema Peer-To-Peer. Los nodos se comunican o conectan a través de Gossip, este protocolo permite el intercambio de información entre nodos continuamente.

Cuando se realiza una petición a uno de los nodos se generan logs de los commit en cada uno de los nodos, por lo tanto, la disponibilidad de los datos es sumamente alta.

El cluster creado consta de 3 nodos y cada una tiene réplicas de las otras, por lo tanto si se desconectará un solo nodo, no pasaría nada, pero si se desconectaran dos nodos y estos dos nodos son los que tienen la tabla paciente, se tendría perdida o falta de datos, ya que el nodo restante solo tiene las recetas.

La red generada es eficiente en cuanto a la disponibilidad de datos y en la facilidad de escalabilidad, pero si se realiza una escalabilidad bastante considerable es muy probable que el rendimiento se vea afectado, ya que será muy lento navegar por muchos nodos encontrando la consulta

cassandra-driver utiliza una política de balanceo de carga por defecto llamado DefaultLoadBalancingPolicy, el cual crea réplicas locales para una key determinada y si no está disponible, crea en el datacenter local nodos de forma round-robin.

- Cassandra posee principalmente dos estrategias para mantener redundancia en la replicación de datos. ¿Cuáles son estos? ¿Cuál es la ventaja de uno sobre otro? ¿Cuál utilizaría usted para en el caso actual y por qué? Justifique apropiadamente su respuesta.

Las dos estrategias principales para mantener la redundancia en la replicación de datos son: SimpleStrategy y NetworkTopologyStrategy. La principal ventaja que tiene NetworkTopologyStrategy sobre SimpleStrategy es que se puede almacenar múltiples copias en diferentes datacenters.

Dado la implementación actual que utiliza solo 1 datacenter, el cual contiene a todos los nodos necesarios para el desarrollo de esta actividad. Utilizaríamos la estrategia SimpleStrategy por el simple hecho de que se posee solo un datacenter, también SimpleStrategy coloca los nodos en el sentido de las agujas del reloj, esto sería la topología propuesta en el enunciado de la tarea.

- Teniendo en cuenta el contexto del problema ¿Usted cree que la solución propuesta es la correcta? ¿Qué ocurre cuando se quiere escalar en la solución? ¿Qué mejoras implementaría? Oriente su respuesta hacia el Sharding (la replicación/distribución de los datos) y comente una estrategia que podría seguir para ordenar los datos.

La solución propuesta es correcta, pero si consideramos que se implementa en una clínica con muchos pacientes y muchas recetas escritas a la vez, este tendría problemas de cuello de botella, ya que existe solamente un punto en donde se pueden realizar las peticiones.

Si se escala de forma vertical se tendría que aumentar la memoria de los nodos para que puedan almacenar más datos. Por otro lado si se quiere escalar de forma horizontal se tendría que crear otro cluster con los mismos nodos que el primero y conectarlo con el culster anterior, esto produciría otro datacenter, por lo que se deberá aplicar la estrategia de NetworkTopologyStrategy para mantener la redundancia en la replicación de datos.

Para realizar mejoras a la topología actual implementaríamos un sistema de Sharding, incrementando los cluster, de tal forma que haya una replicación por cada uno y al mismo tiempo un balanceo de carga entre ellos. Por ejemplo:

![Alt text](images/topologia.png "topología")

Los datos se distribuyen por los shard que hay en la topología. El config server almacena las rutas de los datos que contiene cada shard y aplica la política de distribución. Además todos los cluster contienen réplicas para la disponibilidad de los datos.

Antes de realizar la petición, primero la API REST debe consultar a config server para saber cuál shar se la debe realizar.

---
[![Alt text](https://pbs.twimg.com/media/FQ3Ei63XwAAFH8c.jpg)](https://www.youtube.com/watch?v=qde6w2b2-Yo)
____
## Uso de docker compose en cassandra


[![Alt text](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSvtcqf3wp4ZgUbF8narExnf44rNq48En-ZEB0V1dZEW0tWtXa3WmVZrnzk-EL9ZftB8vU&usqp=CAU)](https://www.youtube.com/watch?v=rSOjLD3C778)

_(La motivación, es un placer que solo los soñadores tenemos...)_