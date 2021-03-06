# COMUNICACIONES I – PROYECTO PARCIAL II

## REGLAS


1. Formas de iniciar el juego en la primera ronda (por jerarquía):
    * Tener el doble 6.
    * Tener el doble más alto.
    * Tener la ficha con mayor pinta.
    * Al azar.
2. El juego es individual.
3. El ‘sentido’ del juego será en el orden de conexión de los jugadores.
4. Las rondas serán de 2 a 4 jugadores.
5. Son 7 fichas por jugador.
7. Si el juego inicia con menos de 4 jugadores, el resto de fichas se descarta.
8. Si el jugador que tiene la mano pasa (en algún momento de la ronda), ahora la mano la tendría el siguiente jugador, siguiendo el sentido del juego.
9. Si no tiene fichas apropiadas para jugar, debe pasar.
10. En caso de una jugada errónea, el jugador es sacado de la partida y su mano se descarta.
11. No hay pozo.
12. Si un jugador se desconecta, se retira del juego.
13. No se permite reconexión.
14. La mano del jugador que se desconecta se descarta.
15. Una partida se gana cuando alguno de los jugadores llega a 100 puntos (independientemente el número de rondas).
16. Formas de ganar una ronda:
    * Dominó la ronda (se le acaben las fichas).
    * Que el juego se tranque y tenga la mejor puntuación, esto es, el menor número de pintas.
    * Ser el último jugador en la mesa (por desconexión del resto de jugadores).
17. Formas de desempate con el juego trancado (por jerarquía):
    * Gana el jugador que trancó.
    * Gana el jugador que tenga la mano (o el más cercano a él, siguiendo el sentido del juego).
    * El puntaje del ganador en una ronda será la suma de las pintas de los demás jugadores.

## PROTOCOLO
### Contemplaciones:
- El servidor es quien genera las fichas.
- El servidor y el cliente llevan los puntajes generales internamente y solo se intercambia lo establecido en el protocolo.
- Las conexiones unicast serán TCP.

Durante las comunicaciones del juego se enviarán exclusivamente mensajes codificados como JSON.

Cada objeto de tendrá en su body una clave identificador con el valor DOMINOCOMUNICACIONESI. Ejemplo:
```json
{	
    "identificador": "DOMINOCOMUNICACIONESI"
}
```

**El servidor inicia una mesa de la siguiente forma:**
* El servidor emitirá mensajes a través de UDP Broadcast dirigidos al puerto 3001 de los clientes activos. La emisión se hará cada 5 seg mientras la mesa se encuentre esperando jugadores, para evitar la congestión de la red. Estos mensajes contendrán el nombre de la mesa. Ejemplo:

```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
    "nombre_mesa": "Nombre de la mesa definido por el servidor"
}
```

* El servidor esperará la conexión TCP de los jugadores a través del puerto 3001, hasta que se cumplan las condiciones necesarias para iniciar el juego. Luego de iniciado el juego se rechazarán las conexiones entrantes.

El cliente ha de escuchar los mensajes broadcast para conocer las mesas disponibles, luego selecciona la mesa a la que se quiere conectar, intenta conectarse y si su conexión no es rechazada envía un mensaje unicast TCP al servidor. Ejemplo:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
    "nombre_jugador": "Nombre del jugador definido por el cliente"
}
```


El servidor responde al unicast del cliente con un mensaje para confirmar la conexión. Ejemplo:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"multicast_ip": "IP multicast definida por el servidor",
	"jugador" : "Identificador que la mesa genera para el jugador al que le envia estas fichas."
}
```

*Nota:* El servidor espera 30 segundos, a partir del ingreso del segundo jugador, y reiniciando dicho conteo cada vez que entra alguien. El juego se inicia cuando el conteo es excedido o cuando la mesa se llena.

### Mensajes de juego

#### Mensajes del servidor

Luego de que se tienen todos los jugadores en la mesa el servidor enviará solo mensajes de juego, dichos mensajes serán multicast (excepto el de asignación de fichas) y se podrán identificar a través de un campo denominado ```tipo```.

**Tipos de mensajes del servidor**

Valor | Tipo
----- | ----
0 | mensaje de inicio de juego
1 | mensaje de inicio de ronda
2 | mensaje de asignación de fichas
3 | mensaje de control de juego
4 | mensaje de fin de ronda
5 | mensaje de fin de partida
6 | mensaje de desconexión

Ejemplos:

##### Mensaje de inicio de juego

Este mensaje se envía cuando la mesa decide que debe iniciarse el juego. Contiene los jugadores que se hallan en la mesa.
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
    "tipo" : 0,
    "jugadores": [
        {
            "identificador" : "identificador_jugador_1", 
            "nombre" : "nombre que el jugador 1 le envió al servidor"
        },
        {
            "identificador" : "identificador_jugador_2", 
            "nombre" : "nombre que el jugador 2 le envió al servidor"
        },
        {
            "identificador" : "identificador_jugador_3", 
            "nombre" : "nombre que el jugador 3 le envió al servidor"
        },
        {
            "identificador" : "identificador_jugador_4", 
            "nombre" : "nombre que el jugador 4 le envió al servidor"
        }
    ]
}
```
##### Mensaje de inicio de ronda

Contiene el número de ronda actual. Es multicast.

```json
{
    "identificador": "DOMINOCOMUNICACIONESI",
    "tipo" : 1,
    "ronda" : 1
}
```

##### Mensaje de asignación de fichas

Este mensaje se envía por unicast TCP. Contiene las fichas asignadas a cada jugador.
```json
{
    "identificador": "DOMINOCOMUNICACIONESI",
    "tipo" : 2,
    "fichas": [
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 1
        },
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 2
        },
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 3
        },
        {
            "token": "estoesuntoken",
            "entero_uno": 6,
            "entero_dos": 4
        }
    ]
}
```

*Nota:* el array de fichas siempre debe contener 7 fichas para que cada jugador pueda comenzar la partida. Además, se envía un token único con la finalidad de que las jugadas de los clientes sean mediante el envío de dicho token y no con los valores. El servidor será quien valide estas condiciones
 
##### Mensaje de control de juego

Este mensaje es el que usará el servidor para definir quién jugó la última vez y a quién le toca jugar.

Los campo que contiene son:
* ```identificador``` => el identificador del protocolo ```"DOMINOCOMUNICACIONESI"```.
* ```jugador``` => Identificador del jugador que debe iniciar o que tiene el turno.
* ```tipo``` => el tipo para este mensaje será 3.
* ```punta_uno``` y ```punta_dos``` indican los números al inicio y final de la lista de fichas jugadas.
* ```evento_pasado``` contiene la información de la jugada anterior, dicha jugada se podrá identificar con su tipo:
    * 0: jugada normal
    * 1: jugada errónea
    * 2: pasó

Ejemplo:

Mensaje de juego:
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que debe iniciar o que tiene el turno",
	"tipo": 3,
	"punta_uno": "Número inicial de la lista o -1",
    "punta_dos": "Número final de la lista o -1",
    "evento_pasado": {
            "tipo": 0,
            "jugador": "Identificador de quien jugó",
    	    "ficha": { 
		        "entero_uno": 6, 
    		    "entero_dos": 6
            },
    		"punta": "true //Booleano indicando el lugar de juego. True: punta uno. False: punta dos "
    } 
}
```
*Nota:* si es el inicio de la ronda punta_uno y punta_dos serán -1

##### Mensaje de fin de ronda

Este mensaje indica el final de una ronda, su ganador y su puntuación. Es multicast.
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que ganó la ronda",
	"tipo": 4,
	"puntuacion": "Puntuación del ganador de la ronda",
	"razon": "Descripción del fin de la ronda. Máximo 45 caracteres"
}
```

##### Mensaje de fin de partida

Este mensaje indica el final de una partida y las puntuaciones de todos los jugadores. Es multicast.
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que ganó la partida",
	"tipo": 5,
	"puntuacion_general": [
        {
            "jugador": "Identificador del jugador",
            "puntuacion": "Puntuación del jugador"
        }
    ],
    "razon": "Descripción del fin de la partida. Máximo 45 caracteres"
}
```

##### Mensaje de desconexión

Este mensaje indica si algún jugador se desconectó de la mesa durante el juego. Es multicast.
```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador que se desconectó",
	"tipo": 6
}
```

#### Mensajes del jugador

Cada jugador justo después del mensaje de juego enviará un mensaje unicast con la información de su jugada, esto si está en turno, si no, un mensaje para confirmar que sigue en línea. Ejemplo:

##### Jugador en turno y jugando

```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"ficha": { 
        "token": "estoesuntoken"  
        },
	"punta": "Booleano indicando el lugar de juego. True: punta uno. False: punta dos"
}
```

*Nota:* Si el token es -1, significa que el jugador quiere pasar.

##### *Deprecated*: Jugador que no está en turno (mensaje unicast) [Se sugiere su omisión]

```json
{
	"identificador": "DOMINOCOMUNICACIONESI",
	"jugador": "Identificador del jugador"
}
```

*Nota:* el servidor espera este mensaje durante 10 segundos, si no hay respuesta asume que el cliente se desconectó. También, este mensaje será igual para cada turno.
