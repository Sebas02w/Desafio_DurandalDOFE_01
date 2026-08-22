# Propuesta de implementación de RAID 5 para InnovaCloud Solutions

**Autor:** Sebastian Fernando Calles Lemus

InnovaCloud Solutions está atravesando teniendo dificultades respecto al almacenamiento, su servidor principal presenta fallos de disco frecuentes y al no contar con redundancia cada avería implica la pérdida de información.

El problema de fondo es que un servidor sin respaldo se convierte en un único punto de fallo. Si el disco físico se daña, los servicios y aplicaciones alojados quedan fuera de línea por completo, lo que significa tiempos largos de inactividad. A esto se suma la pérdida de datos empresariales que no solo afecta la operación diaria, sino que pone en riesgo la integridad de la información de los clientes. Las consecuencias pueden ser graves: gastos de recuperación, daño a la reputación de la consultora y también afecta a la continuidad del negocio.

## Lo que proponemos

Para resolver esta vulnerabilidad, sugerimos montar un arreglo **RAID 5**.

## ¿Por qué RAID 5?

La tecnología RAID permite agrupar varios discos físicos para que el sistema los maneje como una sola unidad lógica. En el caso del nivel 5, combina dos cosas: reparte los datos entre los discos (lo que agiliza la lectura y escritura) y, al mismo tiempo, guarda información de paridad que permite reconstruir los datos si un disco falla. En otras palabras, se gana rendimiento sin sacrificar seguridad.

Esto responde justamente a lo que necesita InnovaCloud Solutions: si uno de los discos del arreglo deja de funcionar, la información se conserva y el servidor principal puede seguir operando sin problemas.

## Secuencia de los comandos para implementar la solución

### 1. Identificar los discos

Primero se listan las unidades de disco conectadas al servidor para identificar cuáles se usarán.

```bash
sudo fdisk -l | grep sd
```

### 2. Crear el arreglo RAID 5

Se crea el volumen agrupando 4 discos físicos (por ejemplo, `sdb`, `sdc`, `sdd` y `sde`, tomado de la guía n.º 5).

```bash
sudo mdadm --create /dev/md127 --level=raid5 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

### 3. Verificar el estado del volumen

Se puede validar el arreglo recién creado mostrando un informe.

```bash
sudo mdadm -D /dev/md127
```

### 4. Formatear el arreglo

El volumen configurado necesita un sistema de archivos para funcionar; se asigna el formato `ext4`.

```bash
sudo mkfs.ext4 /dev/md127
```

### 5. Crear el punto de montaje

Se crea un directorio específico en la ruta `/media` para poder acceder al arreglo.

```bash
sudo mkdir /media/raid5
```

### 6. Montar el RAID 5

Ahora se monta el volumen en el directorio creado.

```bash
sudo mount -t ext4 /dev/md127 /media/raid5/
```

### 7. Verificar el montaje

Se confirma que el arreglo RAID 5 sí se encuentra montado en el sistema.

```bash
mount | grep md127
```

### 8. Consultar el UUID

Para que el arreglo inicie automáticamente con cada reinicio del servidor, se debe copiar su identificador UUID.

```bash
sudo blkid /dev/md127
```

### 9. Configurar el arranque

Se ingresa al archivo de configuración de arranque de volúmenes del sistema operativo con el editor de texto `nano`.

```bash
sudo nano /etc/fstab
```

### 10. Aplicar la persistencia

Al final del archivo `/etc/fstab`, se pega el UUID (sustituyéndolo por el real), junto con la ruta y los parámetros de montaje.

```text
UUID=732fb485-eb47-464c-b213-e5a54c659040 /media/raid5 ext4 defaults,nofail 0 2
```
