# InnovaCloud Solutions

### Propuesta integral de infraestructura y administración de servidores Linux

Repositorio correspondiente a la propuesta de solución tecnológica desarrollada para **InnovaCloud Solutions**, orientada a mejorar la disponibilidad, conectividad, administración y mantenimiento de su infraestructura de servidores Linux.

---

## 1. Descripción del proyecto

InnovaCloud Solutions presenta diferentes necesidades relacionadas con la administración de su infraestructura tecnológica. Entre los principales problemas identificados se encuentran la falta de comunicación adecuada entre máquinas virtuales, la ausencia de procedimientos estandarizados para el diagnóstico de red, la gestión descentralizada de paquetes y la falta de redundancia en el almacenamiento del servidor principal.

Como respuesta a estas necesidades, se propone una solución integral compuesta por cuatro áreas principales:

* **Configuración de red:** implementación del modo **Adaptador Puente (Bridged)** en VirtualBox y configuración de direcciones IP estáticas mediante **Netplan**.
* **Diagnóstico y verificación:** establecimiento de un procedimiento estandarizado para comprobar interfaces, direccionamiento, conectividad, puertos y servicios.
* **Gestión de paquetes:** implementación de un **repositorio espejo local** para centralizar y optimizar la distribución de paquetes en la red.
* **Almacenamiento:** implementación de un arreglo **RAID 5** para proporcionar tolerancia ante la falla de uno de los discos del servidor.

Estas soluciones buscan proporcionar una infraestructura más estable, administrable y preparada para las actividades de desarrollo y operación de InnovaCloud Solutions.

---

## 2. Solución integral propuesta

### 2.1 Configuración de red

Se propone reemplazar el modo **NAT** utilizado actualmente por el **Adaptador Puente (Bridged)** de VirtualBox.

A diferencia de NAT, el modo Puente permite que cada máquina virtual se incorpore directamente a la red física como un dispositivo independiente. Esto facilita la comunicación entre servidores y equipos de la empresa.

Además, en los servidores Ubuntu se propone utilizar **Netplan** para establecer direcciones IP estáticas, evitando que las direcciones cambien y afecten las conexiones entre los diferentes servicios.

Ejemplo de configuración:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Aplicación de la configuración:

```bash
sudo netplan apply
```

Verificación:

```bash
ip addr
```

**Documentación completa:** [Networking.md](Networking.md)

---

### 2.2 Diagnóstico y verificación de red

Para evitar diagnósticos aislados o incompletos, se propone establecer un procedimiento común para todos los servidores Linux.

El procedimiento parte de la configuración local y avanza progresivamente hacia la conectividad y los servicios:

```text
Interfaces
    ↓
Direcciones IP
    ↓
Tabla de rutas
    ↓
Conectividad
    ↓
Ruta hacia el destino
    ↓
Puertos en escucha
    ↓
Puertos accesibles desde la red
    ↓
Servicios activos
    ↓
Registros del sistema
```

Entre las principales herramientas utilizadas se encuentran:

| **Herramienta** | **Función**                                                     |
| :-------------- | :-------------------------------------------------------------- |
| `ip`            | Identificación de interfaces, direcciones IP y rutas.           |
| `ping`          | Verificación de conectividad y pérdida de paquetes.             |
| `tracepath`     | Análisis de la ruta de los paquetes hacia un destino.           |
| `ss`            | Consulta de puertos, conexiones activas y servicios en escucha. |
| `nmap`          | Análisis de puertos y servicios accesibles desde la red.        |
| `systemctl`     | Consulta y administración del estado de los servicios.          |
| `journalctl`    | Revisión de registros y errores del sistema.                    |

Ejemplos:

```bash
ip -br addr
ip route
ping -c 4 <IP_DESTINO>
tracepath <IP_DESTINO>
sudo ss -tulpn
nmap <IP_SERVIDOR>
systemctl --type=service --state=running
sudo journalctl -u <servicio> -n 50
```

**Documentación completa:** [diagnostics.md](diagnostics.md)

---

### 2.3 Repositorio espejo local

Para reducir descargas repetitivas y mejorar la consistencia de los paquetes instalados, se propone implementar un **repositorio espejo local**.

El servidor local se sincronizará con los repositorios oficiales de Ubuntu y posteriormente proporcionará los paquetes a los equipos de la red interna.

La herramienta propuesta para realizar la sincronización es `apt-mirror`.

Instalación:

```bash
sudo apt update
sudo apt install apt-mirror
```

Configuración:

```bash
sudo nano /etc/apt/mirror.list
```

Sincronización:

```bash
sudo apt-mirror
```

Los equipos clientes podrán utilizar posteriormente el repositorio local como fuente de paquetes, reduciendo las descargas repetidas desde Internet y facilitando una administración más centralizada.

**Documentación completa:** [packages.md](packages.md)

---

### 2.4 Almacenamiento con RAID 5

Debido a las fallas frecuentes de disco identificadas en el servidor principal, se propone implementar un arreglo **RAID 5** utilizando cuatro discos.

RAID 5 distribuye los datos entre los discos y utiliza información de paridad para permitir la recuperación de la información cuando uno de ellos falla. Esto proporciona tolerancia ante una falla de disco y permite mantener la disponibilidad del almacenamiento mientras se reemplaza la unidad afectada.

Ejemplo de creación del arreglo:

```bash
sudo mdadm --create /dev/md127 \
  --level=raid5 \
  --raid-devices=4 \
  /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

Verificación:

```bash
sudo mdadm -D /dev/md127
```

Formateo:

```bash
sudo mkfs.ext4 /dev/md127
```

Montaje:

```bash
sudo mkdir /media/raid5
sudo mount -t ext4 /dev/md127 /media/raid5/
```

Para mantener el montaje después de reiniciar el servidor, se configura el UUID correspondiente en `/etc/fstab`.

> **Nota:** RAID 5 proporciona redundancia y tolerancia a fallos, pero **no sustituye una estrategia de copias de seguridad**. Se recomienda mantener respaldos independientes de la información crítica.

**Documentación completa:** [storage.md](storage.md)

---

## 3. Documentación del proyecto

| **Documento**                    | **Contenido**                                       |
| :------------------------------- | :-------------------------------------------------- |
| [Networking.md](Networking.md)   | Configuración de red, modo Puente y Netplan.        |
| [diagnostics.md](diagnostics.md) | Procedimiento de diagnóstico y verificación de red. |
| [packages.md](packages.md)       | Implementación del repositorio espejo local.        |
| [storage.md](storage.md)         | Propuesta e implementación de RAID 5.               |

---

## 4. Integrantes del equipo

| **Integrante**                      | **Rol / Área de trabajo**                      |
| :---------------------------------- | :--------------------------------------------- |
| **Sebastian Fernando Calles Lemus** | Almacenamiento y configuración de RAID 5       |
| **Leonel Oswaldo Rosales Franco**   | Configuración de red y Netplan                 |
| **Mauricio Eddye Lozano Pérez**     | Diagnóstico y verificación de red              |
| **Josué Daniel Menjivar Batres**    | Gestión de paquetes y repositorio espejo local |

---

## 5. Tecnologías y herramientas

La propuesta utiliza las siguientes tecnologías y herramientas:

* **VirtualBox** — Virtualización de los servidores.
* **Ubuntu Server** — Sistema operativo de los servidores Linux.
* **Netplan** — Configuración de red en Ubuntu.
* **apt-mirror** — Creación y sincronización del repositorio espejo.
* **mdadm** — Administración del arreglo RAID.
* **ext4** — Sistema de archivos utilizado para el almacenamiento.
* **ip, ping y tracepath** — Diagnóstico de conectividad y enrutamiento.
* **ss y nmap** — Verificación de puertos y servicios.
* **systemctl y journalctl** — Administración y análisis de servicios del sistema.

---

## 6. Beneficios generales

La implementación conjunta de las soluciones propuestas permitirá a InnovaCloud Solutions:

* Mejorar la comunicación entre sus máquinas virtuales.
* Facilitar el acceso a los servidores desde otros equipos autorizados.
* Mantener direcciones IP estables para los servicios.
* Estandarizar el diagnóstico de problemas de red.
* Reducir el tiempo necesario para localizar fallas.
* Centralizar la distribución de paquetes de software.
* Reducir descargas repetitivas desde Internet.
* Mejorar la consistencia de las versiones instaladas.
* Proporcionar tolerancia ante la falla de un disco.
* Facilitar la administración y mantenimiento de la infraestructura.

En conjunto, estas medidas permiten construir una infraestructura más **estable, administrable, disponible y preparada para el crecimiento** de InnovaCloud Solutions.

---

## 7. Enlaces de entrega

**Repositorio de GitHub:**
[Desafío DurandalDOFE 01 - InnovaCloud Solutions](https://github.com/Sebas02w/Desafio_DurandalDOFE_01)

**Video de defensa:**
[Enlace al video de la defensa](#)


---

## 8. Conclusión

La solución propuesta aborda de manera integral los principales problemas identificados en la infraestructura de InnovaCloud Solutions.

La combinación de una red basada en **Adaptador Puente con direccionamiento estático mediante Netplan**, un **procedimiento estandarizado de diagnóstico**, un **repositorio espejo local** y un sistema de almacenamiento basado en **RAID 5** permite mejorar la comunicación, administración, disponibilidad y tolerancia a fallos de los servidores.

Cada componente cuenta con documentación técnica, ejemplos de comandos y procedimientos de configuración que permiten reproducir y verificar la solución dentro del entorno de desarrollo.

---

### InnovaCloud Solutions

**Propuesta de infraestructura | Administración de servidores Linux | Virtualización | Redes | Almacenamiento**
