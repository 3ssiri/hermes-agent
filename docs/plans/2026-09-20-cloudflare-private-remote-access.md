# Cloudflare private remote access plan

Date: 2026-09-20
Status: proposal / implementation task
Scope: remote access only; no runtime behavior changed by this document.

## Goal

Allow the owner to reach Hermes from iPhone/iPad outside the home network without exposing terminal, RPC, or device-control surfaces directly to the public Internet.

## Default architecture

Because Hermes can execute commands on the host, the default transport should be a **Cloudflare Private Network** reached through the Cloudflare One client, not a public hostname.

```
iPhone / iPad
   ↓ Cloudflare One client
Cloudflare Zero Trust / Private Network
   ↓
cloudflared on Windows
   ↓
Hermes Dashboard / explicitly allowed local services
```

A public hostname behind Cloudflare Access may be considered later for a narrowly scoped low-risk UI, but it is not the default for this task.

## Implementation constraints

1. Discover the actual Hermes dashboard/gateway ports on the target Windows host first. Do not assume a port.
2. Expose only the Dashboard in the first pass.
3. Run `cloudflared` as an independently managed Windows service.
4. Restrict access to the owner's identity/devices.
5. Do not expose terminal backends, RPC, shell-execution endpoints, or broad device-control APIs publicly.
6. Keep Cloudflare credentials/tokens outside the repository and outside Hermes memory.
7. Preserve normal local access when Cloudflare is unavailable.
8. Do not introduce a generic service registry or new abstraction unless the implementation actually requires it.
9. Do not change provider/model/gateway architecture as part of this work.

## Local-service relationship

Hermes may later act as a local client/operator for services such as:

- MyVault
- marsad-subscriptions
- the personal memory project ("ذاكرتي")
- Madarek when needed

This plan does not republish those services and does not weaken their own authentication. Each service owns its own exposure policy.

## Security invariants

- No router port forwarding.
- No permanent Quick Tunnel.
- No public terminal.
- No HTTP endpoint that executes arbitrary shell commands from the public Internet.
- No auth bypass.
- No Cloudflare service tokens committed to Git.
- Use the narrowest route and least privilege possible.
- Any future execution endpoint requires a separate threat review and explicit authentication/authorization.

## Minimal observability

Add or document a safe status check that can confirm, without printing credentials:

- tunnel up/down;
- dashboard reachable locally;
- private route reachable.

Avoid logging tokens, cookies, Access assertions, or secrets.

## Verification

Verify on the real target host:

1. Hermes Dashboard is reachable from iPhone over cellular through Cloudflare One.
2. An unauthorized device/user cannot reach it.
3. Local Dashboard access still works with Cloudflare stopped.
4. Stopping `cloudflared` only removes remote access and does not stop Hermes.
5. No terminal/RPC execution surface is configured as a public hostname.
6. Logs contain no Cloudflare credentials.
7. Run relevant tests/checks and inspect the final diff.

## Out of scope

- A native Hermes mobile app.
- Reworking the Hermes gateway.
- Full Windows remote control over HTTP.
- Turning Hermes into a VPN manager.
- Automatically publishing every local service.

## Definition of done

The owner can reach the Hermes Dashboard from an authorized mobile device outside the home network through a private Cloudflare route, with no public execution surface, no inbound router ports, and documented start/stop/recovery steps.
