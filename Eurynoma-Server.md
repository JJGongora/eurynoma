# Documentación - Eurynoma Server

- [Introducción](#introducción)
- [Configuración inicial](#configuración-inicial)
    - [Particionado de discos](#particionado-de-discos)
    - [Usuarios](#usuarios)
    - [Grupos](#grupos)
    - [Configuración de red](#configuración-de-red)
- [Configuración del servidor](#configuración-de-servidor)
    - [Nginx](#nginx)
        - [Instalación](#instalación)
        - [Configuración de proxy hosts](#configuración-de-proxy-hosts)
    - [WireGuard](#wireguard)

## Introducción
Este archivo contiene la documentación sobre el funcionamiento del servidor principal de Eurynoma Software Solutions: Eurnyoma. Este servidor aloja los servicios de red básicos utilizados por Eurynoma Solutions.

## Configuración inicial

### General
- Instalar SSH Server y las utilidades estándar del sistema.

### Particionado de discos
- El esquema de particionado deberá separar las particiones /home, /var y /tmp.
- Se usará EXT4 como formato de particionado.

### Usuarios
Los siguientes usuarios tendrán acceso al servidor. Las contraseñas no se encuentran en este archivo de documentación.

- __jjgongora__: Este usuario controlará la infraestructura del sistema.
- __root__: Este usuario tiene control total sobre el sistema. Solo utilizado en casos de emergencia.
- __euryguest__: Usado para invitados. Acceso limitado a la lectura de algunos directorios.
- __euryvpn__: Usado para el servicio de WireGuard.

~~~
root@eurynoma:~# sudo adduser euryguest
root@eurynoma:~# sudo adduser euryvpn
~~~

### Grupos

- __coreadmins__: Este grupo tiene acceso a los directorios de configuración de Nginx, Wireguard, AdGuardHome y el archivo de configuración DDNS.


### Configuración de red
Este servidor usará la dirección `192.168.1.10`, con la máscara de subred `255.255.255.0`.

## Configuración de servidor

### Nginx

#### Instalación
Para instalar Nginx, siga los pasos a continuación indicados o véalos en [Nginx Installation for Debian](https://nginx.org/en/linux_packages.html#Debian).

1. Instalar los prerrequisitos con el comando:
~~~
sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring certbot python3-certbot-nginx
~~~

2. Importar una llave de acceso oficial de Nginx para verificar la autenticación de los paquetes con el comando:
~~~
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \ | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
~~~

3. Verificar que los archivos descargados contengan la llave:
~~~
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
~~~

- Si fue correcto, la terminal mostrará la fingerprint `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62` como se muestra a continuación:
    
        ~~~
        pub   rsa2048 2011-08-19 [SC] [expires: 2027-05-24]
            573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
        uid                      nginx signing key <signing-key@nginx.com>
        ~~~

4. Para configurar el repositorio apt para estabilizar los paquetes de Nginx, usar el siguiente comando:
~~~
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
~~~

5. Para usar los paquetes mainline de Nginx, usar el siguiente comando:
~~~
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
~~~

6. Configurar la fijación del repositorio para preferir los paquetes de Nginx sobre los proporcionados por la distribución (Debian):
~~~
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
~~~

7. Finalizar la instalación con:
~~~
sudo apt update
sudo apt install nginx
~~~

#### Configuración de proxy hosts
El siguiente ejemplo muestra una ruta de un host proxy configurado dentro del archivo `/etc/nginx/nginx.conf`:

~~~
server {
    listen 80;
    server_name backremote.eurynoma.com;

    location / {
        proxy_pass http://127.0.0.1:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
~~~

- Aquí, en la sección `listen` se ingresa el puerto a usar por esta ruta. En este caso, el `80`.
- En `server_namne` ingresamos la dirección (subdominio/dominio) de la ruta, la dirección que se usará para acceder a la misma desde otros dispositivos.
- En `proxy_pass` ingresamos la ruta del servicio al que deseamos dirigir la ruta.

Después, para verificar que no haya errores en la sintaxis de Nginx, ejecutamos:
~~~
sudo nginx -t
~~~

Si no hay errores, podemos reiniciar Nginx ejecutando:
~~~
sudo nginx -s reload
~~~

### WireGuard

Puesto que nuestra red VPN (`10.0.0.0/24`) es diferente a nuestra red LAN (`192.168.1.0/24`), será necesario configurar NAT y forwarding para WireGuard, lo que permitirá comunicarnos con los demás dispositivos de nuestra red LAN de manera remota. Para ello podemos:

1. Añadir el siguiente bloque de instrucciones a la parte final del archivo de configuración de WireGuard (`/etc/wireguard/wg0.conf`):
~~~
# Habilitar tráfico de tu cliente a toda la LAN
PostUp   = iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
~~~

2. Ejecutar en el servidor WireGuard la siguiente instrucción
~~~
sudo iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
~~~

3. Verificar que el archivo `/etc/sysctl.conf` contenga activado
~~~
net.ipv4.ip_forward=1
~~~