<p align="end">
   <strong>🌐 Change language:</strong><br>
   <a href="README.es.md">
    <img src="https://github.com/Nachopuerto95/multilang/blob/main/ES.png" alt="Español" width="50">
  </a>&nbsp;&nbsp;&nbsp;
  <a href="/README.md">
    <img src="https://github.com/Nachopuerto95/multilang/blob/main/EN.png" alt="English" width="50">
  </a>
</p>

# ft_irc (42cursus)

<img src="https://github.com/Nachopuerto95/multilang/blob/main/42-Madrid%20-%20Edited.jpg">

## 📜 About Project

> Write your own IRC server in C++98 and talk to it with a real IRC client.

```html
	🚀 This project is where sockets, non-blocking I/O and the IRC protocol
	meet. You build a TCP server that accepts multiple clients at once,
	authenticates them and relays messages between users and channels.
```

> [!NOTE]
> Because of 42 School norm requirements:
> * C++98 standard only (`-std=c++98`).
> * A single `poll()` (or equivalent) must handle every client — no `fork`, no threads per client.
> * Every I/O operation must be non-blocking.

### 📌 Description

`ircserv` is a lightweight IRC server loosely following RFC 1459. It listens on a TCP port protected by a password, lets users register with the classic `PASS / NICK / USER` handshake, and from there handles channels, private messages and basic operator commands.

It's built around a single `poll()` loop: one process, one thread, many sockets. Each client has its own receive buffer where raw bytes accumulate until a full IRC line (`\r\n`) can be parsed and dispatched.

### 🔧 How to Use

Compilation:

```bash
make
```

Execution:

```bash
./ircserv <port> <password>
```

Example:

```bash
./ircserv 6667 mypass
```

Any real IRC client works against it. With `nc`:

```bash
nc localhost 6667
PASS mypass
NICK nacho
USER nacho 0 * :Nacho Puerto
JOIN #general
PRIVMSG #general :Hello world
```

With `irssi`:

```bash
irssi -c localhost -p 6667 -w mypass
/nick nacho
/join #general
```

### 💬 Implemented commands

| Command | What it does |
|---|---|
| `PASS` | Set the server password (required before registration). |
| `NICK` | Pick or change a nickname. Must be unique. |
| `USER` | Send user identity. Closes the registration handshake. |
| `JOIN` | Enter a channel (creates it if it doesn't exist). |
| `PRIVMSG` | Send a message to a user or to a channel. |
| `KICK` | Operator kicks a user out of a channel. |
| `INVITE` | Invite a user into a channel. |
| `QUIT` | Disconnect cleanly from the server. |
| `HELP` | List the commands the server understands. |

> Full `MODE` flag coverage and `TOPIC` editing are **partial** — the server focuses on the command set required for evaluation.

### 🧠 How it works inside

```
 main ──► Server::setupListener()    // socket + bind + listen on <port>
      └─► Server::run()
            └─► poll(fds, n, -1)     // blocks until any fd is ready
                  ├─► listener ready → accept() new client (non-blocking)
                  └─► client ready   → recv into client.recv_buffer
                                      → split by "\r\n"
                                      → AuthMiddleware gates PASS/NICK/USER
                                      → dispatch command
                                      → queue replies into client.send_buffer
                                      → flush on POLLOUT
```

Highlights:

- **Non-blocking sockets** via `fcntl(fd, F_SETFL, O_NONBLOCK)`.
- **`SO_REUSEADDR`** on the listener so restarts don't need to wait for the port to free.
- **`AuthMiddleware`** enforces the strict `PASS → NICK → USER` order before any other command is accepted.
- **Per-client buffers** in both directions, because `recv` and `send` can always return partial lines.

### 📂 Project layout

```
ft_irc/
├── includes/                 # headers
├── src/
│   ├── main.cpp
│   ├── Server/
│   │   ├── Server.cpp        # poll loop, command dispatch
│   │   └── AuthMiddleware.cpp
│   ├── Client.cpp            # per-connection state
│   └── Channel.cpp           # channel + member list
├── Makefile
└── tester.sh                 # quick smoke test
```

### 📸 Demo

<!-- TODO: add screenshot of two clients chatting in a channel -->

### 👥 Team

This is a two-person project from the 42 cursus:

- **Nacho Puerto** — [@Nachopuerto95](https://github.com/Nachopuerto95)
- **Mayte del Pino** — [@madel-04](https://github.com/madel-04) / [@guacamoleconqueso](https://github.com/guacamoleconqueso)

### 📚 References

- [RFC 1459 — Internet Relay Chat Protocol](https://tools.ietf.org/html/rfc1459)
- [Modern IRC Documentation](https://modern.ircdocs.horse/)
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)
