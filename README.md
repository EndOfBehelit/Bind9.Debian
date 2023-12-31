# Bind9.Debian
Configuración de Bind9 en Debian. (Las imágenes son ejemplos orientativos, puede haber pequeñas variaciones respecto al texto)

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

## Configuración del servicio en `/etc/bind/named.conf.options`

- **Creación de lista blanca de usuarios que puedan acceder al servicio**<br>
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
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/c248dae4-b24e-4d57-9f0d-7dabeb8965b4)

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
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/30c13323-d1d2-4bf2-9d2f-773c74ef4159)

- **Comprobación de errores**<br>
Podemos comprobar si hemos cometido algún error, estando en el directorio **`/etc/bind`**:
```
  sudo named-checkconf
```
- **Reinicio del servicio**
```
  sudo systemctl restart bind9
```
- **Creación de zonas directa e inversa en `/etc/bind/named.conf.local`**
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
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/a6c36320-6968-4734-9497-10cb505262cf)

- **Creación y modificación de las tablas DNS directa e inversa**<br>
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
Aquí debemos modificar la línea 12 sustituyendo `@ IN NS localhost` con `@ IN NS ns1.zonaDirecta.`.
También ahora debemos añadir todos los host que queramos:
```
  ns1 IN A 192.168.1.1
  ubuntu IN A 192.168.1.12
  gateway IN A 192.168.1.1
  dns IN A 192.168.1.1
  dhcp IN A 192.168.1.1
```
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/b863b799-80fe-4637-8a3c-752f590bd75c)<br>
Ahora debemos hacer algo muy similar pero en el archivo inverso:
```
  sudo nano -c db.1.168.192.inversa
```
Debemos modificar la línea 12 sustituyendo `**@ IN NS localhost**` con `**@ IN NS ns1.zonaDirecta.**` y comentar o borrar la línea siguiente.<br>
A continuación, añadimos los host inveros:
```
  1 IN PTR dns.zonaDirecta.
  1 IN PTR dhcp.zonaDirecta.
  1 IN PTR gateway.zonaDirecta.
  1 IN PTR ns1.zonaDirecta.
  12 IN PTR ubuntu.zonaDirecta.
```
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/98d87f98-d129-4b6c-b147-873961a2c2a5)

- **Comprobación de errores**
```
  sudo named-checkzone zonaDirecta db.zonaDirecta
```
```
  sudo named-checkzone 1.168.192.in-addr.arpa db.1.168.192.inversa
```
## Uso de bind9 a la vez que isc-dhcp-server
- **Cambios en el `/etc/dhcp/dhcpd.conf`**<br>
Ahora nuestra máquina hace tanto de DNS como de DHCP, por lo que debemos quitar los otros DNS:
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/9f1aae7c-6add-43e6-9ec4-953e33442688)
- **Cambios en `/etc/network/interfaces`**<br>
Debemos cambiar los DNS por el localhost:
![imagen](https://github.com/EndOfBehelit/Bind9.Debian/assets/154753826/aaef3ec0-d3b1-49e5-81df-84ac6b2e3c2f)

