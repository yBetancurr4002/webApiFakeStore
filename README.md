# webApiFakeStore - APIs con .NET

## ¿Qué es una API?

Una **API (Application Programming Interface)** es un proveedor de métodos y funciones a otras aplicaciones. Se trata de un conjunto de definiciones y protocolos que se utiliza para desarrollar e integrar el software de las aplicaciones, permitiendo la comunicación entre dos aplicaciones de software a través de un conjunto de reglas.

Una API es indiferente a la plataforma tecnológica que la consuma y una API puede consumir otras API.

### Caraterísticas:
* Exponer métodos y funciones para ser consumido por otras aplicaciones.
* Posee una capa de abstracción para ser consumida
* Puede tener capas de seguridad que brinden autenticación y autorización.

## ¿Qué es REST?
**REST (REPRESENTATIONAL STATE TRANSFER)** es una interfaz para conectar varios sistemas basados en el protocolo HTTP (uno de los protocolos más antiguos) y nos sirve para obtener y generar datos y operaciones, devolviendo esos datos en formatos muy específicos, como XML y JSON.

### Características:
* Estilo de arquitectura para el diseño de interfaces web.
* Permite el manejo de recuros desde el lado del servidor.
* Mejor rendimiento, mayor escalabilidad, simplicidad y confiabilidad.

### Verbos HTTP

* **GET**: Permite obtener y leer recursos.
* **POST**: Creación de recursos.
* **PUT**: Actualización de un registro.
* **PATCH**: Actualización parcial de un registro.
* **DELETE**: Eliminación de registros

### Manejo de URLs

* URI: Los recursos en **REST** siempre se manipulan a partir de la URL, identificadores universales de recursos.
* Se necesita una URL por recurso

### Tipos de Respuestas HTTP

* Informativas : 100-199
* Satisfactorias : 200-299
* Se insertó bien 201
* Se realizó bien el proceso pero no cuenta con nada para responder : 204
* Redirecciones : 300-399
* Error Cliente : 400-499
* Error Server : 500-599

# Funcionamiento de una API en DotNet
## Consumiendo una API desde Postman

En este proyecto se trabajará con Postman para consumir las APIs.

## Análisis del template para APIs de .NET

MVC o model view controller es un patrón de diseño que nos permite separar las responsabilidades de nuestro proyecto.

Con web api se elimina la interfaz de usuario.

### Componentes

A continuación se listan los archivos por defecto que se crean al crear un proyecto.

* **Controllers**: los archivos que nos permiten otorgar la lógica a cada una de las clases
* **Models**: son los modelos que configuran las clases de los datos que se van a consumir
* **webapi.csproj**: configuración de paquetes del proyecto.
* **Program.cs**: contiene la configuración del proyecto y la forma como se va a ejecutar.
* **appsettings.Development.json**:
* **appsettings.json**: configuraciones como contraseñas, cadenas de conexión, ...
* **Launchsettings.json**: cómo se va a ejecutar la API.

## Atributos para verbos HTTP

Los verbos se manejan desde los controladores, para esto, indicamos el Verbo hacia el cual vamos a hacer el llamado. Obsérvese en el siguiente ejemplo:

```csharp
public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
        if (ListWeatherForecast == null || !ListWeatherForecast.Any())
        {
            ListWeatherForecast = Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        }).ToList();   
        }
    }

// Método GET: Leer los datos
    [HttpGet(Name = "GetWeatherForecast")]
    {
    return ListWeatherForecast;
    }

    // Método POST: 

    [HttpPost]
    public IActionResult Post(WeatherForecast weatherForecast)
    {
        ListWeatherForecast.Add(weatherForecast);

        return Ok();
    }

    // Método Eliminar - Indicar el Id
    [HttpDelete("{index}")]
    public IActionResult Delete(int index)
    {
        ListWeatherForecast.RemoveAt(index);

        return Ok();
    }
```

## Manejo de rutas

Puedo asginar más de una ruta tanto a mis controladores como a mis métodos:

```csharp
[Route("[controller]")]
[Route("OtraRuta")]
```

Simplemente agregando otra *Route*.

Puedo definir que la Ruta este relacionada con el nombre del método:

```csharp
 [HttpGet(Name = "GetWeatherForecast")]
 [Route("[Action]")]
    public IEnumerable<WeatherForecast> Get()
```

De esta manera puedo acceder desde la ruta definida desde el Controlador o desde la ruta que hace referencia al Action:
<pre>http://localhost:5196/weatherforecast</pre>

Estos dos son ejemplos de cómo podemos manejar las rutas de una manera dinámica y estática, notese como se utiliza ( "[...]" ) cuando queremos hacer referencias dinámicas

## Minimal API Vs Web API

Nueva plantilla de API con un estilo minimalista. Utiliza las últimas mejoras de C# y .NET para disminuir líneas de código, facilitando la curva de aprendizaje de APIs en .NET.

Una “Minimal API” es una API que proporciona solo las funcionalidades esenciales para cumplir con sus requisitos. Se enfoca en ser simple, rápida y fácil de utilizar. Una Minimal API solo incluirá los recursos y los métodos HTTP necesarios para cumplir con las necesidades de los usuarios.

![Cuándo usar cual](./minimalvsweb.png)

# Arquitectura y configuración

## ¿Qué son los middlewares?

En .NET un middleware es una clase que permite manipular una petición (o respuesta) HTTP.

Son una serie de instrucciones de código que se agregan al ciclo de vida de una petición HTTP. Provee una ejecución de peticiones a través de capas. Facilita la implementación de filtros sobre la api.

![Orden de los Middleware](./OrdenMiddleware.png)

## Creando un nuevo middleware

Un middleware es una clase de C#

```csharp

public class TimeMiddleware
{
    // Propiedad para invocar el siguiente Middleware
    readonly RequestDelegate next;

    // constructor
    public TimeMiddleware(RequestDelegate nextRequest)
    {
        next = nextRequest;
    }

    //
    public async Task Invoke(HttpContext context)
    {
        await next(context);
        if (context.Request.Query.Any(p => p.Key == "time"))
        {
            await context.Response.WriteAsync(DateTime.Now.ToShortTimeString());
        }
    }

}

  // Poder agregar el middleware en program.cs

    public static class TimeMiddlewareExtension
    {
        public static IApplicationBuilder UseTimeMiddleware(this IApplicationBuilder builder)
        {
            return builder.UseMiddleware<TimeMiddleware>();
        }
    }

```

* Los dos primeros métodos son de configuración, son basicamente una "plantilla"

* Creamos un contexto que indica bajo qué condiciones se ejecuta este middleware, para este caso, si se pasa como parámetro "time" <pre> http://localhost:5196?time</pre>

* Finalmente, implementamos otra clase que nos permite exportarlo en nuestro archivo principal, para poder invocarlo.


## Inyección de dependencias
Inyección de Dependencias es un patrón de diseño de software que nos permite desarrollar componentes acoplados libremente o desacoplados para obtener como resultado la fácil gestión de cambios a futuro, implementación fácil de pruebas unitarias, factoría para emitir instancias de clases, prevención de fugas de memoria, entre otros.

### ¿Qué es una dependencia?

...

## Agregando Logging a API

Agregar logging a una API es importante para tener una visibilidad completa sobre la actividad de la API y para poder solucionar problemas más eficientemente.

```csharp
private readonly ILogger<WeatherForecastController> _logger;

public IActionResult Post([FromBody] WeatherForecast weatherForecast)
	{
		_logger.LogDebug("Log Al Insertar Data");
		ListWeatherForecast.Add(weatherForecast);
		return Ok();
	}
``` 

Configuración

1. *app.settings.json*

## Documentando API con Swagger

Swagger es una herramienta basada en el estándar OpenAPI que nos permite documentar y probar nuestros Web APIs, para que sean fácilmente accesibles y entendibles por los usuarios o desarrolladores que pretendan utilizarlos.

* La configuración predeterminada indica que el ambiente de aplicación es desarollo, porque esta configuración no debe verse en prudcción, ya que revela información sensible de la API.

```csharp
// Program.cs
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

* Swagger nos puede servir para probar las diferentes funcionalidades de nuestra API, sin embargo, es mayormenteusada para la docuemtnaciónn de esta.

# Manipulación de datos con Entity Framework

## Agregando librerías para Entity Framework

1. [nuget.org](https://www.nuget.org/)
2. Ejecutar los CLI de: Microsft Entity Framework Core => Librería Entity Framework 
3. Ejecutar los CLI de: Microsoft Entity Framework Core In Memory =>  Manejar base de datos en memoria
4. Ejecutar los CLI de: Entity Framwe work SQL Server = Manejo de BD

## Configuración de Entity framework y clases base

## Creación de servicios

Acciones ejecutadas en la API. Generaremos para cada clase los métodos asociados a los *endpoints*

### Usings
<pre>
using webapi;
using webapi.Models;
namespace webapi.Services;
</pre>

## Inyectando servicios como dependencia

Agregaremos los servicios de Categorias y Tareas a nuestro programa principal. Recordar que para esto accedemos a los métodos del *builder*.

```csharp
//configurar inyección dependencias
builder.Services.AddScoped<IHelloWorldService, HelloWorldService>();
builder.Services.AddScoped<ICategoriaService, CategoriaService>();
builder.Services.AddScoped<ITareasService, TareasService>();
```
* La inyección del HelloWorld, se había realizao en ejercicios previos

## Creando controladores
... 

## Probando API con una base de datos SQL server

1. Program.Cs => realizar la configuración de EF

```csharp
builder.Services.AddSqlServer<TareasContext>("Connection String");
```
2. Validar en Postman
3. Ejecutar las acciones planteadas en los endpoints


****************************
Les recomiendo usar el comando:

dotnet watch run, en lugar de
dotnet run.
Lo que hace este comando es, compilar de forma automática cada vez que se modifica un archivo y no tener que estar bajando el servicio cada vez que modificamos algo.


 
