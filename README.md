# Proyecto: Configurar WSL 2, Docker y RabbitMQ con un Ejemplo Hello World 

## Descripción
Este proyecto proporciona una guía para configurar WSL 2, instalar Docker Desktop y ejecutar RabbitMQ utilizando una imagen oficial. Además, incluye un ejemplo práctico de comunicación con RabbitMQ utilizando dos aplicaciones de consola en C#.

---

## Instalación de WSL 2

1. Instala WSL 2 ejecutando el siguiente comando en PowerShell:
   ```powershell
   wsl --install
   ```

2. Verifica la instalación:
   ```powershell
   wsl -l -v
   ```

### Solución de problemas
- **Error: Permission Denied**
  Si te encuentras con el error: `error:docker: permission denied while trying to connect to the Docker`, verifica si el usuario `docker` tiene permisos suficientes en el sistema.

---

## Instalación de Docker Desktop

1. Descarga Docker Desktop desde su [página oficial](https://www.docker.com/get-started/).
2. Instala y configura Docker Desktop en tu sistema.

---

## Ejecución en Docker (RabbitMQ con Imágenes Oficiales)

**Requisitos:**
- Docker instalado y en funcionamiento en tu sistema.

### Ejecución de RabbitMQ

Utiliza el siguiente comando para ejecutar una instancia de RabbitMQ con el plugin de administración habilitado:

```bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management
```

Esto iniciará RabbitMQ y hará que los puertos 5672 (mensajes) y 15672 (interfaz de administración) estén accesibles.

---

## Ejemplo Hello World con RabbitMQ

Sigue el tutorial oficial de RabbitMQ para .NET disponible en: [RabbitMQ Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-dotnet)

### Crear la Solución en Visual Studio

1. Crea una solución con dos proyectos de consola:
   - **RabbitMQ-Receive**
   - **RabbitMQ-Send**

   ![Estructura de la solución](attachment:5244af2c-e5f2-4721-a9c1-624cdc18acc1:image.png)

2. Agrega la dependencia **RabbitMQ.Client (7.0.0)** a ambos proyectos.

### Configuración del Código

#### Program.cs (RabbitMQ-Receive)

```csharp
using RabbitMQ.Client;
using RabbitMQ.Client.Events;
using System.Text;

var factory = new ConnectionFactory { HostName = "localhost" };
using var connection = await factory.CreateConnectionAsync();
using var channel = await connection.CreateChannelAsync();

await channel.QueueDeclareAsync(queue: "hello", durable: false, exclusive: false, autoDelete: false,
    arguments: null);

Console.WriteLine(" [*] Waiting for messages.");

var consumer = new AsyncEventingBasicConsumer(channel);
consumer.ReceivedAsync += (model, ea) =>
{
    var body = ea.Body.ToArray();
    var message = Encoding.UTF8.GetString(body);
    Console.WriteLine($" [x] Received {message}");
    return Task.CompletedTask;
};

await channel.BasicConsumeAsync("hello", autoAck: true, consumer: consumer);

Console.WriteLine(" Press [enter] to exit.");
Console.ReadLine();
```

#### Program.cs (RabbitMQ-Send)

```csharp
using RabbitMQ.Client;
using System.Text;

var factory = new ConnectionFactory { HostName = "localhost" };
using var connection = await factory.CreateConnectionAsync();
using var channel = await connection.CreateChannelAsync();

await channel.QueueDeclareAsync(queue: "hello", durable: false, exclusive: false, autoDelete: false,
    arguments: null);

const string message = "Hello World!";
var body = Encoding.UTF8.GetBytes(message);

await channel.BasicPublishAsync(exchange: string.Empty, routingKey: "hello", body: body);
Console.WriteLine($" [x] Sent {message}");

Console.WriteLine(" Press [enter] to exit.");
Console.ReadLine();
```

---

## Ejecución

1. Inicia el proyecto **RabbitMQ-Receive** para escuchar mensajes.
2. Ejecuta el proyecto **RabbitMQ-Send** para enviar un mensaje.

### Resultado esperado
El terminal del receptor debe mostrar el mensaje:
```
 [x] Received Hello World!
```

