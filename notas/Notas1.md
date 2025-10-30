KEY VALUES
 	- workers need a single messaging app
 		- needs to be organized, easy to find information, centralized notifications, easy to communicate with all in the company

GAOLS
 	- Desing Xlack platform
 		- centralized messaging (groups, channels, boards)
 			- notifications, channels, direct messages, files containers

SCENARIOS
 	- case 1 ="#development" channel needs to have a functionality to interact with one message and responde to it without interfiering with the rest of the channel communication
 	- case 2 = direct messages, notifications, videocalls
 	- case 3 = #Projects" channel, whole app needs to have mentions using @, mentioned person should receive notification and be able to response, all of the other people can responde as well



select todos los mensajes donde parent_message_id sea NULO, ordenados por thread_last_reply_at descendientemente en lugar de tener que hacer una consulta propia para calcular cual es el ultimo reply al thread y después ordenar


TEXT

- Descripción -
 	Para solucionar el problema actual de mensajeria y notificaciones se plantea la solución de crear una aplicación SPA de mensajeria similar a slack y una API que se encargue de manejar la lógica de negocio(conexiones a base de datos, autenticación/autorización) , llamada "Xlack", con esta estructura, primero se contara con workspaces para mantener una estructura principal y poder identificar a donde pertenecen cada uno de los canales, seguido se contara con canales que puedan ser públicos o privados donde se podrá tener n cantidad de miembros los cuales serán capaces de enviar mensajes ya 	sea solo texto o con archivos, para solucionar las notificaciones, se creara un centro de notificaciones y se implementara un servicio en tiempo real (websocket) que se encargara de enviar las notificaciones en cuanto sean creadas.

- Puntos clave -
 	/MVP1/
 	. Para poder hacer uso de la aplicacion se deberan registar usuarios y hacer uso de un login.
 	. Los usuarios deberan tener un rol especifico ya sea como "admin", "moderador", "usuario", etc. para poder distinguir las acciones de cada uno.
 	. Los mensajes pueden ser enviados a otros usuarios mediante mensajes directos.
 	. Los usuarios se podran mencionar unos a otros usuarios con un "@" en los mensajes.
 	. Las notificaciones se podrán identificar cuando han sido leidas y cuando aun no.
 	. Se podrá tener threads en los mensajes y se llevara la cuenta de respuestas en un mismo thread para su uso en ui, y también será fácil identificar las respuestas en el thread asi mismo el sort de estas gracias a un atributo
      en la base de datos.
 	. Cuando una mención suceda se enviara una notificación especifica para el usuario mencionado.
 	. Si un usuario envia un mensaje con archivos estos deben ser guardados para despues obtener el url para obtenerlos en un futuro, esto puede ser en algo como s3.
 	. Los canales y workspaces podran ser creados mediante un "superuser" que tambien dara acceso a otros usuarios con funciones como "anadir usuario a workspace/canal".
- Mejoras -
 	/MVP2/
 	. Busqueda avanzada(mensajes, archivos)
 	. Estado (online/offline/ocupado/out of office)
 	. Edicion/eliminacion de mensajes
 	. Reacciones (quizas emojies o iconos personalizados)
 	. Anadir rol para "invitados" que puedan entrar a un workspace con acciones limitadas.
    . Anadir creacion de usuarios en batch
    . Anadir invitacion de usuarios a un workspace en batch
    . Anadir agregar usuarios a un canal en batch

- Ruta de implementacion v1 -
    - Backend Api v1
    . Diseno de api login/register, cruds(mensaje, users, channels, worspaces), implementacion de servicios de api, conexion a base de datos, implementar almacenamiento de archivos.
        - Modulo de autenticacion (login/signup)
        . Jwt, hash password, token verification
            - Endpoints: `POST: /register` `POST: /login`
            - Logica: 
                . Implementar registro de usuarios con validaciones en los atributos como estan en el entity diagram.
                . Usar Bcrypt para el hashing de las contrasenas.
                . Implementar construccion de JWT (JSON web tokens) tras un login exitoso. 
                . Implementar middleware para verificar tokens, si la verificacion es erronea enviar codigo http `unauthorized 401`, y que verifique los roles para saber si tiene acceso a el servicio solicitado.
        - Modulo de worspace
        . crud worspace
            - Endpoints: `GET: /workspace` `GET: /workspace/id` `POST: /workspace` `PUT: /workspace/id` `DELETE: /workspace/id` `GET: /workspace/id/channels`
            - Logica:
                . Implementar crud para poder manejar correctamente los worspaces.
        - Modulo hannel
        . crud channel
            - Endpoints: `GET: /channel` `GET: /channel/id` `POST: /channel` `PUT: /channel/id` `DELETE: /channel/id` `POST: /channel/channel_id/addUser/user_id` `POST: /channel/channel_id/removeUser/user_id`
            - Logica:
                . Implementar crud para poder manejar correctamente los channel.
                . Implementar servicio `channel_addUser` para anadir usuario a un channel en especifico.
                . Implementar servicio `channel_removeUser` para eliminar usuario de un channel en especifico.
                . Implementar logica para guardar en la base de datos cuando un usuario sea anadido a un canal en la entidad `ChannelUser` para saber en cuantos channel esta un usuario.
        - Modulo de Mensajeria
        . crud de mensajes
            - Endpoints: `GET: /message` `GET: /message/id` `POST: /message` `PUT: /message/id` `DELETE: /message/id`
            - Logica: 
                . Implementar crud para poder manejar correctamente los mensajes.
                . Implementar logica para la respuesta en `thread` de los mensajes usando el endpoint `POST: /message`, con el atributo `parent_message_id` que es el id del "mensaje padre" para poder identificarlos facilmente y hacer uso de la misma entidad en la base de datos y con los atributos `thread_reply_count` y `thread_last_reply_at` para una buena otimizacion en vistas.
                . Implementar logica en endpoint  `POST:/message`(si este tiene un archivo debio hacer uso del servicio de `POST: /attachment/generate-upload-url` para obtener un url pre-firmado para despues agregarlo al body).
                . Implementar logica de publicar un evento al websocket para transmitirlo al cliente.
        - Modulo Attachment
        . almacenamiento de archivos
            - Endpoint: `POST: /attachment/generate-upload-url`
            - Logica:
                . Implementar endpoint `POST: /attachment/generate-upload-url` para generar "urls pre-firmados" subiendo los archivos a un servidor de archivos como `s3` para poder usarlo en los mensajes con attachment antes de ser guardados en la base de datos.
        - Modulo de notificaciones
        . crud de notificaciones
            - Endpoints: `GET: /notification` `GET: /notification/id` `GET: /notification/user/id` `PUT: /notification/id`
            - Logica:
                . Implementar logica para la creacion de notificaciones.
                . Implementar logica para obtener todas las notificaciones en orden y que aun no son leidas con el endpoint `GET: /notification/user/id`.
                . Implementar logica del endpoint `PUT: /notification/id` para actaualizar la notificacion y con esto poder actualizar `read_at` para saber si volver a traer notificacion.
                . Implemenatr logica en los servicios de `POST: /message` y `PUT: /message/id` para dedectar las `@mention` del mensaje para enviar la notificacion personalizada de una mencion.
                . Implementar logica de publicar un evento al websocket para transmitirlo al cliente.
        - Modulo de websocket
        . implementacion de websocket para envio de notificaciones y mensajes
            - Logica:
                . Implementar servicio websocket para que este funcionando siempre y cuando exista una nueva notificacion enviarla al usuario que sea necesario, esto puede ser desde una mencion, un mensaje nuevo en un canal o un mensaje directo.
        - Modulo de Reacciones
        . implementar post/put/get de reacciones
            - Endpoints: `POST:/reactions/messages/message_id`, `DELTE:/reactions/reaction_id/messages/message_id`, `GET:/reactions/messages/message_id`
            - Logica:
                . Implementar endpoint `POST:/reactions/messages/message_id` para anadir una reaccion al mensaje, puede ser mediante caracteres ascii, emojis o iconos personalizados.
                . Implementar endpoint `DELTE:/reactions/reaction_id/messages/message_id` para eliminar una reaccion especifica.
                . Implementar endpoint `GET:/reactions/messages/message_id` para obtener todas las reacciones que tiene un mensaje.
    - FrontEnd v1
 	. Diseno de SAP para consumir la api principal, implementacion de vistas login/signup, implementacion de ui principal, que consuma la api para enviar mensajes.
        - Vistas de login / register
        - Vista "dashboard" con todos los modulos necesarios para mensajes directos, worspaces y canales, notificaciones 




Flow diagram
    NEW MESSAGE
        -> STARTS
        -> cliente "hace click" enviar mensaje
        -> client checks if has attachment?
            -> YES - Cliente usa servicio de la api `POST /attachments/generate-upload-url` para "pre-guardar" el archivo y obtener un "pre-signed" URL
                    -> se sube archivo a S3
                    -> cliente espera a recibir url y lo anade al body
                    -> cliente llama a `POST /channels/channel_id/messages`
            -> NO - 
                -> cliente llama a `POST /channels/channel_id/messages`
        -> server starts db transaction
        -> server checks if body has parent_message_id (message is a thread response)
            -> YES - server actualiza/anade thread_reply_count y thread_last_reply_at 
                -> server guarda message with parent_message_id included
                -> crea "objeto" de websocket "THREAD_RESPONSE"
            -> NO -> server inserta en db nuevo mensaje con parent_message_id=null
                -> crea "objeto" de websocket "NEW_MESSAGE"
        -> server checks if message has mention?
            -> YES - servidor de api obtiene los usuarios mencionados
                -> insert en bd notificacion
                -> crea "objeto" de websocket "MENTION"
            -> NO -> sigue logica
        -> server commit db transaction
        -> server checks if db transaction is succesfull
            -> YES  -> sends websocket events
            -> NO - Responde con error 500 por fallo en el guardado de la base de datos
        ->ENDS

Flow diagrams considerados

USER1 -> send message to channel -> message save in db -> message display in channel

USER1 -> send message with attachment -> message save in db, attachment save in s3 -> message display in channel

USER1 -> sends message with mention -> server recieves petition -> message save in db, notification save in db -> user2 receibes notification -> user2 cliks notification -> notification update in db read_at

USERADMIN -> creates channel1
USERADMIN -> adds members to channel1

USER3 -> receibes notification from Chanel1 add