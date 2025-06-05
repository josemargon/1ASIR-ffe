# Proxmox

En este proyecto vamos a instalar 2 hipervisores de tipo 1 Proxmox y los uniremos en cluster

## Preparación de hyper-v

Para instalar el SO de Proxmox utilizaremos Hyper-V y añadiremos una nueva maquina virtual a la que asignaremos 6Gb de RAM, desactivaremos los puntos de control, asignaremos un switch por defecto para tener salida a internet y activaremos la virtualización de hiper-v a través de powershell.

![brr](Imagenes/ram.png)

![brr](Imagenes/puntodecontrol.png)

![brr](Imagenes/PS.png)

![brr](Imagenes/Switch.png)

## Instalación de Proxmox

Al iniciar el instalador de Proxmox escogemos la primera opción para instalar la versión gráfica.

![brr](Imagenes/i1.png)

Aceptamos los terminos y condiciones.

![brr](Imagenes/i2.png)

Cambiamos al modo sistema de archivos xfs.

![brr](Imagenes/i3.png)

Seleccionamos país, idioma del teclado y zona horaria.

![brr*](Imagenes/i4.png)

Establecemos una contraseña y un correo electronico.

![brr](Imagenes/i5.png)

En este paso debemos asegurarnos que cambiamos el hostname para que no se llamen pve los dos Proxmox, sino no podremos conectarlos al cluster.

![brr](Imagenes/i6.png)

Aqui vemos un resumen de lo que hemos ido seleccionando en la instalación y si está todo correcto pulsamos install.

![brr](Imagenes/i7.png)

Una vez finalizada la instalación apagamos la máquina, sacamos la iso y la volvemos a ejecutar.

![brr](Imagenes/disk.png)

Al volver a iniciarlo nos dará una ip que introduciremos en el navegador.

![brr](Imagenes/ip.png)

Para iniciar sesión usaremos como nombre de usuario "root" y como contraseña la que hayamos escrito durante la instalación.

![brr](Imagenes/log.png)

## Creación del cluster

Para crear el cluster iremos a la sección cluster y seleccionamos crear cluster, al crearlo nos saldrá un apartado llamado información de unión, deberemos copiar el párrafo.

![brr](Imagenes/c1.png)

Para conectar los dos Proxmox que hemos creado iremos ahora al segundo, donde seleccionaremos Unirse al cluster, en la ventana que se nos abrirá copiamos la información de unión, lo cual nos desbloqueará nuevos campos que rellenar, deberemos escribir nuestra contraseña y seleccionar la ip de la red del cluster.

![brr](Imagenes/c2.png)

Una vez los tengamos conectados tendremso una pantalla como esta

![brr](Imagenes/conectado.png)

## Ubuntu server

Instalaremos un ubuntu server minimal con 512 MB de RAM en hyper-v que actuará como servidor NFS para el cluster.

Para instalar el servidor NFS ejecutamos el siguiente comando.

```
sudo apt install nfs-kernel-server
```
Una vez instalado deberemos crear la carpeta que queremos compartir con nuestro Proxmox.

![brr](Imagenes/insnfs.png)

Configuramos y aplicamos el archivo /etc/exports con el directorio que hemos creado y las ip que tendrán acceso.

![brr](Imagenes/exports.png)

![brr](Imagenes/apex.png)

Para sincronizarlo con Proxmox debemos seleccionar desde este Almacenamiento > Agregar > NFS y al rellenar los datos pulsamos agregar.

![brr](Imagenes/NFS.png)

## Contenedor

Para poder tener acceso a internet en los contenedores deberemos crear un nuevo adaptador añadiendo lo siguiente al archivo /etc/network/interfaces y reiniciaremos el nodo antes de crear el contenedor.

```
auto vmbr1
iface vmbr1 inet static
    address 192.168.100.1/24
    bridge-ports none
    bridge-stp off
    bridge-fd 0
    post-up echo 1 > /proc/sys/net/ipv4/ip_forward
    post-up iptables -t nat -A POSTROUTING -s '192.168.100.0/24' -o vmbr0 -j MASQUERADE
    post-down iptables -t nat -D POSTROUTING -s '192.168.100.0/24' -o vmbr0 -j MASQUERADE
```


Para crear un contenedor vamos a necesitar la plantilla correspondiente a instalar. Para ello desde el almacenamiento que nos interese seleccionamos Plantillas de CT > Plantillas En este caso vamos a utilizar Debian

![brr](Imagenes/debian.png)

Para instalarlo seleccionamos en la esquina superior derecha Crear CT y se nos abrirá una ventana en la que iremos rellenando los datos que se nos pida para crear el contenedor.

![brr](Imagenes/insta.png)

![brr](Imagenes/ct.png)

Ahora iniciamos el contenedor y pulsamos iniciar para que arranque, para poder manejarlo seleccionamos  ">_ Consola".

![brr](Imagenes/brum.png)

Para comprobar el correcto funcionamiento de la red, se ha descargado el servidor ssh y me he conectado desde la máquina anfitriona a este.

```
apt install openssh-server
```

![brr](Imagenes/debssh.png)

## Instalar máquinas virtuales

Para instalar máquinas virtuales en proxmox descargaremos la iso que vayamos a utilizar, en este caso será alpine linux. Para cargarla en proxmox y poder utilizarla seleccionamos uno de los almacenamientos de los nodos > Imágenes ISO > Cargar

![brr](Imagenes/mv.png)

Con la ISO cargada en Proxmox, seleccionamos en la esquina superior derecha "Crear VM" y se nos abrirá una ventana similar a la que salió al crear los contenedores. Rellenamos los datos que se nos vayan pidiendo y pulsamos finalizar.

![brr](Imagenes/alpine.png)

Ahora podremos iniciar la máquina y desde el a partado consola podremos comprobar que está funcionando.

![brr](Imagenes/alpine2.png)

Realizamos los mismos pasos en el otro nodo pero con un SO diferente, en este caso tiny core linux.

![brr](Imagenes/tiny.png)

Instalamos un servidor nginx
y la comprobamos que podemos acceder a este.

![brr](Imagenes/nginx2.png)

## Migración

Existen dos tipos de migración, migración en caliente y en frio. La migración en caliente es pasar de un nodo a otro un SO estando este en funcionamiento mientras que enfrio el SO estará apagado, en ambos será necesario que la iso esté almacenada en el servidor NFS que he añadido previamente.
Para hacer una migración tan solo debemos seleccionar el SO que nos interese y pulsar el botón de la esquina superior derecha que pone "Migrar" o haciendo click derecho sobre lo que queramos migrar.

![brr](Imagenes/m1.png)

Seleccionamos el nodo al que vayamos a migrar la máquina o el contenedor.

![brr](Imagenes/m2.png)

Y asi se migraria una máquina virtual o un contenedor en frio entre nodos.

Ahora vamos a devolverla a su nodo original pero esta vez haciendo una migración en caliente, osea sé, encendida. El proceso sería el mismo pero nos indicaría en modo reinicio al hacerlo.

![brr](Imagenes/m4.png)

Aqui en el proceso se puede ver como durante la migración apaga el equipo, lo migra y lo vuelve a iniciar.

![brr](Imagenes/m5.png)

## Red interna entre máquinas

Crearemos un bridge desde la interfaz de red de cada nodo de Proxmox sin conexión a la interfaz física para que actue como red interna

Desde las maquinas configuraremos la conexion propia, estando ambas en la misma red.

En la primera máquina la configuramos con una red 192.168.100.10/24

En la segunda máquina configuramos la red 192.168.100.20/24

Ahora ambas máquinas están en la misma red y cuentan con un adaptador bridge para poder conectarse entre si, para comprobarlo solo hariamos un ping a la ip de la otra.

## Buckup

Para hacer un buckup debemos seleccionar la máquina o contenedor que deseemos y seleccionar Respaldo > Respaldar ahora

![brr](Imagenes/bu.png)

![brr](Imagenes/bu2.png)

Como resultado nos queda el backup pudiendo guardarlo tanto en local, como en nuestro servidor NFS.

![brr](Imagenes/bu3.png)

Ahora una vez eliminada la máquina se puede restaurar al estado anterior gracias al backup.

![brr](Imagenes/bu5.png)

![brr](Imagenes/bu6.png)

Para crear un backup automatico lo podemos configurar desde Centro de datos > Respaldo > Agregar, donde se nos abrira una ventana en la que podremos configurar todo lo necesario para ello.

![brr](Imagenes/bu4.png)