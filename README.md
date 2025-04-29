# PPS-Unidad3Actividad9-AdrianCurtoSanchez

- [Configuración para deshabilitar la seguridad en PHP 8.2](#configuración-para-deshabilitar-la-seguridad-en-php-82)
- [Código vulnerable ante RFI](#securizacion-del-código)
- [Explotación de RFI](#explotación-de-rfi)
- [Mitigación de RFI](#mitigación-de-rfi)
- [Securizacion del código](#securizacion-del-código)
- [Restaurar php.ini](#restaurar-phpini)



## Configuración para deshabilitar la seguridad en PHP 8.2

Accedemos al contendor de la pila lamp con el siguiente comando:
```
docker exec -it lamp-php83 /bin/bash
```

Creamos una copia de seguridad del archi `php.ini` para ello empleamos el comando:
```
cp /usr/local/etc/php/php.ini /usr/local/etc/php/php.backup
```

Editamos el fichero y añadimos las siguientes líneas de configuración al final del fichero:
```
disable_functions =
allow_url_include = On
allow_url_fopen = On
open_basedir = 
```

Reiniciamos el contenedor de docker para que se apliquen los cambios ejecutando:
```
docker compose restart
```

## Código vulnerable ante RFI

Creamos un fichero con el siguiente código cunerable:
```
?php
// Verificar si se ha pasado un archivo por parámetro
if (isset($_GET['file'])) {
        $file = $_GET['file'];
        include($file);
}

?>
<form method="GET">
        <input type="text" name="file" placeholder="Usuario">
        <button type="submit">Subir Archivo</button>
</form>

```

## Explotación de RFI

El atacante crearía el siguiente codigo malicioso.
```
<?php
echo "¡Servidor comprometido!";
// Código malicioso, como una web shell o un backdoor
?>
```

Dicho código malicioso estaría almacenado en un servidor del atacante.

El atacante colocaría la URL del fichero malicioso en el formulario del código vulnerable para la ejecución.

Tras la subida el código malicioso se ejecuta en el servidor víctima.




## Mitigación de RFI

Para miticar el ataque RFI se deberá bloquear la inclusión de URLs externas como muestra la siguiente corrección del código vulnerable.


Si intentamos de nuevo subir el fichero remoto del ataquete vemos que no está permitido.

Esta solución no es completa, puesto que permite a subida de archivos locales maliciosos.


La siguiente aproximación, sería limitar la inclusión de archivos solo a una lista de archivos específicos dentro del servidor:
```
<?php
// Verificar si se ha pasado un archivo por parámetro
if (isset($_GET['file'])) {
        $file = $_GET['file'];
        // Lista blanca de archivos permitidos
        $whitelist = ['file1.php', 'files/file2.php'];
        if (!in_array($file, $whitelist)) {
                die("Acceso denegado.");
        }
        // Incluir solo archivos de la lista blanca
        include($file);
}

?>
<form method="GET">
        <input type="text" name="file" placeholder="Usuario">
        <button type="submit">Iniciar Sesión</button>
</form>
```

## Securizacion del código

## Restaurar php.ini
