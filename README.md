# NetBird Dockhand Template

A **type-3 (stack) template** for Dockhand's template library that deploys the full NetBird control plane: combined `netbird-server` (management + signal + relay + STUN + embedded Dex IdP), dashboard, Postgres, NetBird reverse-proxy, CrowdSec, and Caddy (with layer4 TLS-passthrough for exposed resources).


## Setup
1. **Add the source** in Dockhand → **Templates → Sources → Add source**: `https://raw.githubusercontent.com/shaban00/netbird-dockhand/main/templates.json` The **NetBird (Server)** tile appears in the library.

2. **Deploy** the tile, open the stack's **env** editor, fill in (domain, postgres creds, and the two secrets below).
   
   __AUTH_SECRET__
   ```bash
   openssl rand -base64 32 | sed 's/=//g'
   ```

   __DATASTORE_ENCRYPTION_KEY__
   ```bash
   openssl rand -base64 32
   ```

3. **Activate Proxy** — connect to the **`netbird-server`** container and create proxy token. In Dockhand: **Shell → `netbird-server`** then run:
   ```bash
   /go/bin/netbird-server token create --name "default-proxy" --config /etc/netbird/config.yaml
   ```

   Copy the `nbx_...` token and set it as the `NB_PROXY_TOKEN` env var. The token is issued by the management server, so it's the `netbird-server` container — the proxy then authenticates to the server with it.

4. **Activate CrowdSec** — connect to the **`netbird-crowdsec`** container and create an API key. In Dockhand: **Shell → `netbird-crowdsec`** then run:
   ```bash
   cscli bouncers add proxy -o raw
   ```

   Copy the key and set it as the `NB_PROXY_CROWDSEC_API_KEY` env var.

5. **Redeploy** the stack so the proxy picks up both new values.

### Updating environment variables
In Dockhand, open **Stacks → (your NetBird stack) → env**, **Add** or edit the variable, save, then **Redeploy** the stack. A plain container *restart* does not re-inject env — only a redeploy (`compose up`) applies new values.

### Ports
The server must be publicly accessible on TCP ports `80` and `443`, and UDP port `3478`
Port `51820` (UDP) is optional - WireGuard — NetBird reverse-proxy / peer-to-peer


### Client
Set `NB_SETUP_KEY` (**Dashboard > Settings > Setup Keys**) in that stack's `.env`; `NB_MANAGEMENT_URL` defaults to `https://api.netbird.io:443`. It runs in host network mode with `NET_ADMIN`/`SYS_ADMIN`/`SYS_RESOURCE` and `/dev/net/tun`.
