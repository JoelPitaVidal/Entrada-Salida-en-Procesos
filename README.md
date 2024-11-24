
# Entrada/Salida en Procesos

## Introducción

Cuando ejecutamos un subproceso en un sistema operativo, este no tiene, por defecto, un terminal o consola donde mostrar información. Toda la entrada/salida estándar es redirigida al proceso padre, que puede gestionar estos flujos para interactuar con el subproceso.

La mayoría de los sistemas operativos manejan tres flujos estándar de entrada/salida (E/S):

1. **Entrada estándar (`stdin`)**: Es el flujo desde el cual un programa recibe datos de entrada. Por defecto, está conectado al teclado.
2. **Salida estándar (`stdout`)**: Es el flujo donde un programa envía su salida normal. Por defecto, está conectado a la consola.
3. **Salida de error (`stderr`)**: Es un flujo separado donde se envían los mensajes de error. Aunque suele estar conectado a la consola, opera de manera independiente a `stdout`.

### Redirección de Flujos con Tuberías

En sistemas como Linux, los flujos de E/S pueden redirigirse entre procesos mediante tuberías (`pipes`). Java proporciona mecanismos similares para manejar estos flujos, permitiendo una comunicación eficiente entre procesos padre e hijo.

---

## Métodos Clave para Gestionar E/S

### `getInputStream()`

El método `getInputStream()` de la clase `Process` permite acceder a los datos enviados a la salida estándar del proceso hijo. Sin embargo, leer directamente de este flujo puede ser ineficiente.

Para mejorar el rendimiento, se recomienda usar `BufferedReader` junto con `InputStreamReader`. Esto permite:

- Convertir automáticamente bytes a caracteres.
- Usar un buffer para lecturas más rápidas.
- Utilizar métodos convenientes como `readLine()` para manejar texto.

#### Ejemplo:
```java
Process process = new ProcessBuilder("ls").start();
BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
String line;
while ((line = reader.readLine()) != null) {
    System.out.println(line);
}
```

#### Consideraciones sobre la Codificación
El sistema operativo puede usar distintas codificaciones (UTF-8, Windows-1252, etc.). Es crucial especificar la codificación correcta al usar `InputStreamReader` para evitar errores en la interpretación de los datos.

---

### `getErrorStream()`

El método `getErrorStream()` permite leer los mensajes de error generados por el proceso hijo. Si necesitas combinar la salida estándar y de error, puedes usar el método `redirectErrorStream(true)` del `ProcessBuilder`.

#### Ejemplo:
```java
ProcessBuilder builder = new ProcessBuilder("ls", "-la");
builder.redirectErrorStream(true);
Process process = builder.start();
BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
reader.lines().forEach(System.out::println);
```

---

### `getOutputStream()`

El método `getOutputStream()` permite enviar datos al proceso hijo. Para simplificar la escritura, se recomienda usar `PrintWriter`.

#### Ejemplo:
```java
Process process = new ProcessBuilder("sort").start();
try (PrintWriter writer = new PrintWriter(process.getOutputStream())) {
    writer.println("line1");
    writer.println("line3");
    writer.println("line2");
}
BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
reader.lines().forEach(System.out::println);
```

---

## Encadenamiento de Procesos con `startPipeline()`

En Java, puedes encadenar procesos utilizando el método `startPipeline()` de la clase `ProcessBuilder`. Esto conecta automáticamente la salida de un proceso con la entrada del siguiente.

#### Ejemplo:
```java
ProcessBuilder ls = new ProcessBuilder("ls");
ProcessBuilder grep = new ProcessBuilder("grep", ".java");
List<Process> processes = ProcessBuilder.startPipeline(List.of(ls, grep));
```

---

## Redirección de Flujos a Archivos

Con `ProcessBuilder`, puedes redirigir la salida estándar o de error a archivos. Esto es útil para guardar logs o resultados.

#### Ejemplo:
```java
File logFile = new File("output.log");
ProcessBuilder builder = new ProcessBuilder("java", "-version");
builder.redirectOutput(logFile);
builder.start();
```

Para agregar datos a un archivo sin sobrescribirlo, usa `Redirect.appendTo()`.

---

## Información sobre Procesos

La clase `ProcessHandle.Info` proporciona detalles sobre los procesos en ejecución, como:

- El comando ejecutado.
- Los argumentos usados.
- El tiempo de inicio.
- El tiempo de CPU utilizado.

#### Ejemplo:
```java
ProcessHandle handle = ProcessHandle.current();
ProcessHandle.Info info = handle.info();
System.out.println("Comando: " + info.command().orElse("Desconocido"));
System.out.println("Usuario: " + info.user().orElse("Desconocido"));
```

---

## Ejercicio Propuesto

**Crea un programa en Java que:**

1. Ejecute `dir` o `ls` según el sistema operativo.
2. Solicite parámetros al usuario.
3. Muestre el resultado del comando en pantalla.

#### Pista:
Usa `System.getProperty("os.name")` para identificar el sistema operativo y ajustar el comando.

---

## Referencias

- [Documentación de `Process`](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/lang/Process.html)
- [Documentación de `ProcessBuilder`](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/lang/ProcessBuilder.html)
