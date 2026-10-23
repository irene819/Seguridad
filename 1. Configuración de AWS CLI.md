## Gestión con AWS CLI (Línea de Comandos)

### Instalación y Configuración

Descarga de AWS CLI: [https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html)

Instala **AWS CLI v2** en tu equipo descargando el archivo ejecutable o paquete correspondiente a tu sistema operativo (Windows, macOS o Linux). Si estás en Windows, puedes usar el instalador ejecutable y abrir tu consola de comandos o **CMD** para configurar las credenciales de tu laboratorio:

```bash
aws configure

```

Copia los datos de la sección *AWS Details* de tu laboratorio dentro de tu archivo de credenciales local (en Windows: `C:\Users\TU_USUARIO\.aws\credentials` o en Linux/Mac: `~/.aws/credentials`).

> **⚠️ Importante:** Una vez guardadas las credenciales, **cierra la terminal o CMD actual y abre una nueva ventana de comandos** para asegurarte de que el sistema cargue correctamente las variables y la configuración.

---

### ⚡ Consulta de Instancias desde la Terminal

Desde tu **nueva ventana de CMD o consola**, ejecuta el siguiente comando para listar tus máquinas:

```bash
aws ec2 describe-instances --query "Reservations[*].Instances[*].{Nombre:Tags[?Key=='Name']|[0].Value,Tipo:InstanceType,IPPublica:PublicIpAddress,VPC:VpcId,Subred:SubnetId,ZonaDisponibilidad:Placement.AvailabilityZone}" --output table

```

*Nota: Puedes cambiar el parámetro final `--output table` por `--output json` o `--output text` según prefieras.*

---

> 📌 **ENTREGA DE LA PRÁCTICA**
> 
> Un PDF que muestre las capturas de la ventana de la terminal o CMD mostrando la información de tus instancias en los tres formatos solicitados: **tabla (table), JSON (json) y texto (text)**:
> 
> 1. Nombre de la instancia.
> 2. Tipo de instancia.
> 3. Zona de disponibilidad.
> 4. Red (VPC).
> 5. IP pública.
> 
> *Añade una descripción en cada captura y una breve explicación de los pasos que has realizado para configurar AWS CLI.*
```

```
