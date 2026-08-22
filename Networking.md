# Configuración de Red para el Entorno de Desarrollo de InnovaCloud Solutions

**Autor de esta sección:** *(Leonel Oswaldo Rosales Franco)*

---

## Problematica:

Las máquinas virtuales que usa el equipo de InnovaCloud Solutions están configuradas con el modo de red que **VirtualBox** trae activado por defecto: el modo **NAT**.

Imaginemos que cada máquina virtual vive dentro de una "caja cerrada" y que la única forma que tiene de salir al mundo exterior (a Internet, por ejemplo) es pidiéndole permiso a la computadora física que la contiene (el "anfitrión"). Esa computadora actúa como una especie de intermediario o traductor que permite la salida, pero **no permite que nadie desde afuera pueda entrar a esa caja**, ni siquiera otra máquina virtual que esté en la misma computadora o en otra distinta.

Esto genera varios problemas concretos para el trabajo diario del equipo:

- **Las máquinas virtuales no pueden "verse" entre sí.** Cada una queda aislada dentro de su propia caja, por lo que si dos compañeros necesitan que sus servidores virtuales se comuniquen (por ejemplo, para probar una aplicación que depende de una base de datos en otra máquina), esto no funciona de forma natural.
- **No hay acceso desde otros equipos de la empresa.** Si alguien del equipo de soporte o un compañero desde su propia computadora necesita conectarse a una de estas máquinas virtuales para revisar un servicio, no podrá hacerlo, porque el modo NAT las mantiene "escondidas" detrás del equipo anfitrión.
- **Complica las pruebas colaborativas.** Un entorno de desarrollo en equipo necesita que las máquinas se comporten como si fueran servidores reales dentro de la red de la empresa, algo que el modo NAT no permite por diseño, ya que su propósito es justamente aislar y simplificar, no facilitar la conexión entre varios equipos.

El modo NAT es cómodo para tareas individuales y básicas, pero no es adecuado cuando varias personas necesitan que sus máquinas virtuales trabajen juntas como si fueran parte de una misma red corporativa.

---

## Los modos y sus diferencias

VirtualBox ofrece tres formas principales de conectar una máquina virtual a una red: **NAT, Puente y Red Interna**

| **NAT** (el actual) | Esconde la máquina virtual detrás del equipo anfitrión; solo permite salir a Internet, no recibir conexiones ni comunicarse con otras VMs. | Para uso individual, básico, sin necesidad de compartir nada. |
| **Adaptador Puente (Bridged)** | Conecta la máquina virtual directamente a la red física de la empresa, como si fuera una computadora más, con su propia dirección IP asignada por la red. | Cuando se necesita que la máquina sea visible y accesible desde otros equipos, servidores o compañeros de la red. |
| **Red Interna** | Crea una red privada aislada donde solo las máquinas virtuales configuradas ahí pueden hablar entre sí; no hay salida a Internet ni a la red de la empresa. | Para laboratorios de prueba totalmente aislados, sin conexión al resto de la infraestructura. |

---

## Propuesta:

Considerando que el objetivo de InnovaCloud Solutions es que varias máquinas virtuales de distintos integrantes del equipo se comuniquen entre sí y también con otros recursos de la empresa, la opción más adecuada es el **modo Puente (Bridged)**.

¿Por qué esta opción y no las otras?

- Se descarta **NAT** porque, como se explicó, aísla cada máquina y no permite ese tipo de comunicación abierta que el equipo necesita.
- Se descarta **Red Interna** porque, si bien permite que las máquinas virtuales se vean entre sí, las deja completamente aisladas del resto de la red de la empresa (sin acceso a Internet ni a otros servidores corporativos), lo cual sería demasiado restrictivo para un entorno de desarrollo colaborativo.
- El modo **Puente** es el único que logra ambas cosas a la vez: cada máquina virtual se conecta directamente a la red física de la empresa y obtiene su propia dirección IP, funcionando exactamente igual que si fuera una computadora física conectada al mismo switch o router. Esto permite que cualquier compañero, servidor o herramienta de la red pueda comunicarse con ella sin restricciones artificiales.

Al usar el modo Puente, las máquinas virtuales dejan de estar "escondidas" y pasan a formar parte de la red de la empresa como un dispositivo más.


### Configurar direcciones IP fijas (estáticas)

Una vez que las máquinas virtuales están en modo Puente, reciben una dirección IP automáticamente (de forma dinámica), igual que la mayoría de dispositivos en una red. El inconveniente es que **esa dirección puede cambiar con el tiempo**, lo cual sería un problema si, por ejemplo, un compañero necesita conectarse siempre a la misma máquina virtual para hacer pruebas: si la dirección cambia, tendría que estar preguntando cuál es la nueva cada vez.

Para evitar esto, se recomienda asignar a cada máquina virtual una **dirección IP estática**, es decir, una dirección fija que no cambie. Así, todo el equipo puede saber de antemano cómo llegar a cada servidor sin depender de configuraciones que varíen automáticamente.

En los servidores Ubuntu que usa InnovaCloud Solutions, esta configuración se realiza mediante una herramienta llamada **Netplan**, que permite definir la dirección IP fija de forma sencilla en un archivo de texto (en formato YAML), sin necesidad de complicados ajustes manuales en el sistema.

Así luce un archivo de configuración básico de Netplan con una IP fija:

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

Este archivo le indica al sistema: "esta máquina siempre debe usar la dirección 192.168.1.100 dentro de esta red, y debe salir hacia Internet a través de la puerta de enlace 192.168.1.1". Una vez guardado el archivo, basta con aplicar los cambios con un solo comando:

```bash
sudo netplan apply
```

Y para confirmar que la dirección quedó asignada correctamente, se puede verificar con:

```bash
ip addr
```

---

## 4. Ventajas de la propuesta

- **Comunicación real entre máquinas virtuales**, permitiendo pruebas colaborativas sin barreras artificiales.
- **Visibilidad hacia otros recursos corporativos**, ya que cada máquina virtual funciona como un dispositivo más dentro de la red de la empresa.
- **Estabilidad en las conexiones**, gracias al uso de direcciones IP fijas, evitando confusiones por cambios de dirección.
- **Una configuración simple de mantener**, ya que Netplan centraliza todo en un solo archivo, fácil de revisar y modificar si en el futuro la red de la empresa cambia.

