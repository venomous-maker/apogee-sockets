# apogee-sockets

A WebSocket server for real-time events in [Apogee](https://github.com/venomous-maker)
apps, speaking the Pusher Channels protocol (version 7). Browsers connect with
the usual clients (pusher-js, Laravel Echo); the app sends events through the
server's HTTP API, with [apogee-broadcast](https://github.com/venomous-maker/apogee-broadcast)'s
`SocketsBroadcaster` or any Pusher server library. Checked against the
official `pusher-js` client and `pusher` server library.

```typescript
import "std"
import "apogee-sockets"
use apogee.*

fn main() {
    let server = new SocketsServer(SocketsConfig.fromEnv())   // or new SocketsConfig([new SocketsApp(id, key, secret)])
    server.serve("0.0.0.0", 6001)
}
```

```js
// the browser
const pusher = new Pusher("app-key", { wsHost: "example.com", wsPort: 6001, forceTLS: false,
                                       enabledTransports: ["ws", "wss"], cluster: "mt1",
                                       authEndpoint: "/broadcasting/auth" })
pusher.subscribe("private-orders.7").bind("OrderShipped", (e) => show(e))
```

## In an apogee-framework app

Importing the module is all it takes: `SocketsPackage` (apogee-package) connects it when
the app is created. `sockets:serve` runs the server with the app's broadcasting credentials (`broadcasting.connections.sockets.*`, set by apogee-broadcast); `--host` and `--port` override the settings.

**Commands:** `sockets:serve`.

| Setting | Default |
|---|---|
| `sockets.host` | `SOCKETS_HOST`, `0.0.0.0` |
| `sockets.port` | `SOCKETS_PORT`, 6001 |

Your own configuration wins over these defaults.

## Channels

- **public** (`orders`): anyone may subscribe.
- **private** (`private-orders.7`): the app signs each subscription (its
  `/broadcasting/auth` endpoint answers with `key:HMAC(secret, socket_id:channel)`;
  apogee-broadcast's `Broadcast.auth` does this).
- **presence** (`presence-chat.1`): private, and everyone subscribed is told who
  is there (`subscription_succeeded` carries the members), who joins
  (`member_added`) and who leaves (`member_removed`). A user with several
  connections counts once.

Clients may send `client-` events on private and presence channels; they go to
the other subscribers (with `user_id` on presence channels), not back to the
sender.

## The HTTP API

Requests are signed as in the Pusher HTTP API (`auth_key`, `auth_timestamp`
within 600 seconds, `auth_version`, `body_md5`, `auth_signature`).

| | |
|---|---|
| `POST /apps/{id}/events` | `{"name", "data", "channels": [...] or "channel", "socket_id"?}`: `socket_id` is left out (the sender's own connection) |
| `POST /apps/{id}/batch_events` | `{"batch": [event, ...]}` |
| `GET /apps/{id}/channels` | occupied channels (`filter_by_prefix`, `info=user_count`) |
| `GET /apps/{id}/channels/{name}` | `occupied`, and with `info`: `subscription_count`, `user_count` |
| `GET /apps/{id}/channels/{name}/users` | a presence channel's users |
| `GET /up` | health check |

From Grav in the same process, `server.trigger(appId, channels, event, data, exceptSocket)`
does what `POST .../events` does.

## Configuration

`SocketsApp(id, key, secret)` with `maxConnections` (0: no limit) and
`clientEvents`; `SocketsConfig(apps)` with `activityTimeout` (seconds, told to
clients), `maxMessageBytes` and `allowedOrigins` (empty: any).
`SocketsConfig.fromEnv()` reads `SOCKETS_APP_ID`, `SOCKETS_APP_KEY`,
`SOCKETS_APP_SECRET` and `SOCKETS_ALLOWED_ORIGINS`.

Each connection has its own thread (the standard library's WebSocket server);
channels are shared under one lock, and messages are sent outside it. For TLS,
put it behind a proxy that terminates TLS, or serve it with the standard
library's `HttpServer.listenTls`.

## Tests

```bash
grav test tests/sockets.grav
grav test tests/package.grav
```
