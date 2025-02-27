# Firmador XML en PHP para DGII

### Requisitos

- PHP 8.0 en adelante.
- Composer.
- Instalar OpenSSL.
- Habilitar la extensión de OpenSSL en `php.ini` en caso de no tenerla activada.
- Tener activado el cifrado RC2-40-CBC en OpenSSL.

### Probar

1. Coloca el certificado `.p12` en `storage/certs`.
2. Coloca el XML a firmar en `storage/xml`.
3. Definir las variables de entorno `.env`:
   ```env
   CERT_NAME=""
   CERT_PASSWORD=""
   XML_NAME=""
   ```
4. Ejecutar el script:
   ```bash
   php src/index.php
   ```
   
## Aclaraciones

Aunque la documentación de la DGII explica cómo utilizar la librería **XMLDSIG** paraleer los certificados, y esta aplicación está basada en esa documentación, destaco que, por lo menos para mí, hay una parte de la documentación que no está del todo clara.

A continuación, muestro algunos casos:

### Caso 1: Cambiar la clase `XmlSigner.php` (o hacer una copia de la misma con los cambios de lugar)

Uno de estos casos se encuentra en la parte sobre la siguiente línea de código:

```php
$canonicalData = $element->C14N(true, false);
```

Debería cambiarse a:

```php
$canonicalData = $element->C14N(false, false);
```

Si bien la documentación explica que se debe cambiar esta línea, no menciona que también es necesario realizar el mismo cambio en otra parte del archivo:

```php
$c14nSignedInfo = $signedInfoElement->C14N(true, false);
```

Debería cambiarse a:

```php
$c14nSignedInfo = $signedInfoElement->C14N();
```

Este cambio debe realizarse específicamente en la línea 179 del archivo original.

### Caso 2: Error al leer el certificado

La documentación menciona que la librería fue probada en PHP versiones 8.1.12 y 8.1.13, debido a que el cifrado RC2-40-CBC utilizado en los archivos .p12 cambió en las versiones más recientes de OpenSSL, que normalmente vienen con PHP 8.2 en adelante. 

Al intentar leer el archivo .p12 con `openssl_pkcs12_read`, obtenemos un error porque OpenSSL dejó de admitir el cifrado RC2-40-CBC en versiones recientes debido a problemas de seguridad. Sin embargo, este cifrado aún es utilizado por la DGII en los certificados emitidos por entidades certificadas, como la Cámara de Comercio.

Para solucionarlo, debemos modificar el archivo `openssl.cnf` para que admita el cifrado que necesitamos, cambiando la configuración por defecto al modo "legacy".

#### Habilitar cifrado "legacy"

1. Edita el archivo `openssl.cnf` con el siguiente comando:
   ```bash
   sudo nano /etc/ssl/openssl.cnf
    ```
   
2. Busca la sección [default_sect] y cámbiarla a:
   ```bash
    [default_sect]
    activate = 1
    ```

3. Luego, busca la sección [legacy_sect] y cámbiarla a:
   ```bash
    [legacy_sect]
    activate = 1
    ```
   
4. Por último, busca la sección [provider_sect] y cámbiarla a:
   ```bash
    [provider_sect]
    default = default_sect
    legacy = legacy_sect
    ```
   
5.  Finalmente, guardar los cambios y salir del archivo.

## Fuentes

### Error OpenSSL

Más detalles sobre el error pueden encontrarse en el siguiente enlace de Stack Overflow:  
[Convert an old-style .p12 to .pem - Unsupported Algorithm RC2-40-CBC](https://stackoverflow.com/questions/72859711/convert-an-old-style-p12-to-pem-unsupported-algorithm-rc2-40-cbc)

### Documentación PHP DGII

Más detalles sobre la documentación de la DGII pueden encontrarse en el siguiente enlace:  
[Instructivos sobre Facturación Electrónica - Firmado de e-CF](https://dgii.gov.do/cicloContribuyente/facturacion/comprobantesFiscalesElectronicosE-CF/Documentacin%20sobre%20eCF/Instructivos%20sobre%20Facturaci%C3%B3n%20Electr%C3%B3nica/Firmado%20de%20e-CF.pdf)
