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
- **Configuracion de las opciones**
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
