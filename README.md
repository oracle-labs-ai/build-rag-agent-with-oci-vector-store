# Agente RAG con LangChain y OCI Vector Store

Este proyecto contiene un laboratorio práctico para construir un agente RAG con LangChain y OCI Generative AI.

El agente trabaja con documentos Markdown almacenados en OCI Vector Store. Permite cargar e indexar documentos, realizar búsquedas semánticas y usar los fragmentos encontrados para responder preguntas mediante una herramienta de LangChain.

El laboratorio también explica cómo configurar las credenciales locales de OCI e incluye una sección para crear y cargar un vector store. El notebook reúne las explicaciones y el código ejecutable.

Consulta el laboratorio completo en [notebook.ipynb](notebook.ipynb).

## Primeros pasos antes de abrir el notebook

Esta guía te ayuda a preparar el entorno para ejecutar el laboratorio con Visual Studio Code y configurar los recursos necesarios en la consola web de OCI.

Al terminar tendrás:

- Python 3.14.x o superior y Visual Studio Code con las extensiones necesarias.
- El proyecto descargado y abierto en VS Code, con el notebook y la carpeta `data/` preparados.
- Un archivo `.env` configurado para el laboratorio.
- Un perfil local de OCI configurado para autenticarse con una API key.
- Un proyecto y un vector store de OCI Generative AI.
- Un entorno virtual `.venv` con las dependencias instaladas.

> [!NOTE]
> Si tienes inconvenientes durante la preparación o ejecución del laboratorio, consulta la sección [Problemas frecuentes](#problemas-frecuentes).

### 1. Instalar los programas

1. Si aún no tienes Python instalado, descarga e instala [Python 3.14.2](https://www.python.org/downloads/release/python-3142/). También puedes usar una versión superior compatible con las dependencias del laboratorio.
2. Si aún no tienes Visual Studio Code, descárgalo e instálalo desde [code.visualstudio.com](https://code.visualstudio.com/Download).
3. En Visual Studio Code, abre el menú de los tres puntos `(...)` de la parte superior y selecciona **Terminal > New Terminal**. En la terminal, escribe `py --version` y pulsa **Enter**. Debería aparecer la versión de Python instalada, por ejemplo: `Python 3.14.2`.

### 2. Instalar las extensiones de VS Code

Si aún no tienes instaladas las extensiones **Python** y **Jupyter**, ambas publicadas por Microsoft, abre **Extensiones** desde el menú lateral, busca cada una por su nombre e instálalas.

<p align="center">
  <img alt="Estructura del proyecto en VS Code" src="https://github.com/user-attachments/assets/5c15274a-90a0-42f0-b0d7-079eeb4d0c56">
</p>
<p align="center">
  <img alt="Estructura del proyecto en VS Code" src="https://github.com/user-attachments/assets/b76ab27e-b6f5-4177-8622-49c6534cb657">
</p>

### 3. Descargar, abrir y empezar a configurar el proyecto

1. En GitHub, abre el repositorio y selecciona **Code > Download ZIP**.
2. Descomprime el archivo en una ubicación fácil de encontrar, por ejemplo, **Descargas**.
3. En VS Code, selecciona **File > Open Folder...** en la barra superior.
4. Busca la carpeta descomprimida del proyecto y selecciónala. Debes abrir la carpeta raíz, donde están `data/`, `notebook.ipynb`, `requirements.txt`, `.env.template` y `README.md`.
5. Pulsa **Select Folder**. Si VS Code pregunta si confías en los autores, selecciona **Yes, I trust the authors**.
6. En el panel **Explorer**, la estructura debería verse así:
   
<p align="center">
  <img alt="Estructura del proyecto en VS Code" src="https://github.com/user-attachments/assets/047811d2-04f5-4e46-b8e8-ebd626d95e32">
</p>

7. En la carpeta `data/` encontrarás el documento de prueba `lumina_credito_global.md`. Para el laboratorio, te recomendamos agregar otro archivo `.md` sobre un tema de tu interés y probar el agente con ese contenido.
   
> [!NOTE]
> Puedes pedirle a una IA que genere un archivo `.md` sobre un tema de tu interés y guardarlo en `data/`. No incluyas información personal ni confidencial.

8. Abre `.env.template`, copia su contenido y pégalo en un archivo nuevo llamado `.env` en la carpeta raíz del proyecto.
9. Abre el archivo `.env` y escribe en `OCI_SOURCE_DOCUMENT` el nombre del archivo `.md` que agregaste a `data/`. Por ejemplo, si lo llamaste `f1.md`, la línea debe quedar así:

   ```dotenv
   OCI_SOURCE_DOCUMENT=f1.md

> [!WARNING]
> Mantén `OCI_UPLOAD_DOCUMENT=true` para cargar el documento por primera vez. Cuando la carga termine correctamente, cámbialo a `false` para evitar cargarlo de nuevo.

### 4. Preparar OCI en la consola web

1. Ve a [Oracle Cloud](https://www.oracle.com/cloud/sign-in.html) e inicia sesión.
2. Selecciona el **tenancy** correspondiente, el dominio **Default** e ingresa tus credenciales.
3. Una vez dentro de la consola, continúa con los siguientes pasos para configurar el compartment, la API key, el proyecto y el vector store.

#### 4.1 Verificar la región

Verifica que la región seleccionada en la parte superior derecha sea **US East (Ashburn)**. Esta misma región se debe usar para el proyecto y el vector store.

#### 4.2 Copiar el OCID del compartment

1. Abre el menú principal (☰) y ve a **Identity & Security > Compartments**. Si no encuentras la opción, búscala escribiendo `Compartments` en el buscador de la consola.
2. En la fila del compartment destinado para el laboratorio, abre el menú de los tres puntos (`...`) y selecciona **Copy OCID**.
3. Abre el archivo `.env` y pega el valor copiado en `OCI_COMPARTMENT_ID`. La línea debe quedar así:

   ```dotenv
   OCI_COMPARTMENT_ID=ocid1.compartment.oc1..aaaa...

#### 4.3 Copiar el OCID del proyecto de Generative AI

1. Abre el menú principal (☰) y ve a **Analytics & AI > Generative AI > Projects**. Si no encuentras la opción, búscala escribiendo `Projects` en el buscador de la consola.
2. En el filtro **Compartment**, selecciona el compartment destinado para el laboratorio.
3. En la fila del proyecto destinado para el laboratorio, abre el menú de los tres puntos (`...`) y selecciona **Copy OCID**.
4. Abre el archivo `.env` y pega el valor copiado en `OCI_GENERATIVE_AI_PROJECT_ID`. La línea debe quedar con este formato:

   ```dotenv
   OCI_GENERATIVE_AI_PROJECT_ID=ocid1.generativeaiproject.oc1.iad.aaaa...

#### 4.4 Crear el vector store

1. Abre el menú principal (☰) y ve a **Analytics & AI > Generative AI > Vector stores**. Si no encuentras la opción, búscala escribiendo `Vector stores` en el buscador de la consola.
2. En el filtro **Compartment**, selecciona el compartment destinado para el laboratorio.
3. Haz clic en **Create vector store** y escribe un nombre fácil de identificar, por ejemplo, `vs-<tu-nombre>`.
4. En **Data source type**, selecciona **Unstructured**, ya que el laboratorio usa documentos.
5. Opcionalmente, configura una **Expiration date** para definir cuándo expirará el vector store.
6. Haz clic en **Create**.
7. En la fila del vector store que acabas de crear, abre el menú de los tres puntos (`...`) y selecciona **View details**. En details, haz clic en **Copy** junto al Vector store ID.
8.Abre el archivo `.env` y pega el valor en `OCI_VECTOR_STORE_ID`. La línea debe quedar con este formato:

   ```dotenv
   OCI_VECTOR_STORE_ID=vs_ord_...

#### 4.5 Crear la API key y el archivo de configuración de OCI

1. Haz clic en el icono de perfil, en la parte superior derecha, y selecciona tu correo.
2. Ve a **Tokens and keys > API keys** y haz clic en **Add API key**.
3. Selecciona **Generate API key pair** y haz clic en **Download private key** para descargar el archivo `.pem`.
4. Haz clic en **Add**.
5. En el Explorador de archivos, ve a tu carpeta de usuario y crea una carpeta llamada `.oci`. Mueve allí la llave privada descargada y renómbrala como `oci_api_key.pem`.
6. Vuelve a la consola de OCI. En la ventana **Configuration file preview**, haz clic en **Copy**.
7. En VS Code, selecciona **File > New Text File** y pega el contenido copiado. Actualiza el valor de `key_file` para que apunte a la ubicación donde guardaste `oci_api_key.pem`. Debería verse así:

   ```ini
    [DEFAULT]
    user=ocid1.user.oc1..aaaa...
    fingerprint=12:34:56:78:90:ab:cd:ef:...
    tenancy=ocid1.tenancy.oc1..aaaa...
    region=us-ashburn-1
    key_file=C:\Users\pepito\.oci\oci_api_key.pem
   
8. Guarda el archivo con Ctrl + S. Selecciona la carpeta .oci creada en tu carpeta de usuario, escribe config como nombre del archivo y elige No Extension en el campo de tipo. Luego haz clic en Save.

### 5. Crear el ambiente virtual desde VS Code

1. Pulsa `Ctrl + Shift + P` para abrir la paleta de comandos.
2. Escribe **Python: Create Environment** y selecciónalo.
3. Selecciona **Quick Create**. VS Code creará el ambiente virtual con Python e instalará las dependencias de `requirements.txt`.
4. Espera a que termine la instalación. Verás el progreso en la esquina inferior derecha y se creará la carpeta `.venv` en el proyecto.
5. Si VS Code pregunta si quieres usar el nuevo ambiente para el workspace, responde que sí. Al terminar, .venv quedará seleccionado como el entorno de Python del proyecto.

### 6. Verificar la autenticación con OCI

Antes de abrir el notebook, comprueba que tu perfil y el compartment funcionan:

1. Crea un archivo nuevo llamado `test_oci.py` y pega este código.

```python
# %%
import os

import oci
from dotenv import load_dotenv

load_dotenv()

profile = os.getenv("OCI_PROFILE", "DEFAULT")
config = oci.config.from_file(profile_name=profile)
identity = oci.identity.IdentityClient(config)

compartment_id = os.environ["OCI_COMPARTMENT_ID"]
compartment = identity.get_compartment(compartment_id)

print("OK:", compartment.data.name)
```
2. Sobre `# %%` aparece el enlace **Run Cell**. Púlsalo.
3. Si VS Code pide seleccionar un kernel, elige el ambiente `.venv`.
4. Se abrirá una ventana interactiva con el resultado. Si imprime `OK:` con el nombre del compartment, la autenticación está lista.

Puedes borrar `test_oci.py` cuando termines.

### 7. Abrir el notebook y seleccionar el kernel

1. En el **Explorer**, abre `notebook.ipynb`.
2. Arriba a la derecha pulsa **Select Kernel > Python Environments...** y elige el ambiente `.venv`. Si VS Code ofrece instalar `ipykernel`, acepta.
3. Ejecuta las celdas **en orden**, una por una, con `Shift + Enter` o con el botón de ejecución a la izquierda de cada celda. El número entre corchetes junto a la celda indica que se ejecutó; `[*]` significa que sigue en ejecución.
4. Si cambias `.env`, usa el botón **Restart** de la barra del notebook para volver a leer las variables, y ejecuta de nuevo desde la primera celda.
5. Ejecuta la celda de carga del documento **una sola vez**. Debe terminar con `Status: completed`. Después cambia `OCI_UPLOAD_DOCUMENT` a `false`.


> [!WARNING]
> - No guardes credenciales ni OCIDs sensibles directamente en el notebook.
> - No publiques el archivo `.env`, la llave privada `.pem` ni el archivo `config`.
> - Utiliza políticas IAM con los permisos mínimos necesarios.
> - No cargues información personal o confidencial.

### Problemas frecuentes

**`KeyError: 'OCI_COMPARTMENT_ID'` (u otra variable).** El notebook no lee el `.env`. Verifica que se llame exactamente `.env`, que esté en la misma carpeta que `notebook.ipynb`, que el nombre de la variable esté bien escrito y que hayas reiniciado el kernel.

**`ConfigFileNotFound`.** El archivo `config` no está en `<tu-usuario>\.oci\` o se guardó con otra extensión. Vuelve a guardarlo con **File > Save As...** con el nombre exacto `config`.

**`InvalidConfig: {'fingerprint': 'malformed'}`.** El `fingerprint` conserva el texto de ejemplo, se copió incompleto o tiene comillas. Cópialo de nuevo desde **My profile > API keys**.

**`TypeError: Invalid "auth" argument`.** Se instaló `openai` 3.x. Revisa que `requirements.txt` incluya `openai>=2.6.1,<3`, vuelve a crear el ambiente y reinicia el kernel.

**Python 3.14 no aparece al crear el ambiente.** Reinicia VS Code después de instalar Python, o selecciona el intérprete manualmente con **Enter interpreter path...**.

**`Vector creation failed`.** Es un mensaje genérico de la consola, y la causa más probable son los permisos. Pide a tu administrador una política como `allow group <your-group> to manage generative-ai-family in compartment <your-compartment>` y, para datos no estructurados, el dynamic group y la política de Object Storage que describe la guía *QuickStart Permissions for Developing Agents* de la documentación de OCI. Puedes consultar el detalle del fallo en **Governance & Administration > Work requests**.

**`NameError: name 'VECTOR_STORE_ID' is not defined`.** Se ejecutó una celda sin haber ejecutado antes las anteriores. Ejecuta el notebook en orden, desde la primera celda.

**`NotAuthorizedOrNotFound`.** Revisa que la región, el compartment, el proyecto y el ID del vector store coincidan entre sí.
