# SyncGate Lite

**The whole gateway on one box, in one command.** A Raspberry Pi, a mini-PC, an old
workstation — no cloud, no cluster, no identity provider, nothing to hand-configure before it
runs.

```bash
curl -O https://raw.githubusercontent.com/benwold-lgtm/SyncGate-Lite/main/docker-compose.yml
docker compose up -d
docker compose logs device-mcp-ui-bff | grep -A6 first-run   # your admin login
```

Then open `http://localhost:8080`. Three containers, ~500 MB, amd64 or arm64.

---

## What it is

Lite gives an LLM client — Claude Desktop, Cursor, anything speaking MCP — a safe, audited way
to reach the devices on your network. It is the same SyncGate images the full deployment runs,
in their simplest posture:

- **Embedded mode.** Device pods live in the gateway process, backed by SQLite. No Redis, no
  separate worker, no shared state to corrupt.
- **Local users only.** One password login. No SSO to stand up.
- **Self-provisioning secrets.** The UI password, session secret and gateway API key are all
  generated on first boot and printed once.

```
  Browser ──▶ web (:8080) ──▶ BFF ──▶ gateway (:8000, embedded) ──▶ your devices (LAN)
                                          ▲
                          MCP / LLM clients (Claude Desktop, Cursor, …)
```

## What it deliberately is not

These are design decisions, not a roadmap. Lite is a **single-operator appliance**, and each of
these follows from that:

| Not here | Why |
|---|---|
| Backup & restore in the console | Lite is local-user-only. A shared password is not an identity, and an archive is a credential-bearing export — so the console does not offer one. |
| SSO / OIDC | There is one operator. An IdP would be more machinery than the thing it protects. |
| Multi-tenancy, a device catalog, delegated support | Those exist to let one party operate a fleet on another party's behalf. Lite has one party. |
| Horizontal scale, HA, replicas | Embedded mode is one process by definition. That is the trade that buys "no Redis". |

If you need any of them, you want the full SyncGate deployment, not a bigger Lite.

## When to use the full deployment instead

Lite stops being the right answer when you need more than one operator, more than one gateway
replica, or an audit trail tied to real identities. The full deployment runs the same images in
distributed mode — Redis, separate workers, OIDC — under Docker Compose or Kubernetes.

## Documentation

**[docs/deploy.md](docs/deploy.md)** — the full walkthrough: requirements, first-run
credentials, connecting an MCP client, registering a device, self-signed device certificates,
hardening before you expose it beyond localhost, and rotating secrets.

## Where the images come from

Lite ships no source of its own. It is a deployment of the SyncGate images, built and published
from the gateway and UI repositories. This repo owns the compose file, the defaults, and the
docs — which is what makes it a separate product rather than a profile.

## Before you expose it beyond localhost

The defaults assume a trusted LAN: plain HTTP, and the SSRF guard allowed to reach private
addresses (`MCP_ALLOW_PRIVATE_TARGETS=true`) because home devices live on `192.168.x` and
`10.x`. Put TLS in front of it and read
[the hardening section](docs/deploy.md#before-you-expose-it-beyond-localhost) before it is
reachable from anywhere you do not control.

## License

PolyForm Noncommercial 1.0.0 — see [LICENSE](LICENSE).
