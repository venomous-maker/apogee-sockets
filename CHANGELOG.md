# Changelog

Every release of apogee-sockets, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

### Fixes

- A Redis connection shared by several threads (the cache, session and queue-dashboard stores, used by a multi-threaded server or queue worker) no longer mixes up replies or stalls: every command is one locked round trip (grav-redis 0.2.2). Uses the new releases of apogee-console, apogee-package.

## 0.2.1 (2026-11-27)

### Fixes

- Database connections that the server dropped while idle (MySQL's `wait_timeout`, a restart, a proxy's idle limit) reconnect on their next use instead of failing with "server has gone away" (grav-db 0.2.1). Uses the new releases of apogee-console, apogee-package.

## 0.2.0 (2026-11-25)

Needs grav 1.12.0 or newer.

- `SocketsPackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring and commands moved here from the framework.

### Dependencies

- Uses apogee-package 0.1.0, apogee-console 0.2.2 (from their GitHub tags).

## 0.1.1 (2026-11-21)

### Performance

- An event is encoded once per channel instead of once per recipient: fanning
  one out to 100 subscribers takes 43% fewer instructions, and delivery went
  from about 84,000 to about 104,000 messages a second.
- Each app's channels have their own lock, so busy apps do not wait on each
  other.

## 0.1.0 (2026-11-20)

The first release. Needs grav 1.10.0 or newer.

- `SocketsServer`: the Pusher Channels protocol (7) over WebSockets: public,
  private and presence channels, subscription signatures, client events,
  pings, connection quotas, allowed origins.
- The signed HTTP API: triggering events (one, a batch, leaving out a socket)
  and asking about channels and presence users; a health check.
- Works with the official pusher-js client and the pusher server library.
