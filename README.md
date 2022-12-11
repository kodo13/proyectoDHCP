# Proyecto DHCP

## Creación del fichero docker-compose.yml

```
version: '3'
services:
  dhcpd:
    # docker-compose settings
    container_name: isc-dhcp-server
    network_mode: "host"
    # docker swarm settings
    image: networkboot/dhcpd
    volumes:
      - ./data:/data
```

Usamos el network_mode: "host". De este manera, la pila de red de ese contenedor no está aislada del host de Docker (el contenedor comparte el espacio de nombres de red del host) y el contenedor no obtiene su propia dirección IP asignada. Por ejemplo, si ejecuta un contenedor que se une al puerto 80 y utiliza host la red, la aplicación del contenedor está disponible en el puerto 80 en la dirección IP del host.

## Configuración fichero dhcpd.conf

```
subnet 10.0.2.0 netmask 255.255.255.0 {    
    range 10.0.2.100 10.0.2.200;    
    option routers 10.0.2.1;    
    option domain-name-servers 10.0.2.10, 10.0.2.11;    
}


subnet 10.0.9.0 netmask 255.255.255.0 {
#  range 10.0.9.150 10.0.9.159;
  option domain-name-servers 8.8.8.8, 193.146.92.2, 193.146.96.3;
  option domain-name "asir.tv";
  option routers 10.0.9.254;
  option broadcast-address 10.0.9.255;

  default-lease-time 600;
  max-lease-time 7200;
}

host kleo {hardware ethernet 08:00:27:ce:3f:74;
fixed-address 10.0.9.28;
}

group {
    default-lease-time 604800;
    max-lease-time 691200;

}
```

Una vez lista la anterior configuración podemos arrancar el servicio. Si es la primera vez que arrancamos el servidor DHCP, fallará si no existe el archivo dhcpd.leases. Lo podemos crear mediante comando o interfaz.

Para arrancar el servicio, ejecutamos ***sudo docker-compose up***. Si hacemos cualquier modificación en el archivo de configuración, tendremos que parar y volver a levantar el servicio para que se apliquen los cambios.

En el fichero de dhcpd.leases, podemos ver todas las IP que ha asignado nustro servidor DHCP.


## Apartados a configurar

1. Configurar IP fija en la red interna

Para establecer una IP fija, tenemos que editar el fichero de dhcpd.conf. Aquí, en option routers, añadimos la IP fija.

2. Configuración interfaz de escucha

El puerto por defecto de escucha del servidor DHCP es el 67. Si no se le indica otro, este hará la escucha en el predeterminado, el 67, y las respuestas en el 68.

3. Adición de rangos IPs

Para poner los rangos disponibles para asignar las IPs, escribimos después de escribir la red la opción de range seguida del rango que queramos. 

Importante asegurarnos de dar un rango libre para que no se sobrepongan unas IPs sobre otras.

4. Configuración IP fija para el cliente

Para configurar una IP fija para el cliente, lo haremos mediante un filtrado MAC. Para ello tenemos que saber la MAC del cliente y una vez la tengamos, simplemente añadimos el host con una IP que le queramos dar.

```
host kleo {hardware ethernet 08:00:27:ce:3f:74; #MAC del cliente
fixed-address 10.0.9.28; # IP fija asignada
}
```

## Comprobación funcionamiento server DHCP

Antes de nada, en el server nos aseguramos de tener dos interfaces de red, una en NAT y otra en adaptador puente.

Para comprobar que funciona el server DHCP, cogemos cualquier otra máquina que tengamos y la arrancamos. Ponemos la interfaz de red en adaptador puente para que sean accesibles.

Como la mñaquina cliente ya tiene asignada una IP, ejecutamos el comando `sudo dhclient -r` para borrar la IP que tiene asignada. Después, ejecutamos el comando `sudo dhclient` para forzar que pida de nuevo IP al DHCP. 

En el registro del nuestro server DHCP podemos ver este proceso de request y de offer, así como la asignación de una IP, apareciendo como DHCPACK.

![Imagen](https://github.com/kodo13/proyectoDHCP/blob/main/images/Captura%20de%20pantalla%202022-12-07%20180838.png?raw=true)
>Vemos que se ha asignado la IP 10.0.9.28 a la MAC que parece en pantalla, que es la que hemos puesto anteriormente en la configuración del filtrado por MAC.



