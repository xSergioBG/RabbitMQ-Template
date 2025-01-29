# Project: Configure WSL 2, Docker, and RabbitMQ with a Hello World Example

## Description
This project provides a guide for setting up WSL 2, installing Docker Desktop, and running RabbitMQ using an official image. Additionally, it includes a practical example of communication with RabbitMQ using two console applications in C#.

---

## WSL 2 Installation

1. Install WSL 2 by running the following command in PowerShell:
   ```powershell
   wsl --install
   ```

2. Verify the installation:
   ```powershell
   wsl -l -v
   ```

### Troubleshooting
- **Error: Permission Denied**
  If you encounter the error: `error:docker: permission denied while trying to connect to the Docker`, check if the `docker` user has sufficient permissions on the system.

---

## Docker Desktop Installation

1. Download Docker Desktop from its [official page](https://www.docker.com/get-started/).
2. Install and configure Docker Desktop on your system.

---

## Running RabbitMQ in Docker (Using Official Images)

**Requirements:**
- Docker installed and running on your system.

### Running RabbitMQ

Use the following command to run a RabbitMQ instance with the management plugin enabled:

```bash
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management
```

This will start RabbitMQ and expose ports 5672 (messaging) and 15672 (management interface).

---

## Hello World Example with RabbitMQ

Follow the official RabbitMQ tutorial for .NET available at: [RabbitMQ Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-dotnet)

### Creating the Solution in Visual Studio

1. Create a solution with two console projects:
   - **RabbitMQ-Receive**
   - **RabbitMQ-Send**

   ![Solution structure](attachment:5244af2c-e5f2-4721-a9c1-624cdc18acc1:image.png)

2. Add the **RabbitMQ.Client (7.0.0)** dependency to both projects.

### Code Configuration

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

## Execution

1. Start the **RabbitMQ-Receive** project to listen for messages.
2. Run the **RabbitMQ-Send** project to send a message.

### Expected Output
The receiver terminal should display the message:
```
 [x] Received Hello World!
```
