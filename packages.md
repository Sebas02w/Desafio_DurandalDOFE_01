# Gestión de Paquetes
**Autor:** Josué Daniel Menjivar Batres
## Problema identificado

InnovaCloud Solutions realiza la instalación de paquetes de software de forma manual en sus diferentes equipos. Esta forma de trabajo puede provocar inconsistencias, ya que algunos equipos podrían instalar diferentes versiones de un mismo programa o realizar actualizaciones en momentos distintos.

Además, si cada equipo descarga los mismos paquetes directamente desde Internet, se generan descargas repetidas que aumentan el consumo del ancho de banda de la empresa.

## Solución propuesta: Repositorio espejo local

Se propone implementar un **repositorio espejo local** dentro de la infraestructura de InnovaCloud Solutions.

El repositorio local funcionará como un punto central para la gestión de paquetes. En lugar de que cada equipo consulte y descargue individualmente los paquetes desde repositorios externos, los equipos podrán utilizar el servidor local como fuente principal.

El funcionamiento general de la solución sería el siguiente:

El servidor Mirror Local InnovaCloud Solutions estaria sincronizado con el repositorio oficial de Ubuntu y el servidor mirror alimentaria con la gestion de paquetes a la red interna de la empresa

Para crear y mantener una copia local de los repositorios se puede utilizar la herramienta `apt-mirror`.

### Instalación de la herramienta

En el servidor destinado a funcionar como repositorio local se ejecutan los siguientes comandos:

```bash
sudo apt update
sudo apt install apt-mirror
```

Con esto se actualiza la información de los paquetes disponibles y se instala la herramienta necesaria para crear el repositorio espejo.

### Configuración del repositorio

La configuración de `apt-mirror` se realiza mediante el siguiente archivo:

```bash
sudo nano /etc/apt/mirror.list
```

En este archivo se define el repositorio que será sincronizado.

De esta manera, la empresa puede seleccionar los repositorios necesarios para su infraestructura, evitando descargar contenido que no sea requerido.

### Sincronización del repositorio

Una vez configurado el repositorio, se ejecuta:

```bash
sudo apt-mirror
```

Este proceso descarga y organiza una copia local de los paquetes seleccionados.

Posteriormente, los equipos de la empresa pueden configurarse para utilizar el servidor local como fuente de paquetes.

Después de configurar la nueva fuente, el cliente actualiza la información de los paquetes mediante:

```bash
sudo apt update
```

A partir de ese momento, las instalaciones de software pueden consultar el repositorio local en lugar de realizar las mismas descargas repetidamente desde Internet.

## Beneficios de la solución

La implementación de un repositorio espejo local proporciona los siguientes beneficios:

* **Mayor eficiencia:** los paquetes pueden distribuirse desde un punto central dentro de la red de la empresa.
* **Mayor consistencia:** los equipos utilizan una misma fuente de paquetes, reduciendo las diferencias entre versiones instaladas.
* **Optimización del ancho de banda:** se reducen las descargas repetidas desde Internet, ya que los paquetes se sincronizan y distribuyen desde el servidor local.
* **Administración centralizada:** la empresa dispone de un punto de control para la gestión y disponibilidad de los paquetes utilizados en su infraestructura.
