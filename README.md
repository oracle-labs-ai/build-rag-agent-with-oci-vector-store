# Agente RAG con LangChain y OCI Vector Store

Este proyecto contiene un laboratorio práctico para construir un agente de generación aumentada por recuperación (RAG) con LangChain y OCI Generative AI.

El agente trabaja con documentos Markdown almacenados en un OCI Vector Store: carga e indexa contenido, realiza búsquedas semánticas y utiliza los resultados para responder preguntas mediante una tool de LangChain.

El laboratorio también incluye la configuración de credenciales locales de OCI y una sección para evaluar la creación, carga y eliminación de varios vector stores. El notebook es el punto principal del proyecto y reúne las explicaciones y el código ejecutable.

Consulta el laboratorio completo en [notebook.ipynb](notebook.ipynb).


## Primeros pasos antes de abrir el notebook

Esta guía deja todo listo para ejecutar el laboratorio **usando solo la interfaz de Visual Studio Code**, sin escribir comandos en una terminal. Las únicas acciones fuera de VS Code son la instalación de programas y la preparación de recursos en la consola web de OCI.

Al terminar tendrás:

- Python 3.14 y Visual Studio Code con las extensiones necesarias.
- Una carpeta de proyecto con el notebook, un documento Markdown y un archivo `.env`.
- Un perfil de OCI (`config`) con una API key.
- Un proyecto y un vector store de OCI Generative AI.
- Un ambiente virtual `.venv` con todas las dependencias instaladas.

### 1. Instalar los programas

1. Descarga e instala **Python 3.14** desde [python.org](https://www.python.org/downloads/). Elige la última versión 3.14.x. Es la versión con la que se validó este laboratorio.
2. Descarga e instala **Visual Studio Code** desde [code.visualstudio.com](https://code.visualstudio.com/).

Si trabajas en un equipo de empresa, usa el portal de software de tu organización si la instalación directa está restringida.

### 2. Instalar las extensiones de VS Code

1. Abre VS Code y pulsa `Ctrl + Shift + X` (en macOS, `Cmd + Shift + X`) para abrir el panel de extensiones.
2. Busca e instala **Python** (publicada por Microsoft).
3. Busca e instala **Jupyter** (publicada por Microsoft).

### 3. Crear la carpeta del proyecto

1. En VS Code, ve a **File > Open Folder...**.
2. Crea una carpeta nueva con el nombre que prefieras (por ejemplo, `rag-oci-lab`) y ábrela. Si VS Code pregunta si confías en los autores de la carpeta, elige **Yes, I trust the authors**.
3. Copia `notebook.ipynb` dentro de la carpeta arrastrándolo al panel **Explorer** de VS Code.
4. En el panel **Explorer**, pulsa el icono **New Folder** y crea una carpeta llamada exactamente `data`.
5. Con la carpeta `data` seleccionada, pulsa **New File** y crea un archivo Markdown, por ejemplo `documento.md`. Pega o escribe contenido real (de una a tres páginas sobre un tema que conozcas). **No uses información personal ni confidencial.**

El notebook necesita al menos un archivo `.md` en `data/`.

### 4. Preparar OCI en la consola web

Estos pasos se hacen una sola vez en la consola de OCI, desde el navegador.

#### 4.1 Elegir la región

Selecciona la región en la parte superior derecha de la consola. Se recomienda `us-ashburn-1` (US East, Ashburn), la región con la que se validó el laboratorio. **El proyecto, el vector store y la variable `OCI_REGION` deben usar la misma región.**

#### 4.2 Copiar el OCID del compartment

1. Abre el menú principal (☰) y ve a **Identity & Security > Compartments**.
2. Entra al compartment (o subcompartment) donde trabajarás.
3. Junto a **OCID**, pulsa **Copy**. Empieza con `ocid1.compartment.oc1..`.

Usa **un solo compartment** para el proyecto y el vector store. Guarda el valor en un bloc de notas: irá en `OCI_COMPARTMENT_ID`.

#### 4.3 Crear la API key

1. Pulsa tu icono de perfil (arriba a la derecha) y entra a **My profile**.
2. Ve a **API keys** y pulsa **Add API key**.
3. Elige **Generate API key pair**, pulsa **Download private key** (archivo `.pem`) y luego **Add**.
4. Se mostrará la ventana **Configuration file preview**. Déjala abierta o copia su contenido.

#### 4.4 Crear el proyecto de Generative AI

1. Confirma que la región seleccionada sea la correcta.
2. Ve a **Analytics & AI > Generative AI > Projects**. Si no ves la opción, búscala con el buscador de la consola escribiendo `Projects`.
3. En el panel izquierdo, selecciona el compartment del paso 4.2.
4. Pulsa **Create project**, escribe un nombre (por ejemplo, `rag-lab`) y confirma con **Create**.
5. Abre el proyecto y copia su **OCID** (`ocid1.generativeaiproject...`). Irá en `OCI_GENERATIVE_AI_PROJECT_ID`.

#### 4.5 Crear el vector store

1. Busca **Vector stores** en el menú de Generative AI o en el buscador de la consola.
2. Selecciona el mismo compartment y la misma región del proyecto.
3. Pulsa **Create vector store** y escribe un nombre (por ejemplo, `rag-lab-store`).
4. Deja desactivada la opción de semantic store (NL2SQL): este laboratorio usa documentos Markdown, es decir, datos no estructurados.
5. Opcional: activa una política de expiración para que se elimine tras unos días de inactividad.
6. Pulsa **Create**, abre el vector store y copia su **ID**. Irá en `OCI_VECTOR_STORE_ID`.

Si aparece `Vector creation failed`, consulta la sección de problemas frecuentes.

### 5. Crear el archivo de configuración de OCI

El notebook se autentica con un perfil del archivo `config`, que debe vivir en la carpeta `.oci` de tu directorio de usuario (por ejemplo, `C:\Users\<tu-usuario>\.oci\config`). Se puede crear completamente desde VS Code:

1. En VS Code, ve a **File > New Text File**.
2. Pega el contenido de la ventana **Configuration file preview** y completa los datos:

```ini
[DEFAULT]
user=ocid1.user.oc1..<your-user-ocid>
fingerprint=<your-fingerprint>
tenancy=ocid1.tenancy.oc1..<your-tenancy-ocid>
region=us-ashburn-1
key_file=C:\Users\<tu-usuario>\.oci\oci_api_key.pem
```

3. Ve a **File > Save As...**, navega a tu carpeta de usuario y crea una carpeta llamada `.oci`. Si Windows no permite crear una carpeta que empiece con punto, escribe `.oci.` (con un punto al final): Windows lo elimina automáticamente.
4. Guarda el archivo dentro de `.oci` con el nombre exacto `config` (sin extensión). VS Code no agrega `.txt`.
5. Mueve la llave privada descargada (`.pem`) a la carpeta `.oci` y renómbrala como `oci_api_key.pem`, o ajusta `key_file` a la ruta donde la dejaste. Puedes hacerlo con el Explorador de archivos.

Revisa estos detalles:

- Reemplaza todos los valores de ejemplo (`<...>`) por tus datos reales.
- El `fingerprint` debe estar en una sola línea, con 16 pares hexadecimales separados por `:`, sin comillas.
- Si no encuentras tus OCIDs, el de usuario está en **My profile** y el de tenancy en **Tenancy** (menú del icono de perfil).
- El nombre del perfil es la palabra entre corchetes (`DEFAULT` en el ejemplo). Si usas otro nombre, tendrás que indicarlo en `OCI_PROFILE`.

> **Nota:** no instales `oci-cli` ni uses `oci setup config`. Con Python 3.14 una dependencia del CLI (PyYAML) intenta compilarse y falla en Windows si no están instaladas las Microsoft C++ Build Tools. Crear el archivo `config` manualmente es suficiente.

### 6. Crear el archivo `.env`

1. En el panel **Explorer** de VS Code, con la carpeta raíz seleccionada, pulsa **New File** y nómbralo exactamente `.env`.
2. Escribe las variables, una por línea, sin comillas ni espacios alrededor del `=`:

```dotenv
OCI_COMPARTMENT_ID=ocid1.compartment.oc1..<your-compartment-ocid>
OCI_GENERATIVE_AI_PROJECT_ID=ocid1.generativeaiproject.oc1..<your-project-ocid>
OCI_VECTOR_STORE_ID=<your-vector-store-id>
OCI_REGION=us-ashburn-1
OCI_PROFILE=DEFAULT
OCI_SOURCE_DOCUMENT=documento.md
OCI_UPLOAD_DOCUMENT=true
```

| Variable | Descripción | Valor si no se define |
|---|---|---|
| `OCI_COMPARTMENT_ID` | OCID del compartment | Obligatoria |
| `OCI_GENERATIVE_AI_PROJECT_ID` | OCID del proyecto | Obligatoria |
| `OCI_VECTOR_STORE_ID` | ID del vector store | Obligatoria |
| `OCI_REGION` | Región de OCI | `us-chicago-1` |
| `OCI_PROFILE` | Nombre del perfil de `config` | `DEFAULT` |
| `OCI_MODEL_ID` | Modelo del agente (opcional) | `openai.gpt-5` |
| `OCI_SOURCE_DOCUMENT` | Archivo dentro de `data/` | Primer `.md` disponible |
| `OCI_UPLOAD_DOCUMENT` | Activa la carga del documento | `false` |

Define siempre `OCI_REGION` y `OCI_PROFILE`: si no lo haces, el notebook usa `us-chicago-1` y el perfil `DEFAULT`, que pueden no coincidir con tu configuración. `OCI_UPLOAD_DOCUMENT=true` es necesario la primera vez porque el vector store está vacío. Después de indexar el documento, cámbialo a `false` para no duplicarlo.

### 7. Crear el archivo de dependencias

En el panel **Explorer**, crea un archivo nuevo llamado `requirements.txt` con este contenido:

```text
langchain>=1,<2
langchain-openai>=1,<2
openai>=2.6.1,<3
oci
oci-openai
httpx
python-dotenv
ipykernel
```

**No quites el límite de `openai`.** Sin él se instala la serie 3.x, que todavía no es compatible con `oci-openai` y produce `TypeError: Invalid "auth" argument` al crear el cliente. Cuando `oci-openai` soporte la serie 3.x, podrás retirar el límite.

### 8. Crear el ambiente virtual desde VS Code

1. Pulsa `Ctrl + Shift + P` (en macOS, `Cmd + Shift + P`) para abrir la paleta de comandos.
2. Escribe **Python: Create Environment** y selecciónalo.
3. Elige **Venv**.
4. Selecciona el intérprete **Python 3.14**. Si no aparece en la lista, cierra y vuelve a abrir VS Code, o usa **Enter interpreter path... > Find...** y busca el `python.exe` de la versión 3.14.
5. Cuando VS Code muestre la lista de dependencias, **marca `requirements.txt`** y confirma.
6. Espera a que termine. Verás el progreso en la esquina inferior derecha. Se crea la carpeta `.venv` en el proyecto con las librerías ya instaladas.
7. Si VS Code pregunta si quieres usar el nuevo ambiente para el workspace, responde que sí.

No necesitas activar el ambiente ni abrir una terminal. Este método evita el error de PowerShell *running scripts is disabled on this system*, que ocurre al activar `.venv` manualmente.

### 9. Verificar la autenticación con OCI

Antes de abrir el notebook, comprueba que tu perfil y el compartment funcionan:

1. Crea un archivo nuevo llamado `test_oci.py` y pega este código. La línea `# %%` convierte el bloque en una celda ejecutable:

```python
# %%
import oci

config = oci.config.from_file(profile_name="DEFAULT")  # nombre de tu perfil
identity = oci.identity.IdentityClient(config)

compartment = identity.get_compartment("<your-compartment-ocid>")
print("OK:", compartment.data.name)
```

2. Sobre `# %%` aparece el enlace **Run Cell**. Púlsalo.
3. Si VS Code pide seleccionar un kernel, elige el ambiente `.venv`.
4. Se abrirá una ventana interactiva con el resultado. Si imprime `OK:` con el nombre del compartment, la autenticación está lista.

Puedes borrar `test_oci.py` cuando termines.

### 10. Abrir el notebook y seleccionar el kernel

1. En el **Explorer**, abre `notebook.ipynb`.
2. Arriba a la derecha pulsa **Select Kernel > Python Environments...** y elige el ambiente `.venv`. Si VS Code ofrece instalar `ipykernel`, acepta.
3. Ejecuta las celdas **en orden**, una por una, con `Shift + Enter` o con el botón de ejecución a la izquierda de cada celda. El número entre corchetes junto a la celda indica que se ejecutó; `[*]` significa que sigue en ejecución.
4. Si cambias `.env`, usa el botón **Restart** de la barra del notebook para volver a leer las variables, y ejecuta de nuevo desde la primera celda.
5. Ejecuta la celda de carga del documento **una sola vez**. Debe terminar con `Status: completed`. Después cambia `OCI_UPLOAD_DOCUMENT` a `false`.

### Problemas frecuentes antes de empezar

**`KeyError: 'OCI_COMPARTMENT_ID'` (u otra variable).** El notebook no lee el `.env`. Verifica que se llame exactamente `.env`, que esté en la misma carpeta que `notebook.ipynb`, que el nombre de la variable esté bien escrito y que hayas reiniciado el kernel.

**`ConfigFileNotFound`.** El archivo `config` no está en `<tu-usuario>\.oci\` o se guardó con otra extensión. Vuelve a guardarlo con **File > Save As...** con el nombre exacto `config`.

**`InvalidConfig: {'fingerprint': 'malformed'}`.** El `fingerprint` conserva el texto de ejemplo, se copió incompleto o tiene comillas. Cópialo de nuevo desde **My profile > API keys**.

**`TypeError: Invalid "auth" argument`.** Se instaló `openai` 3.x. Revisa que `requirements.txt` incluya `openai>=2.6.1,<3`, vuelve a crear el ambiente y reinicia el kernel.

**Python 3.14 no aparece al crear el ambiente.** Reinicia VS Code después de instalar Python, o selecciona el intérprete manualmente con **Enter interpreter path...**.

**`Vector creation failed`.** Es un mensaje genérico de la consola, y la causa más probable son los permisos. Pide a tu administrador una política como `allow group <your-group> to manage generative-ai-family in compartment <your-compartment>` y, para datos no estructurados, el dynamic group y la política de Object Storage que describe la guía *QuickStart Permissions for Developing Agents* de la documentación de OCI. Puedes consultar el detalle del fallo en **Governance & Administration > Work requests**.

**`NameError: name 'VECTOR_STORE_ID' is not defined`.** Se ejecutó una celda sin haber ejecutado antes las anteriores. Ejecuta el notebook en orden, desde la primera celda.

**`NotAuthorizedOrNotFound`.** Revisa que la región, el compartment, el proyecto y el ID del vector store coincidan entre sí.

### Notas de seguridad

- No guardes credenciales ni OCIDs sensibles directamente en el notebook.
- No publiques el archivo `.env`, la llave privada `.pem` ni el archivo `~/.oci/config`.
- Crea un archivo `.gitignore` con estas líneas para evitar subirlos por error:

```text
.env
.venv/
*.pem
```

- Utiliza políticas IAM con los permisos mínimos necesarios.
- No cargues información personal o regulada en un laboratorio de demostración.
