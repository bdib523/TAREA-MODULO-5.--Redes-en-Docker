Práctica: Creación y conexión de redes en Docker
1. Objetivo
El objetivo de esta práctica es crear dos redes Docker de tipo
BRIDGE, conectar dos contenedores Ubuntu a redes diferentes y
comprobar que inicialmente están aislados. Después, se conectará la
segunda red al primer contenedor para permitir la comunicación entre
ambos.
2. Crear las redes Docker
Crear red1
Se crea una red de tipo bridge con una dirección de red y una puerta de
enlace configuradas manualmente:
docker network create -d bridge --subnet=172.28.0.0/16 --gateway=172.28.0.1 red1
Datos de red1:
- Nombre: red1
- Subred: 172.28.0.0/16
- Máscara: 255.255.0.0
- Gateway: 172.28.0.1
Crear red2
En esta red, Docker asigna automáticamente la configuración de red:
docker network create -d bridge red2
Comprobar las redes
docker network ls
Para consultar la configuración detallada:
docker network inspect red1
docker network inspect red2
En esta práctica, Docker asignó a red2 la subred 172.18.0.0/16 y el
gateway 172.18.0.1.
3. Crear el primer contenedor: u1
Se crea un contenedor Ubuntu 20.04 conectado a red1, con hostname e IP
fija:
docker run -dit --name u1 --hostname host1 --network red1 --ip 172.28.0.10 ubuntu:20.04
Configuración:

  Parámetro               Valor
  Nombre del contenedor   u1
Imagen                  ubuntu:20.04
Hostname                host1
Dirección IP            172.28.0.10
Red                     red1

Si la imagen no está descargada, Docker la obtiene automáticamente del
repositorio de imágenes.
Instalar ping en u1
Entrar en el contenedor:
docker exec -it u1 bash
Actualizar los paquetes:
apt update
Instalar la utilidad ping:
apt install -y iputils-ping
Salir del contenedor:
exit
4. Crear el segundo contenedor: u2
Se crea otro contenedor Ubuntu 20.04, conectado únicamente a red2.
Docker le asigna automáticamente una dirección IP:
docker run -dit --name u2 --hostname host2 --network red2 ubuntu:20.04
Configuración:

  Parámetro               Valor
  Nombre del contenedor   u2
Imagen                  ubuntu:20.04
Hostname                host2
Dirección IP            Automática por Docker
Red                     red2
Instalar ping en u2
Entrar en el contenedor:
docker exec -it u2 bash
Actualizar los paquetes:
apt update
Instalar ping:
apt install -y iputils-ping
Salir:
exit
5. Comprobar los contenedores y las redes
Comprobar que los dos contenedores están en ejecución:
docker ps
Consultar la configuración de cada red:
docker network inspect red1
docker network inspect red2
En la configuración se puede comprobar que u1 está conectado a red1
y u2 está conectado a red2.
6. Comprobar el aislamiento entre los contenedores
Antes de conectar las redes, los contenedores están en redes bridge
diferentes.

Desde u1, probar la comunicación con u2 por nombre:
docker exec u1 ping -c 4 u2
Desde u2, probar la comunicación con u1 por dirección IP:
docker exec u2 ping -c 4 172.28.0.10
Desde u2, probar también por nombre:
docker exec u2 ping -c 4 u1
En las pruebas documentadas, la resolución de los nombres no funciona y
la prueba desde u2 hacia la IP de u1 termina con un 100 % de pérdida
de paquetes. Esto permite comprobar el aislamiento entre las dos redes.
7. Conectar red2 al contenedor u1
Para permitir la comunicación entre los contenedores sin conectar
directamente u2 a red1, se conecta también u1 a red2:
docker network connect red2 u1
Ahora la distribución queda así:

  Contenedor   Redes conectadas
  u1         red1 y red2
u2         red2

El contenedor u1 tiene conexión a las dos redes y puede comunicarse
con los contenedores conectados a cualquiera de ellas.
8. Comprobar la conectividad final
Desde u1, comprobar la comunicación con u2 mediante su nombre:
docker exec u1 ping -c 4 u2
Comprobar también la comunicación mediante la IP de u2:
docker exec u1 ping -c 4 172.18.0.2
En las capturas finales de la práctica se observa que las dos pruebas
reciben respuesta y terminan con un 0 % de pérdida de paquetes.
9. Resultado final
La práctica queda completada:
- Se han creado dos redes Docker de tipo bridge.
- Se han creado dos contenedores Ubuntu 20.04.
- Se ha instalado iputils-ping en ambos contenedores.
- Se ha comprobado el aislamiento inicial entre las redes.
- Se ha conectado red2 al contenedor u1 mediante
  docker network connect.
- Se ha comprobado que u1 puede comunicarse con u2 por nombre y
  por dirección IP.
