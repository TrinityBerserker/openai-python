# Biblioteca de Python para la API de OpenAI
[![Versión de PyPI](https://img.shields.io/pypi/v/openai.svg)](https://pypi.org/project/openai/)

La biblioteca de Python de OpenAI proporciona acceso conveniente a la API REST de OpenAI desde cualquier aplicación Python 3.7+. La biblioteca incluye definiciones de tipos para todos los parámetros de solicitud y campos de respuesta, y ofrece clientes síncronos y asíncronos con tecnología de [httpx](https://github.com/encode/httpx).

Fue generada a partir de la [especificación OpenAPI](https://github.com/openai/openai-openapi) con [Stainless](https://stainlessapi.com/).

## Documentación
La documentación de la API REST se puede encontrar en [platform.openai.com](https://platform.openai.com/docs). La API completa de esta biblioteca se puede encontrar en [api.md](api.md).

## Instalación
> [!IMPORTANTE]
> El SDK fue reescrito en la v1, que se lanzó el 6 de noviembre de 2023. Consulte la [guía de migración a v1](https://github.com/openai/openai-python/discussions/742), que incluye scripts para actualizar automáticamente su código.

```sh
# instalar desde PyPI
pip install openai
```

## Uso
La API completa de esta biblioteca se puede encontrar en [api.md](api.md).

```python
import os
from openai import OpenAI

client = OpenAI(
    # Este es el valor predeterminado y se puede omitir
    api_key=os.environ.get("OPENAI_API_KEY"),
)

chat_completion = client.chat.completions.create(
    messages=[
        {
            "role": "user",
            "content": "Di que esto es una prueba",
        }
    ],
    model="gpt-3.5-turbo",
)
```

Aunque puede proporcionar un argumento de palabra clave `api_key`, recomendamos usar [python-dotenv](https://pypi.org/project/python-dotenv/) para agregar `OPENAI_API_KEY="Mi clave de API"` a su archivo `.env` para que su clave de API no se almacene en el control de versiones.

### Visión
Con una imagen alojada:
```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": prompt},
                {
                    "type": "image_url",
                    "image_url": {"url": f"{img_url}"},
                },
            ],
        }
    ],
)
```
Con la imagen como una cadena codificada en base64:
```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": prompt},
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:{img_type};base64,{img_b64_str}"},
                },
            ],
        }
    ],
)
```
### Ayudantes de Sondeo
Algunas acciones como iniciar una Ejecución o agregar archivos a almacenes vectoriales son asíncronas y requieren tiempo para completarse. El SDK incluye funciones auxiliares que sondearán el estado hasta que alcance un estado terminal y luego devolverán el objeto resultante.

Si un método de API genera una acción que podría beneficiarse del sondeo, habrá una versión correspondiente del método que termine en '\_and_poll'.

Por ejemplo, para crear una Ejecución y sondear hasta que alcance un estado terminal:
```python
run = client.beta.threads.runs.create_and_poll(
    thread_id=thread.id,
    assistant_id=assistant.id,
)
```
Más información sobre el ciclo de vida de una Ejecución se puede encontrar en la [Documentación del Ciclo de Vida de Ejecución](https://platform.openai.com/docs/assistants/how-it-works/run-lifecycle)


### Ayudantes de Streaming
El SDK también incluye ayudantes para procesar flujos de datos y manejar eventos entrantes.
```python
with client.beta.threads.runs.stream(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="Por favor, dirígete al usuario como Jane Doe. El usuario tiene una cuenta premium.",
) as stream:
    for event in stream:
        # Imprimir el texto de los eventos delta de texto
        if event.type == "thread.message.delta" and event.data.delta.content:
            print(event.data.delta.content[0].text)
```
Puede encontrar más información sobre los ayudantes de streaming en la documentación dedicada: [helpers.md](helpers.md)

## Uso asíncrono
Simplemente importe `AsyncOpenAI` en lugar de `OpenAI` y use `await` con cada llamada a la API:
```python
import os
import asyncio
from openai import AsyncOpenAI
client = AsyncOpenAI(
    # Este es el valor predeterminado y puede omitirse
    api_key=os.environ.get("OPENAI_API_KEY"),
)
async def main() -> None:
    chat_completion = await client.chat.completions.create(
        messages=[
            {
                "role": "user",
                "content": "Di que esto es una prueba",
            }
        ],
        model="gpt-3.5-turbo",
    )
asyncio.run(main())
```
La funcionalidad entre los clientes síncronos y asíncronos es por lo demás idéntica.

## Respuestas de streaming
Proporcionamos soporte para respuestas de streaming utilizando Eventos del Lado del Servidor (SSE).
```python
from openai import OpenAI
client = OpenAI()
stream = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Di que esto es una prueba"}],
    stream=True,
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")
```
El cliente asíncrono utiliza exactamente la misma interfaz.
```python
from openai import AsyncOpenAI
client = AsyncOpenAI()
async def main():
    stream = await client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": "Di que esto es una prueba"}],
        stream=True,
    )
    async for chunk in stream:
        print(chunk.choices[0].delta.content or "", end="")
asyncio.run(main())
```

## Cliente a nivel de módulo
> [!IMPORTANTE]
> Recomendamos encarecidamente crear instancias de cliente en lugar de depender del cliente global.

También exponemos una instancia de cliente global a la que se puede acceder de manera similar a las versiones anteriores a v1.

```py
import openai
# opcional; por defecto será `os.environ['OPENAI_API_KEY']`
openai.api_key = '...'
# todas las opciones del cliente se pueden configurar igual que con la instancia de `OpenAI`
openai.base_url = "https://..."
openai.default_headers = {"x-foo": "true"}
completion = openai.chat.completions.create(
    model="gpt-4",
    messages=[
        {
            "role": "user",
            "content": "¿Cómo imprimo todos los archivos de un directorio usando Python?",
        },
    ],
)
print(completion.choices[0].message.content)
```

La API es exactamente la misma que la API estándar basada en instancias de cliente.

Esto está pensado para usarse en REPL o cuadernos para una iteración más rápida, **no** en código de aplicación.

Recomendamos que siempre cree una instancia de cliente (por ejemplo, con `client = OpenAI()`) en código de aplicación porque:
- Puede ser difícil entender dónde se configuran las opciones del cliente
- No es posible cambiar ciertas opciones del cliente sin causar posibles condiciones de carrera
- Es más difícil de simular para pruebas
- No es posible controlar la limpieza de conexiones de red

## Uso de tipos
Los parámetros de solicitud anidados son [TypedDicts](https://docs.python.org/3/library/typing.html#typing.TypedDict). Las respuestas son [modelos Pydantic](https://docs.pydantic.dev) que también proporcionan métodos de ayuda para cosas como:
- Serializar de nuevo a JSON, `model.to_json()`
- Convertir a diccionario, `model.to_dict()`

Las solicitudes y respuestas escritas proporcionan autocompletado y documentación dentro de su editor. Si desea ver errores de tipo en VS Code para ayudar a detectar errores antes, establezca `python.analysis.typeCheckingMode` en `basic`.

## Paginación
Los métodos de lista en la API de OpenAI están paginados.

Esta biblioteca proporciona iteradores de auto-paginación con cada respuesta de lista, por lo que no tiene que solicitar páginas sucesivas manualmente:

```python
from openai import OpenAI
client = OpenAI()
all_jobs = []
# Obtiene automáticamente más páginas según sea necesario.
for job in client.fine_tuning.jobs.list(
    limit=20,
):
    # Hacer algo con job aquí
    all_jobs.append(job)
print(all_jobs)
```

O, de forma asíncrona:

```python
import asyncio
from openai import AsyncOpenAI
client = AsyncOpenAI()
async def main() -> None:
    all_jobs = []
    # Iterar a través de elementos en todas las páginas, emitiendo solicitudes según sea necesario.
    async for job in client.fine_tuning.jobs.list(
        limit=20,
    ):
        all_jobs.append(job)
    print(all_jobs)
asyncio.run(main())
```

Alternativamente, puede usar los métodos `.has_next_page()`, `.next_page_info()` o `.get_next_page()` para un control más granular al trabajar con páginas:

```python
first_page = await client.fine_tuning.jobs.list(
    limit=20,
)
if first_page.has_next_page():
    print(f"se obtendrá la siguiente página usando estos detalles: {first_page.next_page_info()}")
    next_page = await first_page.get_next_page()
    print(f"número de elementos que acabamos de obtener: {len(next_page.data)}")
# Eliminar `await` para uso no asíncrono.
```

O simplemente trabajar directamente con los datos devueltos:

```python
first_page = await client.fine_tuning.jobs.list(
    limit=20,
)
print(f"cursor de siguiente página: {first_page.after}")  # => "cursor de siguiente página: ..."
for job in first_page.data:
    print(job.id)
# Eliminar `await` para uso no asíncrono.
```



## Parámetros anidados

Los parámetros anidados son diccionarios, tipados usando `TypedDict`, por ejemplo:

```python
from openai import OpenAI

client = OpenAI()

completion = client.chat.completions.create(
    messages=[
        {
            "role": "user",
            "content": "¿Puedes generar un objeto JSON de ejemplo que describa una fruta?",
        }
    ],
    model="gpt-3.5-turbo-1106",
    response_format={"type": "json_object"},
)
```

## Cargas de archivos

Los parámetros de solicitud que corresponden a cargas de archivos pueden pasarse como `bytes`, una instancia de [`PathLike`](https://docs.python.org/3/library/os.html#os.PathLike) o una tupla de `(nombre de archivo, contenido, tipo de medio)`.

```python
from pathlib import Path
from openai import OpenAI

client = OpenAI()

client.files.create(
    file=Path("input.jsonl"),
    purpose="fine-tune",
)
```

El cliente asíncrono usa exactamente la misma interfaz. Si pasas una instancia de [`PathLike`](https://docs.python.org/3/library/os.html#os.PathLike), los contenidos del archivo se leerán de forma asíncrona automáticamente.

## Manejo de errores

Cuando la biblioteca no puede conectarse a la API (por ejemplo, debido a problemas de conexión de red o un tiempo de espera), se genera una subclase de `openai.APIConnectionError`.

Cuando la API devuelve un código de estado de no éxito (es decir, una respuesta 4xx o 5xx), se genera una subclase de `openai.APIStatusError`, que contiene las propiedades `status_code` y `response`.

Todos los errores heredan de `openai.APIError`.

```python
import openai
from openai import OpenAI

client = OpenAI()

try:
    client.fine_tuning.jobs.create(
        model="gpt-3.5-turbo",
        training_file="file-abc123",
    )
except openai.APIConnectionError as e:
    print("No se pudo acceder al servidor")
    print(e.__cause__)  # una excepción subyacente, probablemente generada dentro de httpx.
except openai.RateLimitError as e:
    print("Se recibió un código de estado 429; deberíamos reducir un poco.")
except openai.APIStatusError as e:
    print("Se recibió otro código de estado fuera del rango 200")
    print(e.status_code)
    print(e.response)
```

Los códigos de error son los siguientes:

| Código de Estado | Tipo de Error                |
| --------------- | ---------------------------- |
| 400            | `BadRequestError`            |
| 401            | `AuthenticationError`        |
| 403            | `PermissionDeniedError`      |
| 404            | `NotFoundError`              |
| 422            | `UnprocessableEntityError`   |
| 429            | `RateLimitError`             |
| >=500          | `InternalServerError`        |
| N/A            | `APIConnectionError`         |

## ID de solicitudes

> Para obtener más información sobre la depuración de solicitudes, consulte [estos documentos](https://platform.openai.com/docs/api-reference/debugging-requests)

Todas las respuestas de objetos en el SDK proporcionan una propiedad `_request_id` que se añade desde la cabecera de respuesta `x-request-id` para que pueda registrar rápidamente las solicitudes fallidas e informar sobre ellas a OpenAI.

```python
completion = await client.chat.completions.create(
    messages=[{"role": "user", "content": "Di que esto es una prueba"}], model="gpt-4"
)
print(completion._request_id)  # req_123
```

Tenga en cuenta que, a diferencia de otras propiedades que usan un prefijo `_`, la propiedad `_request_id` *es* pública. A menos que se documente lo contrario, *todas* las demás propiedades, métodos y módulos con prefijo `_` son *privados*.

### Reintentos

Ciertos errores se reintentan automáticamente 2 veces por defecto, con una breve retrocesión exponencial.
Los errores de conexión (por ejemplo, debido a un problema de conectividad de red), 408 Request Timeout, 409 Conflict,
429 Rate Limit, y errores internos >=500 son reintentados por defecto.

Puede usar la opción `max_retries` para configurar o desactivar la configuración de reintentos:

```python
from openai import OpenAI

# Configurar el valor predeterminado para todas las solicitudes:
client = OpenAI(
    # el valor predeterminado es 2
    max_retries=0,
)

# O, configurar por solicitud:
client.with_options(max_retries=5).chat.completions.create(
    messages=[
        {
            "role": "user",
            "content": "¿Cómo puedo obtener el nombre del día actual en Node.js?",
        }
    ],
    model="gpt-3.5-turbo",
)
```

### Tiempos de espera

Por defecto, las solicitudes agotan el tiempo de espera después de 10 minutos. Puede configurar esto con una opción `timeout`,
que acepta un flotante o un objeto [`httpx.Timeout`](https://www.python-httpx.org/advanced/#fine-tuning-the-configuration):

```python
from openai import OpenAI

# Configurar el valor predeterminado para todas las solicitudes:
client = OpenAI(
    # 20 segundos (el valor predeterminado es 10 minutos)
    timeout=20.0,
)

# Control más detallado:
client = OpenAI(
    timeout=httpx.Timeout(60.0, read=5.0, write=10.0, connect=2.0),
)

# Anular por solicitud:
client.with_options(timeout=5.0).chat.completions.create(
    messages=[
        {
            "role": "user",
            "content": "¿Cómo puedo listar todos los archivos en un directorio usando Python?",
        }
    ],
    model="gpt-3.5-turbo",
)
```

Si se agota el tiempo de espera, se lanza un `APITimeoutError`.

Tenga en cuenta que las solicitudes que agotan el tiempo de espera [se reintentan dos veces por defecto](#reintentos).

## Avanzado

### Registro

Usamos el módulo [`logging`](https://docs.python.org/3/library/logging.html) de la biblioteca estándar.

Puede habilitar el registro configurando la variable de entorno `OPENAI_LOG` en `debug`.

```shell
$ export OPENAI_LOG=debug
```



## Cómo determinar si `None` significa `null` o está ausente

En una respuesta de API, un campo puede ser explícitamente `null`, o estar completamente ausente; en ambos casos, su valor es `None` en esta biblioteca. Puede diferenciar los dos casos con `.model_fields_set`:

```py
if response.my_field is None:
  if 'my_field' not in response.model_fields_set:
    print('Obtuve un json como {}, sin una clave "my_field" presente en absoluto.')
  else:
    print('Obtuve un json como {"my_field": null}.')
```

## Accediendo a datos de respuesta raw (por ejemplo, cabeceras)

El objeto de Respuesta "raw" se puede acceder anteponiendo `.with_raw_response.` a cualquier llamada de método HTTP, por ejemplo:

```py
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.with_raw_response.create(
    messages=[{
        "role": "user",
        "content": "Di que esto es una prueba",
    }],
    model="gpt-3.5-turbo",
)
print(response.headers.get('X-My-Header'))

completion = response.parse()  # obtener el objeto que `chat.completions.create()` habría devuelto
print(completion)
```

Estos métodos devuelven un objeto [`LegacyAPIResponse`](https://github.com/openai/openai-python/tree/main/src/openai/_legacy_response.py). Esta es una clase heredada ya que la estamos cambiando ligeramente en la próxima versión principal.

Para el cliente síncrono, esto será prácticamente lo mismo, con la excepción de que `content` y `text` serán métodos en lugar de propiedades. En el cliente asíncrono, todos los métodos serán asincrónicos.

Se proporcionará un script de migración y la migración en general debería ser sencilla.

#### `.with_streaming_response`

La interfaz anterior lee ávidamente todo el cuerpo de la respuesta cuando se hace la solicitud, lo que puede no ser siempre lo que se desea.

Para transmitir el cuerpo de la respuesta, use `.with_streaming_response` en su lugar, que requiere un administrador de contexto y solo lee el cuerpo de la respuesta una vez que llama a `.read()`, `.text()`, `.json()`, `.iter_bytes()`, `.iter_text()`, `.iter_lines()` o `.parse()`. En el cliente asíncrono, estos son métodos asincrónicos.

Como tal, los métodos `.with_streaming_response` devuelven un objeto [`APIResponse`](https://github.com/openai/openai-python/tree/main/src/openai/_response.py) diferente, y el cliente asíncrono devuelve un objeto [`AsyncAPIResponse`](https://github.com/openai/openai-python/tree/main/src/openai/_response.py).

```python
with client.chat.completions.with_streaming_response.create(
    messages=[
        {
            "role": "user",
            "content": "Di que esto es una prueba",
        }
    ],
    model="gpt-3.5-turbo",
) as response:
    print(response.headers.get("X-My-Header"))

    for line in response.iter_lines():
        print(line)
```

El administrador de contexto es obligatorio para que la respuesta se cierre de manera confiable.

## Realizando solicitudes personalizadas/no documentadas

Esta biblioteca está tipada para un acceso conveniente a la API documentada.

Si necesita acceder a puntos finales, parámetros o propiedades de respuesta no documentados, la biblioteca aún se puede usar.

### Puntos finales no documentados

Para hacer solicitudes a puntos finales no documentados, puede hacer solicitudes usando `client.get`, `client.post` y otros verbos http. Las opciones del cliente (como reintentos) se respetarán al hacer esta solicitud.

```py
import httpx

response = client.post(
    "/foo",
    cast_to=httpx.Response,
    body={"my_param": True},
)

print(response.headers.get("x-foo"))
```

### Parámetros de solicitud no documentados

Si desea enviar explícitamente un parámetro adicional, puede hacerlo con las opciones de solicitud `extra_query`, `extra_body` y `extra_headers`.

### Propiedades de respuesta no documentadas

Para acceder a propiedades de respuesta no documentadas, puede acceder a los campos adicionales como `response.unknown_prop`. También puede obtener todos los campos adicionales en el modelo Pydantic como un diccionario con [`response.model_extra`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_extra).

### Configurando el cliente HTTP

Puede anular directamente el [cliente httpx](https://www.python-httpx.org/api/#client) para personalizarlo según su caso de uso, incluyendo:

- Soporte para proxies
- Transportes personalizados
- Funcionalidad [avanzada](https://www.python-httpx.org/advanced/clients/) adicional

```python
from openai import OpenAI, DefaultHttpxClient

client = OpenAI(
    # O usar la variable de entorno `OPENAI_BASE_URL`
    base_url="http://mi.servidor.prueba.ejemplo.com:8083/v1",
    http_client=DefaultHttpxClient(
        proxies="http://mi.proxy.prueba.ejemplo.com",
        transport=httpx.HTTPTransport(local_address="0.0.0.0"),
    ),
)
```

También puede personalizar el cliente por solicitud usando `with_options()`:

```python
client.with_options(http_client=DefaultHttpxClient(...))
```

### Administrando recursos HTTP

De manera predeterminada, la biblioteca cierra las conexiones HTTP subyacentes cuando el cliente es [recolectado por el recolector de basura](https://docs.python.org/3/reference/datamodel.html#object.__del__). Puede cerrar manualmente el cliente usando el método `.close()` si lo desea, o con un administrador de contexto que lo cierre al salir.

## Microsoft Azure OpenAI

Para usar esta biblioteca con [Azure OpenAI](https://learn.microsoft.com/azure/ai-services/openai/overview), use la clase `AzureOpenAI` en lugar de la clase `OpenAI`.

> [!IMPORTANTE]
> La forma de la API de Azure difiere de la forma de la API principal, lo que significa que los tipos estáticos para respuestas/parámetros no siempre serán correctos.

```py
from openai import AzureOpenAI

# obtiene la clave API de la variable de entorno AZURE_OPENAI_API_KEY
client = AzureOpenAI(
    # https://learn.microsoft.com/azure/ai-services/openai/reference#rest-api-versioning
    api_version="2023-07-01-preview",
    # https://learn.microsoft.com/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal#create-a-resource
    azure_endpoint="https://ejemplo-endpoint.openai.azure.com",
)

completion = client.chat.completions.create(
    model="nombre-de-despliegue",  # por ejemplo, gpt-35-instant
    messages=[
        {
            "role": "user",
            "content": "¿Cómo imprimo todos los archivos de un directorio usando Python?",
        },
    ],
)
print(completion.to_json())
```

Además de las opciones proporcionadas en el cliente base `OpenAI`, se proporcionan las siguientes opciones:

- `azure_endpoint` (o la variable de entorno `AZURE_OPENAI_ENDPOINT`)
- `azure_deployment`
- `api_version` (o la variable de entorno `OPENAI_API_VERSION`)
- `azure_ad_token` (o la variable de entorno `AZURE_OPENAI_AD_TOKEN`)
- `azure_ad_token_provider`

Se puede encontrar un ejemplo de uso del cliente con Microsoft Entra ID (anteriormente conocido como Azure Active Directory) [aquí](https://github.com/openai/openai-python/blob/main/examples/azure_ad.py).

## Versionado

Este paquete sigue generalmente las convenciones de [SemVer](https://semver.org/spec/v2.0.0.html), aunque ciertos cambios incompatibles con versiones anteriores pueden lanzarse como versiones menores:

1. Cambios que solo afectan los tipos estáticos, sin romper el comportamiento en tiempo de ejecución.
2. Cambios en los elementos internos de la biblioteca que son técnicamente públicos pero no están destinados o documentados para uso externo. _(Por favor, abra un issue en GitHub para hacernos saber si está utilizando dichos internos)_.
3. Cambios que no esperamos que impacten a la gran mayoría de los usuarios en la práctica.

Nos tomamos la compatibilidad con versiones anteriores muy en serio y nos esforzamos por garantizar que pueda contar con una experiencia de actualización fluida.

Estamos interesados en sus comentarios; por favor, abra un [issue](https://www.github.com/openai/openai-python/issues) con preguntas, errores o sugerencias.

### Determinando la versión instalada

Si ha actualizado a la última versión pero no ve las nuevas características que esperaba, es probable que su entorno de Python aún esté usando una versión anterior.

Puede determinar la versión que se está utilizando en tiempo de ejecución con:

```py
import openai
print(openai.__version__)
```

## Requisitos

Python 3.7 o superior.

## Contribuyendo

Consulte [la documentación de contribución](./CONTRIBUTING.md).
