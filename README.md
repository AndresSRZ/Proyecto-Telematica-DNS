# Proyecto-Telematica-DNS
Realizado por: Andres Suarez, Samuel Alarcon y Jose Manuel Carvajal

# Proyecto Final – Servidor DNS con BIND en AWS

## Descripción General

Este proyecto consiste en la implementación de un servidor DNS usando BIND9 en una instancia de Ubuntu dentro de AWS. El objetivo es permitir que diferentes departamentos (Mercadeo, Gerencia y Finanzas), representados por máquinas virtuales en la misma red, puedan resolver nombres de dominio internos definidos por el administrador, en lugar de usar direcciones IP directamente.

---

## Objetivos

- Instalar y configurar un servidor DNS con BIND9.
- Crear zonas directa e inversa para resolución de nombres internos.
- Configurar múltiples clientes para usar el servidor DNS.
- Validar el correcto funcionamiento del sistema mediante comandos como `ping` y `dig`.

---

## Arquitectura del sistema

Se utilizaron 4 instancias EC2 (Ubuntu 22.04) en la VPC predeterminada de AWS:

| Rol        | Hostname              | IP Privada         |
|------------|------------------------|--------------------|
| DNS Server | dns-server-Ubuntu      | 172.31.22.147      |
| Cliente 1  | Mercadeo               | 172.31.29.125      |
| Cliente 2  | Gerencia               | 172.31.18.76       |
| Cliente 3  | Finanzas               | 172.31.29.159      |

---

## Pasos de configuración

### En el servidor DNS (`dns-server-Ubuntu`)

- Instalar BIND:
  - sudo apt update
  - sudo apt install bind9 dnsutils -y
  - sudo apt update

- Configurar /etc/bind/named.conf.options para aceptar consultas locales.
  
- Crear zonas en /etc/bind/named.conf.local:
  - departamentos.local (zona directa)
  - 31.172.in-addr.arpa (zona inversa)

- Crear los archivos de zona en /etc/bind/zones/.

- Reiniciar BIND y verificar:
  -sudo systemctl restart bind9
  -sudo systemctl enable bind9

### En cada cliente (Mercadeo, Gerencia, Finanzas)
- Remover y reemplazar el archivo /etc/resolv.conf:
  - sudo rm /etc/resolv.conf
  - sudo nano /etc/resolv.conf

- Contenido del archivo:
  - nameserver 172.31.22.147

- (Opcional) Deshabilitar systemd-resolved:
  - sudo systemctl disable systemd-resolved
  - sudo systemctl stop systemd-resolved

- Ajustar /etc/hosts para evitar errores con sudo:
  - 127.0.1.1 ip-172-31-29-125     # para Mercadeo
  - 127.0.1.1 ip-172-31-18-76      # para Gerencia
  - 127.0.1.1 ip-172-31-29-159     # para Finanzas

## Pruebas realizadas

Resolución directa (desde cualquier cliente):
- ping mercadeo.departamentos.local
- ping gerencia.departamentos.local
- ping finanzas.departamentos.local
- ping dns.departamentos.local

Resolución con dig:
- dig mercadeo.departamentos.local
- dig -x 172.31.29.125
  
Todos los nombres fueron correctamente resueltos a sus respectivas direcciones IP.

## Retos enfrentados
- Problemas de conexión en Amazon Linux (resueltos cambiando a Ubuntu).
- Falta de acceso a internet por configuración de tabla de rutas.
- Conflictos con systemd-resolved al modificar /etc/resolv.conf.

## Archivos incluidos en este repositorio
- named.conf.options
- named.conf.local
- departamentos.local.zone
- 31.172.in-addr.arpa.zone
