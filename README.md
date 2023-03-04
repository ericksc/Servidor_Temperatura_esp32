# Estación meteorológica en línea basada en MicroPython en ESP8266 y ESP32

MicroPython es uno de los mejores firmware de microcontrolador que admite una variedad de plataformas integradas. 

Las placas de desarrollo WiFi como ESP8266 y ESP32 se encuentran entre los puertos compatibles con MicroPython. Con MicroPython, es extremadamente simple implementar aplicaciones IoT de primer nivel.  MicroPython tiene un amplio soporte para la programación de redes junto con la implementación de todas las funciones básicas de hardware. 

Combina soporte de hardware de bajo nivel con características de programación de alto nivel, que los ecosistemas de microcontroladores integrados basados en C generalmente no tienen. Además, la sintaxis de Python hace que la programación incrustada sea súper simple, limpia y precisa.


En este proyecto, desarrollaremos una estación meteorológica en línea configurada sobre ESP8266 / ESP32. 

El proyecto utiliza ESP8266 como microcontrolador al obtener lecturas de sensores de DHT-11, mientras que al mismo tiempo, se beneficia de las funciones de programación de red de MicroPython para conectarse y operar dentro de una red IoT. 

Esta estación meteorológica está operativa dentro de los límites de una LAN inalámbrica y transmite temperatura y humedad como un servidor TCP en funcionamiento dentro de la red.

## Requisitos previos

Antes de continuar, debe haber configurado uPyCraft IDE o Thonny IDE como entorno de desarrollo de software. 

Además, debe haber cargado el firmware de MicroPython a ESP8266 / ESP32. 

## Componentes necesarios
1.	ESP8266/ESP32 x1
2.	Cable USB para conectar ESP con el ordenador
3.	DHT11 x1
4.	Placa de pruebas x1
5.	Cables de conexión/cables de puente
6.	Ordenador/Móvil x1

## Conexiones de circuitos

En este proyecto, ESP8266/ESP32 actúan como un servidor TCP que aloja una página web HTML simple. El sensor DHT11 está interconectado con ESP8266/ESP32. El tablero lee la temperatura y la humedad de DHT11 y publica las lecturas en la página web alojada. Las conexiones de circuito solo son necesarias para interconectar DHT11 con ESP8266/ESP32. DHT11 tiene el siguiente diagrama de pines.

![image](https://user-images.githubusercontent.com/20059518/222920167-f6c221f9-5412-4ad7-a295-4b01c77e97d1.png)

Para interconectar DHT11 con ESP8266, conecte los pines VCC y GND de DHT11 con el 3V y GND de ESP8266. Conecte el pin de datos de DHT11 con cualquier GPIO de ESP8266. En este proyecto, el pin de datos de DHT11 está conectado a GPIO14 (D5) de ESP8266. El pin de datos se eleva conectando una resistencia de 10K entre la salida de datos y VCC.


![image](https://user-images.githubusercontent.com/20059518/222920188-fdb9d79e-6d28-4b3a-abfe-84ff3d674001.png)

Si se utiliza ESP32, D14 es el GPIO14 en ESP32. ESP32 tendrá las siguientes conexiones de circuito.

![image](https://user-images.githubusercontent.com/20059518/222920202-99919b82-bc9f-49ef-9e38-bd5f7f015077.png)

Después de las conexiones de circuito, el servidor TCP ESP8266 tendría el siguiente aspecto.

![image](https://user-images.githubusercontent.com/20059518/222920218-37883a0a-7f30-4a2e-a80b-3f3b3bcf4e02.png)

## Cómo funciona

el proyecto Esta estación meteorológica en línea basada en ESP8266/ESP32 lee la temperatura y la humedad del sensor DHT11 y publica las lecturas en una página web en tiempo real. La página web está alojada por el propio ESP8266/ESP32, para lo cual, la placa está configurada como un servidor TCP. Se puede acceder a la página web desde cualquier dispositivo, computadora o móvil que tenga un navegador para acceder a Internet a través del protocolo HTTP. La página web está alojada localmente dentro de LAN inalámbrica, por lo que solo los dispositivos conectados a la misma conexión WiFi pueden acceder a la página web.
El funcionamiento del proyecto es sencillo. La placa lee la temperatura y la humedad de DHT11 utilizando el módulo DHT incorporado de MicroPython. La placa se conecta con una conexión WiFi disponible utilizando el módulo de red de MicroPython. Para alojar una página web y operar como un servidor TCP, se utiliza el módulo de socket de MicroPython. ESP8266/ESP32 lee la temperatura y la humedad e incrusta las lecturas dentro de la página web. La página web se devuelve como una respuesta HTTP a cualquier dispositivo que acceda a la dirección IP del host local.


## El código

```Python
try:
  import usocket as socket
except:
  import socket

import network
from machine import Pin
import dht

import esp
esp.osdebug(None)

import gc
gc.collect()

ssid = 'SSID'
password = 'PASSWORD'

station = network.WLAN(network.STA_IF)

station.active(True)
station.connect(ssid, password)

while station.isconnected() == False:
  pass

print('Connection successful')
print(station.ifconfig())

sensor = dht.DHT11(Pin(14))
temp = 0
hum = 0


def read_dht():
  global temp, hum
  temp = hum = 0
  try:
    sensor.measure()
    temp = sensor.temperature()
    hum = sensor.humidity()
    if (isinstance(temp, float) and isinstance(hum, float)) or (isinstance(temp, int) and isinstance(hum, int)):
      msg = (b'{0:3.1f},{1:3.1f}'.format(temp, hum)
      hum = round(hum, 2)
      return(msg)
    else:
      return('Invalid sensor readings.')
  except OSError as e:
    return('Failed to read sensor.')

def web_page():
  html = """<!DOCTYPE HTML><html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.2/css/all.css" integrity="sha384-fnmOCqbTlWIlj8LyTjo7mOUStjsKC4pOpQbqyi7RrhN7udi9RwhKkMHpvLbHG9Sr" crossorigin="anonymous">
  <style>
    html {
     font-family: Arial;
     display: inline-block;
     margin: 0px auto;
     text-align: center;
    }
    h2 { font-size: 3.0rem; }
    p { font-size: 3.0rem; }
    .units { font-size: 1.2rem; }
    .dht-labels{
      font-size: 1.5rem;
      vertical-align:middle;
      padding-bottom: 15px;
    }
  </style>
</head>
<body>
  <h2>ESP DHT Server</h2>
  <p>
    <i class="fas fa-thermometer-half" style="color:#059e8a;"></i> 
    <span class="dht-labels">Temperature</span> 
    <span>"""+str(temp)+"""</span>
    <sup class="units">&deg;C</sup>
  </p>
  <p>
    <i class="fas fa-tint" style="color:#00add6;"></i> 
    <span class="dht-labels">Humidity</span>
    <span>"""+str(hum)+"""</span>
    <sup class="units">%</sup>
  </p>
</body>
</html>"""
  return html

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.bind(('', 80))
s.listen(5)

while True:
  conn, addr = s.accept()
  print('Got a connection from %s' % str(addr))
  request = conn.recv(1024)
  print('Content = %s' % str(request))
  temphum = read_dht()
  print(temphum)
  response = web_page()
  conn.send('HTTP/1.1 200 OK\n')
  conn.send('Content-Type: text/html\n')
  conn.send('Connection: close\n\n')
  conn.sendall(response)
  conn.close()
```


El código comienza con la importación de un módulo usocket. 

En caso de que un módulo usocket no esté disponible, se importa el módulo socket predeterminado. A continuación, se importan el módulo de red, la clase pin del módulo de máquina y el módulo dht. El módulo de red se utiliza para conectarse con una conexión WiFi. La clase de pin del módulo de la máquina se utiliza para interactuar con el sensor DHT11. El módulo dht se utiliza para obtener lecturas de DHT11. 

El módulo esp se importa y los errores de nivel de sistema operativo se suprimen llamando al método esp.osdebug(None). El módulo gc se importa y la recolección de elementos no utilizados se habilita llamando al método gc.collect(). Esto libera recursos de microcontrolador (RAM) tan pronto como las variables y los objetos no se utilizan.

El SSID y la clave de red de la conexión WiFi disponible se almacenan en variables SSID y contraseña. Deberá reemplazar el SSID y la contraseña WiFi con el SSID y la clave de red de su propia conexión WiFi en las siguientes líneas de código.

```Python
ssid = 'SSID'
password = 'PASSWORD'
```

Una red de objetos. Se crea una instancia de la clase WLAN(), configurando ESP8266/ESP32 como una estación WiFi. El ESP8266/ESP32 como estación WiFi se activa llamando al método station.active(). 

La placa se conecta a una conexión WiFi disponible llamando al método station.connect(). La conexión con la red WiFi se verifica mediante el método station.isconnected() que llama a station.isconnected(). En una conexión correcta con la red WiFi, se imprime un mensaje en la consola y los parámetros de configuración de nivel IP también se imprimen en la consola mediante el método station.ifconfig() que realiza la llamada. El primer parámetro de nivel IP impreso en la consola es la dirección IP de la estación ESP8266/ESP32.


Se declara un sensor variable para crear instancias del objeto de la clase DHT. Las variables temperatura y zumbido se declaran para almacenar lecturas de temperatura y humedad. Se define una función definida por el usuario read_dht() para obtener lecturas de temperatura y humedad de DHT11. La función llama al método measure() para obtener lecturas del sensor DHT11. Utiliza métodos de temperatura () y humedad () para obtener valores de temperatura y humedad, respectivamente, como flotante o entero. Los valores de temperatura y humedad se devuelven en variables globales temp y hum, respectivamente.

Se define una función definida por el usuario web_page() para devolver una página HTML con valores de temperatura y humedad almacenados en variables globales temp y hum incrustados dentro de la página HTML. Se crea una instancia de un objeto de clase de socket y se configura para utilizar la dirección IPv4 y TCP para el protocolo de capa de red. El objeto se enlaza a la dirección localhost mediante el método bind() y se activa para escuchar desde clientes TCP llamando al método listen().

Se inicia un bucle de tiempo infinito. En el bucle, el servidor TCP ESP8266/ESP32 está configurado para aceptar solicitudes HTTP de clientes TCP de la misma red WiFi llamando al método s.connect(). Las solicitudes HTTP de un cliente TCP conectado se almacenan en una solicitud variable llamando al método conn.recv(). 

El mensaje que contiene las lecturas de temperatura y humedad se imprime en la consola, y la página web que contiene los valores de temperatura y humedad incrustados se devuelve como respuesta llamando a los métodos conn.send() y conn.sendall(). Después de devolver la página web como respuesta, la conexión con el cliente TCP se cierra explícitamente llamando al método conn.close().

## Resultado

Se accede a la página web con lecturas de temperatura y humedad del sensor DHT11 escribiendo la dirección IP devuelta por el método station.ifconfig() en la barra de direcciones de cualquier navegador. El servidor TCP ESP8266/ESP32 que funciona como una estación meteorológica en línea devuelve una página web personalizada como respuesta a una solicitud HTTP del navegador de su computadora / móvil. 

La siguiente captura de pantalla muestra la página web devuelta por la estación meteorológica en línea ESP8266/ESP32.

![image](https://user-images.githubusercontent.com/20059518/222920234-0c1386a8-cc21-4371-9c7c-dc998beb1b64.png)
