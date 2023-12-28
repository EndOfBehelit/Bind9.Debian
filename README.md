# Bind9.Debian
Configuración de Bind9 en Debian

## Descarga de Bind9 
```
  sudo apt install bind9 bind9-doc bind9utils
```
Podemos reiniciar o comprobar el servicio de bind9 en cualquier momento con los comandos:

```
  sudo systemctl restart bind9
```

```
  sudo systemctl status bind9
```

## Configuración del servicio en /etc/bind/named.conf.options

- **Creación de lista blanca de usuarios que puedan acceder al servicio**
    Esto no es algo obligatorio pero es una buena práctica.
```
  acl goodclients {
      localhost;
      192.168.1.12;
      192.168.1.13;
      192.168.1.14;
      192.168.1.66;
  };
```
- **Configuración de las opciones**
```
  options {
      directory "/var/cache/bind";
      recursion yes;
      allow-query {  goodclients; }
      forwarders {
          IPS_DNS;
      }
      forward only;
      dnssec-validation no;
      listen-on-v6 { any; };
  };
```
- **Comprobación de errores**
Podemos comprobar si hemos cometido algún error, estando en el directorio **/etc/bind**:
```
  sudo named-checkconf
```
- **Reinicio del servicio**
```
  sudo systemctl restart bind9
```
- **Creación de zonas directa e inversa en /etc/bind/named.conf.local**
```
  zone "zonaDirecta" {
      type master;
      file "/etc/bind/db.directa";
  };
  zone "1.168.192.in-addr.arpa" {
      type master;
      notify no;
      file "/etc/bind/db.1.168.192.inversa"
  }
```
- **Creación y modificación de las tablas DNS directa e inversa**
Podemos copiar los archivos por defecto para usarlos como plantilla.
```
  sudo cp db.local db.directa
```
```
  sudo cp db.local db..1.168.192.inversa
```
Ahora modificaremos la plantilla directa:
```
  sudo nano -c db.directa
```
Aquí debemos modificar la línea 12 sustituyendo **@ IN NS localhost** con **@ IN NS ns1.zonaDirecta.**.
También ahora debemos añadir todos los host que queramos:
```
  ns1 IN A 192.168.1.1
  ubuntu IN A 192.168.1.12
  gateway IN A 192.168.1.1
  dns IN A 192.168.1.1
  dhcp IN A 192.168.1.1
```
