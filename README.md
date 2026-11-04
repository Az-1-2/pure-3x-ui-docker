# pure-3x-ui-docker

> **This is a fork.** The project is a derivative work of [pure-3x-ui](https://github.com/vermiliondevil-web/pure-3x-ui/) (author — [vermiliondevil-web](https://github.com/vermiliondevil-web)) Which in of itself is a fork of [3x-Ui](https://github.com/MHSanaei/3x-ui) (author — [MHSanaei](https://github.com/MHSanaei)) and is distributed under the **AGPL-3.0** license. All changes made in this fork are published under the same license.

A clean build of [3x-ui](https://github.com/MHSanaei/3x-ui) without ads or donation blocks, and a build of [pure-3x-ui](https://github.com/vermiliondevil-web/pure-3x-ui/) with Docker images published for containerised environments. 

### Current state

- **Fork:** [Packages](https://github.com/Az-1-2/pure-3x-ui-docker/pkgs/container/pure-3x-ui)
- **Upstream:** [Packages](https://github.com/MHSanaei/3x-ui/pkgs/container/3x-ui)
- **Sync history:** [docs/upstream-changelog.md](docs/upstream-changelog.md)

## Policy on sponsor commits

This fork **deliberately removes** upstream's sponsor infrastructure: `sponsor.go`, `SponsorSlot`, `SponsorsPage`, `useSponsorsQuery`, donation buttons, the `/sponsors` endpoint. The reason is privacy: upstream code contacts `sponsors.sanaei.dev` on every login page load (when the cache expires), exposing the server IP and SNI to a third party with no way to disable it. For operators in censored countries, this lets DPI identify the panel as 3x-ui.

**What this means for upstream commits:**

- Sponsor commits (new slots, new endpoints, changes to `sponsor.go` or the sponsor frontend components) are **not accepted in this fork as-is**. They are **reworked** via `scripts/remove-ads.sh`: sponsor code is stripped, `sponsor.go` is replaced with a stub, endpoints are removed.
- This is **not** hostility toward upstream. The [3x-ui](https://github.com/MHSanaei/3x-ui) project is an excellent panel, and we are grateful to the author and contributors. But this fork exists **specifically** to remove sponsor tracking. If upstream adds new sponsor features, the fork will strip them, not inherit them.
- If you want your sponsor feature to **stay** in the panel — use [upstream](https://github.com/MHSanaei/3x-ui). It works there as intended. **Here — NO! AND IT NEVER WILL!**

## Upstream Release Tracking
Unlike [pure-3x-ui](https://github.com/vermiliondevil-web/pure-3x-ui/) itself, this fork ****does update on every upstream release****. 

## Who this fork is for

This fork is **for personal use** for those who prefer or must use containerised environments to run applications over binaries on host. The author not sell keys, does not provide access services, and does not serve clients.

**If you sell access** — this fork is probably **not for you**. Use [upstream](https://github.com/MHSanaei/3x-ui) — it has all the features for large setups, and the author supports them.

**If you have 3–10 users** (family, friends, personal server) — this fork is for you: no ads, core preservation, rare updates.
## Support

This fork **does not accept donations** and contains no ads. The project is distributed as-is, for personal and non-commercial use.

If you want to support **upstream development** — please refer to the [original repository](https://github.com/MHSanaei/3x-ui).

## About the project

This is a fork of the **pure-3x-ui** panel by [vermiliondevil-web](https://github.com/vermiliondevil-web) which is a fork of **3x-ui** by [MHSanaei](https://github.com/MHSanaei). This fork exists to offer a way to run pure-3x-ui in a containerised environment rather than binaries. 

## Installation

### Docker Compose (recommended)

1. Create docker-compose.yml
```
services:
  3xui:
    image: ghcr.io/az-1-2/pure-3x-ui:latest
    container_name: 3x-ui
    # hostname: yourhostname <- optional
    # Optional hard memory cap. When set, the panel derives its Go soft limit
    # (GOMEMLIMIT, ~90% of this cap) so it GCs before the OOM killer fires.
    # mem_limit: 512m
    # AmneziaWG works in this image: it runs embedded in the panel process
    # (amneziawg-go over a gVisor userspace netstack), so it needs no kernel
    # module and no host tooling. Publish its UDP listen port to use it.
    volumes:
      - $PWD/db/:/etc/x-ui/
      - $PWD/cert/:/root/cert/
      - $PWD/acme/:/root/.acme.sh/
    environment:
      XRAY_VMESS_AEAD_FORCED: "false"
      XUI_ENABLE_FAIL2BAN: "true"
      XUI_GOGC: "50"
      XUI_MEMORY_RELEASE_INTERVAL: "5"
      XUI_MEMORY_LIMIT: "256"
      XUI_LOG_LEVEL: "warning"
      XUI_DEBUG: "false"
      XUI_PORT: "2053"
      # XUI_MEMORY_LIMIT: "400"      # in MiB
      # GOMEMLIMIT: "400MiB"         # Go syntax, takes precedence
      # XUI_PPROF: "true"           # expose pprof on 127.0.0.1:6060 for profiling
      # XUI_INIT_WEB_BASE_PATH: "/"
    tty: true
    ports:
      - "2053:2053"
      - "2096:2096"
      - "443:443"
    restart: unless-stopped
```
2 Run docker compose up -d to create the container, then go to the panel to begin setup. The panel may be on a local ip, regular ip, or domain on port 2053 depending on your setup. 

**Notes**: You may import an existing sqlite file in the panel if you wish. Postgres may be supported, but is untested and not recommended. 

### Docker Run 

1. Run 
```
docker run --name 3x-ui -v $PWD/db/:/etc/x-ui/ -v $PWD/cert/:/root/cert/ -v $PWD/acme/:/root/.acme.sh/ -e XRAY_VMESS_AEAD_FORCED=false -e XUI_ENABLE_FAIL2BAN=true -e XUI_GOGC=50 -e XUI_MEMORY_RELEASE_INTERVAL=5 -e XUI_MEMORY_LIMIT=256 -e XUI_LOG_LEVEL=warning -e XUI_DEBUG=false -e XUI_PORT=2053 -t -p 2053:2053 -p 2096:2096 -p 443:443 --restart unless-stopped ghcr.io/az-1-2/pure-3x-ui:latest
```

**Notes**: This is recommended for testing only, for a regular setup, it is highly recommended to use compose. 

### Convert an existing 3x-ui install
If you already have 3x-ui running in Docker, you can simply use ghcr.io/az-1-2/pure-3x-ui:latest as a drop in replacement image without any modifications and recreate your container. If you run 3x-ui as a binary (or installed via install.sh), then you will need to backup your sqlite file and follow the compose instructions above, then reimport the sqlite file when installed. 

## Automation

The repository has **two GitHub Actions workflows**:

### 1. `Sync Upstream` — auto-sync with the original

**Runs in ~17 seconds.** Once a day (at 03:00 UTC) or manually:

1. Pulls fresh code from [MHSanaei/3x-ui](https://github.com/MHSanaei/3x-ui).
2. **Takes locales and `update.sh` from upstream directly** — to avoid conflicts.
3. **Commits** those files.
4. Merges the rest with `-X theirs`.
5. Runs `scripts/remove-ads.sh` — strips ads and replaces `sponsor.go` with a stub.
6. Commits the changes and pushes to `main`.

**This means:** when Sanaei releases an update, ads are stripped **automatically**, and there are **no conflicts**.

### 2. `Build and Release` — build and publish releases

Triggered by a `v*` tag or manually:

1. **`Set version from tag`** — extracts the version from the tag and writes it to `internal/config/version`.
2. Strips ads (`scripts/remove-ads.sh`) — including replacing `sponsor.go` with a stub.
3. Builds the frontend (`npm run build`).
4. Builds the Go binary.
5. Downloads the Xray core from XTLS.
6. Packages everything into `x-ui-linux-amd64.tar.gz` + `.sha256`.
7. Publishes the release.

### How to publish a new version

1. **Create a tag** with the new number:

   ```bash
   cd /workspaces/pure-3x-ui
   git pull
   git tag v3.9.23
   git push origin v3.9.23
   ```
2. **GitHub Actions** builds release `v3.9.23` automatically.

3. The **"Update" button** becomes active.

4. **Click it** → the panel updates.

### When to publish a new release

- ✅ Sanaei released a **new stable release**.
- ✅ **Important bug fixes** have accumulated.
- ✅ You made **significant changes**.
- ❌ **Do not** publish a new release for **every commit** from Sanaei.

**Rule:** the version in `internal/config/version` **must match** the tag.
---

## Upstream synchronization

This fork **does not chase** every upstream release. Code is pulled automatically via the `Sync Upstream` workflow, but fork releases are rare — only after testing on a test environment.

### How it works

1. **Automatically** (once a day, 03:00 UTC): `Sync Upstream` pulls new upstream code into `main`.
2. **`remove-ads.sh`** strips ads/sponsors and patches `update.sh` (Xray core preservation).
3. **No fork release is published** — `main` updates, but no `v*` tag is created.
4. **When enough changes accumulate** — manual testing on a separate VPS begins.
5. **After a successful test** — a tag is created, `Build and Release` builds the release, production is updated.

### What this means

- `main` is fresh upstream code with our patches, but **not a release**.
- `v*` tags are verified releases for production.
- Production does not update on every upstream release.

### Why we don't chase every release

- Upstream releases **often** (v3.9.x → v3.10.0 in a month).
- Fresh releases **may contain bugs** — fixes come in v3.10.1, v3.10.2.
- **systemd sandbox** (v3.10.0) is a major change and may conflict with custom configurations.
- **Production needs stability**, not freshness.

## Code review before release

This fork is **not** a simple copy of upstream. Every release is checked for personal data leaks, telemetry, and hidden network requests.

### Why this matters

The original 3x-ui is developed by a **community of 310+ contributors**. Any PR can add:

- Telemetry (sending statistics to third-party servers).
- Sponsor requests (IP and SNI leak).
- Environment scanning (checking `.env`, `.git/config`).
- New dependencies with unknown behavior.

**The upstream author does not check every PR** for privacy. His priority is functionality and monetization.

### What is checked in this fork

| What is checked | How |
|---|---|
| Sponsor code | `scripts/remove-ads.sh` replaces `sponsor.go` with a stub |
| Ad links | Removed from `AppSidebar`, `LoginPage`, `routes` |
| Telemetry | Checked via the binary's `strings` |
| Hidden domains | `grep` over the code for `http://` and `https://` |
| New dependencies | Manual review of `go.mod` and `package.json` |
### How to verify yourself

```bash
# 1. Verify that sponsor code is removed
grep -rn "sponsors.sanaei.dev" internal/ frontend/src/ 2>/dev/null

# 2. Check the binary for suspicious domains
strings /usr/local/x-ui/x-ui | grep -iE "sponsor|sanaei|telemetry" | head

# 3. Monitor outbound connections
tcpdump -i any -n 'not host localhost' -c 20
```
### What this means for the user

- ✅ **Relevant updates** — pulled via `Sync Upstream`.
- ✅ **Leaks** — stripped automatically via `remove-ads.sh`.
- ✅ **New features** — reviewed before release.
- ⚠️ **Not a 100% guarantee** — you can verify the code yourself.
---

## Security

### CVE-2026-55477: Arbitrary file write by an authenticated admin

**What it is:** In 3x-ui versions **before 3.3.1**, an authenticated admin could, via a database import or the Xray config editor, achieve **arbitrary file write** on the server. By manipulating the `xrayTemplateConfig.log.access` value, an attacker could redirect the Xray access log to any file on the filesystem — including `~/.ssh/authorized_keys`.

**Impact:** If Xray runs as root — **full server takeover**.

**CVSS score:** 7.2 (High).

**Is this fork affected?** **No.** This fork is based on **v3.9.22**, which is well above the fixed version 3.3.1.
### `sponsors.sanaei.dev` leak removed

**What it was:** `internal/web/service/panel/sponsor.go` contained hardcoded URLs:
https://sponsors.sanaei.dev/3X/sponsors.json
https://sponsors.sanaei.dev/3X/logos/

On every login page load, the panel sent a request to `sponsors.sanaei.dev` — exposing the **server IP** and **SNI** to a third party. For operators in censored countries, this let DPI identify the panel as 3x-ui.
**What was done:**

- `sponsor.go` replaced with a **stub** (functions return empty values, no network calls).
- `scripts/remove-ads.sh` **automatically** replaces `sponsor.go` with a stub after every `Sync Upstream`.

**Check:**

```bash
strings /usr/local/x-ui/x-ui | grep -c "sponsors.sanaei.dev"
```
# Expected: 0

### Other security measures (inherited)

- **Per-client IP limiting** with fail2ban integration.
- **API tokens with granular scopes** (SHA-256 hashes).
- **TOTP two-factor authentication**.
- **CSRF protection, login rate limiting, SSRF protection**.
- **Path traversal protection** (verified by `TestGeodataRejectsBadRequests`).
### What remains (inherited, not fixed)

- **Client IPs are stored in plaintext** in the SQLite database (`inbound_client_ips`, `node_client_ip`).
- **The panel scans its own environment** at startup.

### Recommendations for operators

1. **Upgrade to v3.3.1+** (this fork is already above).
2. **Change the admin credentials** immediately after installation.
3. **Restrict panel access** to trusted IPs.
4. **Run Xray with minimal privileges**.
5. **Enable fail2ban**.
6. **Make regular backups** of the database.
---

## Origin and license

### This is a fork

`pure-3x-ui` is a derivative work (fork) of [3x-ui](https://github.com/MHSanaei/3x-ui).

- The project **is not** original development.
- Based on [MHSanaei/3x-ui](https://github.com/MHSanaei/3x-ui) code.
- All changes are a layer on top of the original.
- The original author **bears no responsibility** for this fork.
### AGPL-3.0 license

The project is distributed under the **GNU Affero General Public License v3.0** (AGPL-3.0).

**What this means:**

- ✅ You can use, modify, distribute.
- ✅ You can fork.
- ⚠️ You must keep the same license (AGPL-3.0).
- ⚠️ You must credit the original author.
- ⚠️ If you provide network access to the panel, you must provide the source code.
- ⚠️ Closed forks are not allowed.
### Links

- **This fork:** https://github.com/vermiliondevil-web/pure-3x-ui
- **Original:** https://github.com/MHSanaei/3x-ui
- **License text:** [LICENSE](LICENSE)
---

## Acknowledgements

- [MHSanaei](https://github.com/MHSanaei) — for the original 3x-ui
- [XTLS/Xray-core](https://github.com/XTLS/Xray-core) — for the Xray core
- [Acme.sh](https://github.com/acmesh-official/acme.sh) — for SSL certificates
- [@Az-1-2](https://github.com/Az-1-2) — for the idea to strip `SponsorsPage` from `panelRoutes.tsx` (block `4.1b` in `remove-ads.sh`).
- All contributors to the original project
---

## Feedback

- Issues: https://github.com/vermiliondevil-web/pure-3x-ui/issues
- Pull Requests: welcome
---

## Disclaimer

This is an **unofficial fork**. The project is not affiliated with MHSanaei or XTLS.

The panel is provided as-is, for personal and non-commercial use.
