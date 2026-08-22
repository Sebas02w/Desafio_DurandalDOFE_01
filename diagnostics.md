# Diagnóstico y Verificación de Red

**Autor:** [Mauricio Eddye Lozano Pérez]


## Problema identificado
InnovaCloud Solutions no cuenta con un procedimiento estandarizado para verificar el estado de sus interfaces de red,
la conectividad, los puertos abiertos y los servicios activos en sus servidores Linux. 
Esta situación dificulta la identificación del origen de las fallas y puede aumentar el tiempo necesario para recuperar los servicios afectados.

La ausencia de una metodología común también puede provocar diagnósticos incompletos.
Por ejemplo, un servidor puede tener una dirección IP correctamente configurada, pero presentar un problema en su tabla de rutas,
una interfaz inactiva, un servicio detenido o un puerto que no sea accesible desde otros equipos.

Por lo tanto, se requiere establecer un procedimiento que permita comprobar de manera ordenada el funcionamiento de los principales
componentes relacionados con la conectividad y los servicios de red.


## Propuesta de solución

Se propone implementar un **Procedimiento estandarizado de diagnóstico y verificación de red para todos los servidores Linux de InnovaCloud Solutions**.
Este procedimiento permitirá comprobar de forma sistemática las interfaces de red, el direccionamiento IP,
las rutas, la conectividad, los puertos abiertos y los servicios en ejecución.

El diagnóstico se realizará de manera progresiva, comenzando por la configuración local del servidor y 
avanzando hacia la comunicación con otros equipos de la red. De esta forma, cuando se presente una falla, 
el administrador podrá determinar en qué punto se encuentra el problema en lugar de realizar comprobaciones aisladas.


### 1. Verificación de interfaces y direccionamiento

El primer paso consiste en identificar las interfaces de red disponibles y comprobar que se encuentren activas y correctamente configuradas.

```bash
ip -br addr
ip link
```

Estos comandos permiten comprobar el nombre y estado de las interfaces, así como las direcciones IPv4 e IPv6 asignadas.
Esta revisión es importante porque una interfaz deshabilitada o una dirección IP incorrecta puede impedir completamente la comunicación del servidor.

También se debe verificar la tabla de enrutamiento:

```bash
ip route
```

Con este comando se comprueba que el servidor conozca las redes necesarias y que exista una puerta de enlace
válida cuando sea necesaria para comunicarse con otras redes.


### 2. Verificación de conectividad

Una vez validada la configuración local, se debe comprobar la conectividad de forma progresiva. 
Para ello se utilizará `ping`, herramienta que permite verificar si un destino es accesible y observar la pérdida de paquetes y la latencia.

Por ejemplo:

```bash
ping -c 4 127.0.0.1
ping -c 4 <IP_GATEWAY>
ping -c 4 <IP_SERVIDOR_DESTINO>
```

Las pruebas deben realizarse desde el propio servidor hacia la interfaz local, 
posteriormente hacia la puerta de enlace y finalmente hacia otros servidores o destinos externos.

Si existe un problema de conectividad, se puede utilizar `tracepath` para analizar la ruta que siguen los paquetes:

```bash
tracepath <IP_DESTINO>
```

Esto permite identificar posibles problemas en los saltos intermedios o en el enrutamiento de la red.


### 3. Verificación de puertos y servicios

Después de comprobar la conectividad, se debe verificar que los servicios requeridos por la infraestructura se encuentren disponibles.

Para identificar los puertos que están actualmente en escucha en el servidor se utilizará:

```bash
sudo ss -tulpn
```

Este comando permite observar los puertos TCP y UDP utilizados por los procesos del sistema.
También se puede utilizar `nmap` desde otro equipo autorizado para comprobar qué puertos son accesibles desde la red:

```bash
nmap <IP_SERVIDOR>
```

De esta manera se puede comparar lo que el servidor tiene configurado localmente con lo que realmente puede ser alcanzado desde otro equipo.
Posteriormente, se deben revisar los servicios que se encuentran en ejecución mediante:

```bash
systemctl --type=service --state=running
```

Para analizar un servicio específico:

```bash
sudo systemctl status <servicio>
```

Si se detecta un servicio detenido o con errores, se pueden consultar sus registros mediante:

```bash
sudo journalctl -u <servicio> -n 50
```

Los registros permiten obtener información adicional sobre errores de configuración, 
fallos durante el inicio o problemas ocurridos durante la ejecución del servicio.

### Procedimiento general

El procedimiento propuesto para los servidores de InnovaCloud Solutions queda establecido de la siguiente manera:

 ```text
Identificar interfaces
  ↓
 Verificar direcciones IP
    ↓
 Revisar tabla de rutas
      ↓
 Comprobar conectividad
        ↓
 Analizar la ruta si existe una falla
          ↓
 Verificar puertos en escucha
            ↓
 Comprobar puertos desde la red
              ↓
 Auditar servicios activos
                ↓
 Revisar registros en caso de errores
 ```

La aplicación de este procedimiento permitirá que los administradores sigan una misma metodología ante los problemas de red,
reduciendo la posibilidad de omitir alguna comprobación importante.


### Herramientas principales

| **Herramienta** | **Función**                                                           |
| :-------------- | :---------------------------------------------------------------------|
|  `ip`         | Identificación de interfaces, direcciones IP, rutas y estado de la red. |
|  `ping`       | Verificación de conectividad y detección de pérdida de paquetes.        |
|  `tracepath`  | Análisis de la ruta de los paquetes hacia un destino.                   |
|  `ss`         | Consulta de puertos, conexiones activas y servicios en escucha.         |
|  `nmap`       | Análisis de puertos y servicios accesibles desde la red.                |
|  `systemctl`  | Consulta y administración del estado de los servicios del sistema.      |
|  `journalctl` | Revisión de registros, errores y eventos del sistema.                   |



## Beneficios de la solución

La estandarización del diagnóstico permitirá a InnovaCloud Solutions:

* Reducir el tiempo necesario para localizar fallas.
* Detectar interfaces o configuraciones de red incorrectas.
* Verificar que las rutas y puertas de enlace funcionen correctamente.
* Comprobar la conectividad entre los diferentes servidores.
* Identificar puertos abiertos y servicios accesibles.
* Detectar servicios detenidos o que presenten errores.
* Utilizar los registros del sistema para determinar las causas de las fallas.
* Mantener un procedimiento uniforme para la administración de todos los servidores Linux.

## Conclusión

La implementación de un procedimiento estandarizado de diagnóstico permitirá a InnovaCloud Solutions realizar una verificación ordenada
y consistente de su infraestructura de red. La combinación de las herramientas `ip`, `ping`,
`tracepath`, `ss`, `nmap`, `systemctl` y `journalctl` permite revisar desde el estado de las interfaces y el direccionamiento hasta la conectividad,
los puertos y los servicios que proporcionan funcionalidades a la empresa.

De esta manera, ante una falla, el administrador podrá seguir una secuencia definida para identificar
el punto donde se encuentra el problema y obtener información suficiente para aplicar las medidas correctivas correspondientes.
