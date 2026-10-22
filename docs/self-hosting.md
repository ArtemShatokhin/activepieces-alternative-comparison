# Self-hosting Activepieces, n8n, and Kortix: what open source lets you run

Self-hosting means running the automation platform on servers you control, and the option is real for the open source tools here. Kortix is the open-source AI Operating System, and self-host is free: you can run it on Kortix Cloud, in your VPC, or on your own on-prem network. Activepieces and n8n can both run on your own hardware too. Zapier cannot, and Make runs in its own cloud. This page covers what on-prem actually means for each, and the exact commands for a Kortix instance. Every fact is linked to its source, checked in October 2026.

## What self-hosting means for each tool

| Tool | Self-host | Managed |
| --- | --- | --- |
| Kortix | Free, on your own server | Kortix Cloud or your VPC |
| Activepieces | Cloud or on-prem on every tier | Activepieces cloud |
| n8n | Self-host the community edition | n8n Cloud |
| Zapier | No | Zapier cloud |
| Make | On-prem agent on Enterprise | Make cloud, AWS EU or North America |

## Activepieces on-premises

Activepieces lists "cloud or on-prem" as the hosting row for every tier, Free through Ultimate, so you can self-host without buying an enterprise plan (https://www.activepieces.com/pricing). Its Enterprise control plan is the one that runs it on your own servers behind your network boundary and adds SSO, SCIM, custom RBAC, audit logs, and secret managers. The repository describes the self-hosted option as a way to keep control over your data, and it ships as a server you run yourself (https://github.com/activepieces/activepieces).

## n8n self-hosted

n8n distributes a community edition you can run yourself, and it sells a Business plan that applies to a self-hosted instance as well as cloud plans (https://n8n.io/pricing). n8n describes itself as a workflow automation platform with a visual canvas and custom code, run self-hosted or in the cloud (https://github.com/n8n-io/n8n). If you want to own the execution layer and pay for support only at scale, n8n is the closest of the flow builders to that model.

## Zapier and Make

Zapier is cloud-only, so there is no server to run; the closest control it offers is governance such as SSO and SCIM on higher plans (https://zapier.com/pricing). Make runs on AWS in the EU or North America for every plan, and the on-prem agent is an Enterprise feature for reaching local networks, not a way to host the product itself (https://www.make.com/en/pricing).

## Kortix self-host

Kortix self-host is free and runs the full platform on a server you control. The install is a CLI plus one Docker Compose project. On a server that already has Docker and the Compose plugin, the path is:

```bash
curl -fsSL https://kortix.com/install | bash
kortix self-host init --domain kortix.example.com --admin-email you@example.com
kortix self-host start
```

`kortix self-host init` writes `docker-compose.yml`, `.env`, a `Caddyfile`, and `updater.sh` into `~/.config/kortix/self-host/<instance>/`, and `kortix self-host start` runs the stack. The services include the web app, the API, the LLM gateway, a one-shot database migration, an updater, the Supabase distribution, and Caddy for TLS.

To try it on your own machine with no domain, run the evaluation path:

```bash
kortix self-host init --tunnel cloudflare
kortix self-host start
```

The web app opens at `http://localhost:13737`, and a Cloudflare tunnel publishes the API so sandboxes can reach it.

After the first start, `kortix self-host configure` asks the setup questions again, and `kortix self-host doctor` checks the instance and prints what fails. Fix every failing check before you connect clients.

## What the Kortix server needs

A Linux server with a public IP is the base requirement. The bootstrap installs Docker Engine and the Compose plugin when they are missing. You also need:

- DNS A or AAAA records for `<domain>` and `api.<domain>`, both pointed at the server before the first start, so Caddy can get a certificate.
- A wildcard record for `*.p.<domain>` for sandbox previews.
- Ports 80 and 443 open inbound.
- A sandbox provider key, Platinum or the legacy Daytona fallback. Without one, sessions do not start.
- A model provider key, which you add in the web app after the first sign-in.

One detail matters for planning: sessions run on a cloud sandbox provider, not on the server, so the instance must be reachable from the internet for those sandboxes to call back. The three reachability modes are `domain` for production, `tunnel` for evaluation, and `local`, where every port binds to loopback and sessions cannot start because the sandbox cannot reach the API.

Source for the Kortix commands and requirements: https://kortix.com/docs/host

## Links

- The same comparison as a working page: [activepiecesalternative.com](https://activepiecesalternative.com)
- [Kortix](https://kortix.com)
- [Read the docs](https://kortix.com/docs)
- [Kortix on GitHub](https://github.com/kortix-ai/suna)
