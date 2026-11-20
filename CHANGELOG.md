# Changelog

Every release of apogee-sockets, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

The first release. Needs grav 1.10.0 or newer.

- `SocketsServer`: the Pusher Channels protocol (7) over WebSockets: public,
  private and presence channels, subscription signatures, client events,
  pings, connection quotas, allowed origins.
- The signed HTTP API: triggering events (one, a batch, leaving out a socket)
  and asking about channels and presence users; a health check.
- Works with the official pusher-js client and the pusher server library.
