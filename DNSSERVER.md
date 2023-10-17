# dnsservidor-cliente

En esta práctica se requiere de un contenedor en docker que actue como servidor DNS a un cliente. Paras ello usaremos docker compose para crear ambos contenedores [se usa la configuración de red de la práctica anterior y la red la práctica anterior, los archivos named.conf, named.conf.local, named.conf.options y la base de datos db.asircastelao.int serán los mismos]

En docker compose usaemos esta sintaxis:

services:
  bind9:
    image: ubuntu/bind9
    container_name: servidor

    ports:
      - "53:53/tcp"
      - "53:53/udp"
    networks:
      bind9_subnet_asir2:
        ipv4_address: 172.28.5.1
    volumes:
      - ./conf:/etc/bind 
      - ./zonas:/var/lib/bind
    environment:
      - TZ=Europe/Paris

  cliente:
    container_name: cliente
    image: alpine
    tty: true
    stdin_open: true
    dns:
      - 172.28.5.1
    networks:
      bind9_subnet_asir2:
        ipv4_address: 172.28.5.2

networks:
  bind9_subnet_asir2:
    external: true

 Como se ve, se usa la misma red y la dns será el servidor.

 Los lanzamos con: 

 $ docker compose up -d

 Una vez arrancados, para acceder a ellos y a sus terminales pondremos:

 # Para el servidor

 $ docker exec -it servidor bash   

 # Para el cliente

 $ docker exec -it cliente sh

 # Ping y dig desde el servidor

 $ apt update
 $ apt install dnsutils
 $ apt-get install -y iputils-ping

 # Dig y ping en el cliete

 $ apk add --no-cache bind-tools


 Una vez hecho esto se comprobara que tanto servidor como cliente se hacen ping entre sí

 Para usar dig es muy sencillo, si queremos resolver una ip de la base de datos de db.asircastelao.int pondremos en el servidor o el cliente @ seguido de la ip y el nombre del dominio, por ejemplo la base de datos tiene la siguiente información, si ponemos el siguiente comando --> "dig @172.28.5.1 cosas.danielcastelao.int" 
 debe devolvernos (resolvernos) la siguiente IP --> 172.28.5.9

 $TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.asircastelao.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS	ns.asircastelao.int.
ns		IN A		172.28.5.1
test	IN A		172.28.5.4
www     IN A        172.28.5.7
cosas   IN A        172.28.5.9
alias	IN CNAME	test
texto	IN TXT		mensaje