# Práctica: Seguridad, Versionado, Ciclos de Vida y Cifrado en Amazon S3

## Módulo: Seguridad y Alta Disponibilidad 
**Objetivo:** Comprender y aplicar los mecanismos de persistencia, alta disponibilidad, retención de datos, políticas de acceso en formato JSON y seguridad criptográfica en Amazon S3.

---

## 1. Conceptos Clave de AWS S3

* **Bucket (Cubo):** Es el contenedor raíz y principal en Amazon S3 donde se guardan todos los objetos (archivos). Cada bucket tiene un nombre **único a nivel mundial**, y sobre él se aplican las políticas de seguridad en formato JSON, los permisos de acceso y las reglas de retención.
* **Zona de Disponibilidad (Availability Zone - AZ):** Es un centro de datos físico aislado y autónomo dentro de una región de AWS. Los datos en S3 se replican automáticamente entre múltiples Zonas de Disponibilidad para garantizar una disponibilidad del **99,99%** y una durabilidad extrema del **99,999999999%** (11 nueves), protegiendo la información aunque caiga un centro de datos entero.
* **Clases de Almacenamiento y Ciclos de Vida (Lifecycle Rules):** Permiten cambiar automáticamente la clase de almacenamiento de los datos según su antigüedad para optimizar costes:
  * **S3 Standard:** Para datos de acceso frecuente (alta disponibilidad, mayor coste de almacenamiento).
  * **S3 Standard-IA / Intelligent-Tiering:** Para datos de acceso infrecuente con recuperación rápida.
  * **S3 Glacier (Flexible / Deep Archive):** Diseñado para **copias de seguridad (backups) y archivo a largo plazo**. Es **mucho más económico** que el almacenamiento estándar, a cambio de mayores tiempos de recuperación (desde minutos a horas).
* **Cifrado SSE-C (Server-Side Encryption with Customer-Provided Keys):** Modelo de cifrado en el que **AWS no almacena tus claves**. El cliente gestiona la clave de 256 bits y debe proporcionarla en cada operación de subida o descarga. Si se pierde la clave, los datos son irrecuperables.

---

## EJERCICIO 1 – Alta Disponibilidad y Versionado en S3

**Objetivo:** Gestionar el historial de modificaciones, la integridad de los ficheros y la recuperación ante borrados accidentales.

### Paso 1: Crear el bucket
1. Accede a la consola de AWS S3 y haz clic en **Create bucket**.
2. Nombre del bucket: `backups-aaa` (sustituye `aaa` por tus iniciales en minúscula).
3. Configuración:
   * Acceso público bloqueado (**Block all public access** activado).
   * **Bucket Versioning = Enabled**.

### Paso 2: Crear archivo y subirlo
1. En tu equipo local, crea el fichero `DATOaaa.txt` con el siguiente contenido:
```text
Versión 1 del archivo (aaa)
```

2. Súbelo al bucket `backups-aaa`.

### Paso 3: Subir nueva versión

1. Modifica el archivo local `DATOaaa.txt` para que contenga:
```text
Versión 2 del archivo (aaa) con modificaciones de seguridad.

```
2. Súbelo nuevamente al bucket con el mismo nombre. S3 mantendrá ambas versiones almacenadas.

### Paso 4: Borrado y recuperación (Resiliencia)

1. Elimina el objeto `DATOaaa.txt` desde la consola de AWS. El bucket parecerá estar vacío.
2. Activa la casilla **Show versions** en el panel superior de S3.
3. Observa los elementos presentes:
* **Versión 1** del archivo.
* **Versión 2** del archivo.
* **Marcador de borrado (Delete Marker)**.


4. Para restaurar el fichero: selecciona el **Delete Marker** → haz clic en **Delete** → confirma la eliminación. El archivo volverá a estar visible y recuperado.

> 📸 **Entrega 1:** Captura de pantalla de la sección de versionado donde se vean las dos versiones del fichero y el *Delete Marker*.

---

## EJERCICIO 2 – Políticas de Ciclo de Vida y Retención de Backups

**Objetivo:** Automatizar la transición de archivos a clases de almacenamiento económicas para políticas de copia de seguridad.

### Paso 1: Crear la regla de ciclo de vida

1. En tu bucket `backups-aaa`, ve a la pestaña **Management** → sección **Lifecycle rules**.
2. Haz clic en **Create lifecycle rule**.
3. Configuración de la regla:
* **Name:** `Mover-a-Glacier-Backup`
* **Choose rule scope:** Selecciona *This rule applies to all objects in the bucket*.
* Marca la casilla: ✅ **Transition current versions of objects to another storage class**.
* **Storage class transitions:** Selecciona **S3 Glacier Flexible Retrieval**.
* **Days after object creation:** Indica `1` día (para simular el paso del tiempo en el entorno de laboratorio).


4. Guarda la regla con **Create rule**.

> 📸 **Entrega 2:** Captura de pantalla de la regla de ciclo de vida creada (donde se observe el nombre, la acción y la clase de almacenamiento Glacier).

---

## EJERCICIO 3 – Cifrado de Objetos con Clave de Cliente (SSE-C) y Políticas de Seguridad en JSON

**Objetivo:** Aplicar cifrado simétrico mediante AES-256 usando claves propias y reforzar el bucket con políticas JSON para bloquear subidas no cifradas.

### Paso 1: Crear bucket seguro

1. Crea un bucket llamado `confidencialaaa` (privado y con cifrado por defecto SSE-S3).

### Paso 2: Generar la clave criptográfica local

1. Abre tu terminal con la **AWS CLI** configurada.
2. Genera una clave aleatoria de 256 bits (32 bytes) y guárdala en un fichero binario:
```bash
openssl rand -out clave.bin 32

```



### Paso 3: Subida cifrada con SSE-C mediante CLI

1. Sube un archivo (ejemplo: `contrato.pdf`) obligando al uso de tu clave local:
```bash
aws s3 cp contrato.pdf s3://confidencialaaa --sse-c AES256 --sse-c-key fileb://clave.bin

```



### Paso 4: Descarga y verificación

1. Crea un directorio local llamado `descargas` e ingresa en él:
```bash
mkdir descargas && cd descargas

```


2. Descarga el archivo especificando la ruta de la clave:
```bash
aws s3 cp s3://confidencialaaa/contrato.pdf . --sse-c AES256 --sse-c-key fileb://../clave.bin

```



### Paso 5: Hardening — Política de Bucket en formato JSON

1. En la consola de AWS, ve a la pestaña **Permissions** de tu bucket `confidencialaaa` → **Bucket policy** → **Edit**.
2. Pega el siguiente documento JSON para denegar (`Deny`) cualquier subida (`s3:PutObject`) que no incluya el algoritmo SSE-C en las cabeceras HTTP:

```json
{
  "Version": "2012-10-17",
  "Id": "PutObjectPolicy",
  "Statement": [
    {
      "Sid": "RequireSSECObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::confidencialaaa/*",
      "Condition": {
        "Null": {
          "s3:x-amz-server-side-encryption-customer-algorithm": "true"
        }
      }
    }
  ]
}

```

3. **Verificación de Seguridad:** Intenta subir un fichero sin las opciones de cifrado SSE-C:
```bash
aws s3 cp prueba.txt s3://confidencialaaa

```


Comprueba que AWS responde con un error **`Access Denied`**, validando que la directiva de seguridad funciona correctamente.

---

## 🧹 Paso Obligatorio Final (Limpieza de Recursos)

Para evitar la generación de costes no deseados en el Learner Lab:

1. Elimina la regla de ciclo de vida en la pestaña **Management**.
2. Dentro del bucket, activa **Show versions**, selecciona **todos** los objetos, versiones anteriores y marcadores de borrado, y pulsa **Delete**.
3. Selecciona el bucket vacío y haz clic en **Delete bucket** confirmando con su nombre.

```

```
