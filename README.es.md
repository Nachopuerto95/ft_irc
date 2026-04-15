<p align="end">
   <strong>🌐 Cambiar idioma:</strong><br>
   <a href="/README.es.md">
    <img src="https://github.com/Nachopuerto95/multilang/blob/main/ES.png" alt="Español" width="50">
  </a>&nbsp;&nbsp;&nbsp;
  <a href="/README.md">
    <img src="https://github.com/Nachopuerto95/multilang/blob/main/EN.png" alt="English" width="50">
  </a>
</p>

# ft_irc (42cursus)

<img src="https://github.com/Nachopuerto95/multilang/blob/main/42-Madrid%20-%20Edited.jpg">

## 📜 Sobre el proyecto

> Escribir tu propio servidor IRC en C++98 y hablar con él usando un cliente IRC real.

```html
	🚀 En este proyecto se juntan sockets, I/O no bloqueante y el protocolo
	IRC. Montas un servidor TCP que acepta varios clientes a la vez, los
	autentica y reenvía mensajes entre usuarios y canales.
```

> [!NOTE]
> Por las normas de la escuela 42:
> * Solo estándar C++98 (`-std=c++98`).
> * Todos los clientes se manejan con un único `poll()` (o equivalente). Nada de `fork` ni hilos por cliente.
> * Toda la I/O tiene que ser no bloqueante.

### 📌 Descripción

`ircserv` es un servidor IRC ligero que sigue a grandes rasgos la RFC 1459. Escucha en un puerto TCP protegido con una contraseña, deja a los usuarios registrarse con el handshake clásico `PASS / NICK / USER`, y a partir de ahí gestiona canales, mensajes privados y comandos básicos de operador.

Por dentro gira sobre un único bucle de `poll()`: un proceso, un hilo, muchos sockets. Cada cliente tiene su buffer de recepción donde los bytes se van acumulando hasta que aparece una línea IRC completa (`\r\n`) lista para parsear.

### 🔧 Cómo usarlo

Compilar:

```bash
make
```

Ejecutar:

```bash
./ircserv <puerto> <contraseña>
```

Ejemplo:

```bash
./ircserv 6667 mypass
```

Funciona con cualquier cliente IRC real. Con `nc`:

```bash
nc localhost 6667
PASS mypass
NICK nacho
USER nacho 0 * :Nacho Puerto
JOIN #general
PRIVMSG #general :Hello world
```

Con `irssi`:

```bash
irssi -c localhost -p 6667 -w mypass
/nick nacho
/join #general
```

### 💬 Comandos implementados

| Comando | Qué hace |
|---|---|
| `PASS` | Fija la contraseña del servidor (obligatorio antes del registro). |
| `NICK` | Elige o cambia de nick. Tiene que ser único. |
| `USER` | Envía la identidad del usuario. Cierra el handshake de registro. |
| `JOIN` | Entra en un canal (lo crea si no existe). |
| `PRIVMSG` | Manda un mensaje a un usuario o a un canal. |
| `KICK` | El operador echa a un usuario de un canal. |
| `INVITE` | Invita a un usuario a un canal. |
| `QUIT` | Desconectar limpiamente del servidor. |
| `HELP` | Lista los comandos que entiende el servidor. |

> La cobertura completa de flags `MODE` y la edición de `TOPIC` están **parciales** — el servidor se centra en el conjunto de comandos exigido para la evaluación.

### 🧠 Cómo funciona por dentro

```
 main ──► Server::setupListener()    // socket + bind + listen en <puerto>
      └─► Server::run()
            └─► poll(fds, n, -1)     // espera a que cualquier fd esté listo
                  ├─► listener listo → accept() nuevo cliente (no bloqueante)
                  └─► cliente listo  → recv al client.recv_buffer
                                      → partir por "\r\n"
                                      → AuthMiddleware filtra PASS/NICK/USER
                                      → despachar comando
                                      → encolar respuestas en client.send_buffer
                                      → flush en POLLOUT
```

Detalles:

- **Sockets no bloqueantes** con `fcntl(fd, F_SETFL, O_NONBLOCK)`.
- **`SO_REUSEADDR`** en el listener para poder reiniciar el servidor sin esperar a que se libere el puerto.
- **`AuthMiddleware`** fuerza el orden estricto `PASS → NICK → USER` antes de aceptar cualquier otro comando.
- **Buffers por cliente** en ambos sentidos, porque `recv` y `send` pueden devolver líneas a medias.

### 📂 Estructura

```
ft_irc/
├── includes/                 # cabeceras
├── src/
│   ├── main.cpp
│   ├── Server/
│   │   ├── Server.cpp        # loop de poll, dispatch de comandos
│   │   └── AuthMiddleware.cpp
│   ├── Client.cpp            # estado por conexión
│   └── Channel.cpp           # canal + lista de miembros
├── Makefile
└── tester.sh                 # smoke test rápido
```

### 📸 Demo

<!-- TODO: añadir captura de dos clientes hablando en un canal -->

### 👥 Equipo

Es un proyecto en pareja del cursus de 42:

- **Nacho Puerto** — [@Nachopuerto95](https://github.com/Nachopuerto95)
- **Mayte del Pino** — [@madel-04](https://github.com/madel-04) / [@guacamoleconqueso](https://github.com/guacamoleconqueso)

### 📚 Referencias

- [RFC 1459 — Internet Relay Chat Protocol](https://tools.ietf.org/html/rfc1459)
- [Modern IRC Documentation](https://modern.ircdocs.horse/)
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)
