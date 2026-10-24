# DHCP-CF
Termux Network DHCP Tool  An IPv4 DHCP/DNS toolkit

pkg update
pkg install unzip curl nano
termux-setup-storage

git clone https://github.com/ChainsawProtocol/DHCP-CF.git

cd DHCP-CF

see in directory termux-netdhcp.zip
just -> unzip "$HOME/storage/downloads/termux-netdhcp.zip" -d "$HOME"
cd "$HOME/termux-netdhcp"
bash install-termux.sh
./ndhcp doctor



Termux Network DHCP Tool: Installation Tutorial

For beginners and experienced Termux users · Toolkit v1.0.0 · 24 October 2026

This guide installs the toolkit created in this conversation. Start with the
beginner path for explanations and checkpoints. Experienced users can use the
quick path, then the advanced sections for deployment and DHCP.

Download the original termux-netdhcp.zip

Choose what you want to run

Goal

What you need

Where to go

Test DNS with 1.1.1.1 and 1.0.0.1

Termux and Internet access; no root

Beginner steps 1–4

Set the phone's Private DNS

Android 9+; no root

Beginner step 5

Publish an authenticated HTTP endpoint

Cloudflare account, a domain in that account, and Access setup; no root

Beginner steps 6–9

Protect an existing local web app

That app listening on loopback

Advanced section A

Assign addresses to devices on your LAN

Real Android root, a compatible device, and a dedicated LAN you control

Advanced section D

DNS, DHCP and Tunnel are separate functions. DNS resolves names. DHCP
assigns local network settings. Tunnel connects the HTTP gateway to Cloudflare.
Choosing 1.1.1.1 does not automatically create a tunnel. DHCP broadcasts
stay on your LAN; this setup does not tunnel all phone traffic.

Experienced-user quick path

Use this if you already understand Termux, shell paths and Cloudflare Access.
For a first installation, the numbered beginner steps below explain each part.

Extract the ZIP under $HOME, enter termux-netdhcp, and run:

bash install-termux.sh
./ndhcp doctor
./ndhcp dns query example.com

Create a Cloudflare Access self-hosted application for your complete public
hostname, with an Allow policy for your users. Record its team name and AUD.
For a new locally managed tunnel, authenticate and create it:

cloudflared tunnel login
cloudflared tunnel create netdhcp

Replace the example values before running this block. Use the UUID printed
by the create command, not the tunnel name:

NETDHCP_TEAM='your-team'
NETDHCP_AUD='your-application-aud'
NETDHCP_HOST='net.your-domain.tld'
NETDHCP_TUNNEL_ID='your-tunnel-uuid'

./ndhcp init --team "$NETDHCP_TEAM" --aud "$NETDHCP_AUD" \
  --hostname "$NETDHCP_HOST"
./ndhcp gateway check --config gateway.json
./ndhcp tunnel config --id "$NETDHCP_TUNNEL_ID" --settings gateway.json
cloudflared tunnel --config "$PWD/cloudflared.yml" ingress validate
cloudflared tunnel route dns "$NETDHCP_TUNNEL_ID" "$NETDHCP_HOST"

Run the gateway and tunnel in separate normal-user Termux sessions:

./ndhcp gateway run --config gateway.json

./ndhcp tunnel run --config cloudflared.yml

Open your HTTPS hostname, sign in, and complete the checks in beginner step 9.
For an existing installation, preserve and edit its settings rather than
recreating them with init --force.

Beginner path

1. Prepare Termux

For a new installation, obtain Termux from F-Droid
or the project's official GitHub releases.
The official installation instructions
describe the available builds. Use matching installation sources for Termux
and any Termux companion apps. If you already have a working installation,
keep your files; back them up before changing app distributions.

Open Termux and let its initial setup finish. Run:

pkg update
pkg install unzip curl nano

pkg installs software inside Termux. unzip opens the download, curl
checks the gateway later, and nano lets you edit settings.

Checkpoint: the commands finish and return to the terminal prompt. If
package downloads fail, use the troubleshooting table near the end.

2. Download and extract the toolkit

Download the ZIP linked at the top of this guide through ChatGPT. Save it
in Android's Downloads folder as termux-netdhcp.zip. This is a private
download; do not try to use its ChatGPT page URL as a curl download URL.

Give Termux access to Downloads:

termux-setup-storage

Accept Android's storage-permission prompt, then run:

ls "$HOME/storage/downloads"
unzip "$HOME/storage/downloads/termux-netdhcp.zip" -d "$HOME"
cd "$HOME/termux-netdhcp"
ls

If Android saved a different filename, use that exact name in the unzip
command. If this directory already contains an installation, skip extraction
or extract your new copy into a separate directory; preserve your settings.

Checkpoint: you see ndhcp, install-termux.sh, requirements.txt,
README.md and the netdhcp directory.

Keep the working copy under $HOME. Android shared storage such as
/sdcard/Download can prevent executables and virtual environments from
working properly.

3. Install the dependencies

Run the installer as the normal Termux user:

bash install-termux.sh

It installs Python, cloudflared, network utilities and the Python packages
listed in requirements.txt. It creates a .venv directory to hold the
project's Python environment. No activation command is needed: ndhcp
selects this environment automatically.

If a prebuilt cryptography package is unavailable in your Termux repository,
the installer adds build tools for pip. That path can take longer and need
more free space. Let installation finish before running the next command.

./ndhcp --version
./ndhcp doctor

Checkpoint: version 1.0.0 appears. The doctor prints a JSON report with
the DNS addresses and paths to available tools. Missing dnsmasq is normal
if you have not installed the optional DHCP component. Android can restrict
address/route inspection even when DNS queries and tunneling work.

Do not run the installer, gateway or tunnel with su. Root is needed only
for the optional DHCP service. If id -u reports 0, leave the root shell
and use a normal Termux session for these steps.

4. Try the DNS and request checks

./ndhcp dns query example.com
./ndhcp dns query example.com --type AAAA

The toolkit queries Cloudflare over HTTPS, trying 1.1.1.1 first and
1.0.0.1 if the first endpoint fails. Certificate verification stays enabled.

Checkpoint: the output normally contains "Status": 0, DNS answers,
and a resolver value of 1.1.1.1 or 1.0.0.1. Status 3 means the queried
name does not exist; it is different from a connection failure.

Try these offline examples too:

./ndhcp traffic-check '/api/items?q=hello'
./ndhcp traffic-check '/%252e%252e/secret'

Checkpoint: the first sample has "would_block": false. The traversal
sample has "would_block": true. These commands inspect sample text; they
do not visit a website or authenticate anyone.

You now have a working local installation. You can stop here if you only
need DNS diagnostics.

5. Optional: set Android's own Private DNS

Show the instructions:

./ndhcp dns android

On Android 9 or later, open Settings → Network & internet → Private DNS
and choose Private DNS provider hostname. Enter:

one.one.one.one

Menu names vary by phone. Use the hostname in this field. The addresses
1.1.1.1 and 1.0.0.1 belong in a router's IPv4 DNS fields or suitable
manual network DNS fields. This setting is independent of Tunnel.

The toolkit's DNS query command does not change Android's system resolver.
Likewise, DHCP advertising these addresses does not itself encrypt client
DNS. For device details, see Cloudflare's Android setup guide.

6. Prepare Cloudflare Access

Continue only if you want to reach the authenticated HTTP endpoint from
outside the phone. Have your Cloudflare account and a domain managed in
that account ready. You can choose an unused hostname such as
net.your-domain.tld.

In Cloudflare Zero Trust, create an Access self-hosted application for
the entire chosen hostname. Cover all paths. Add an Allow policy for
the users who should be able to sign in. For your first test, include your
own account. Keep this application protected by Access.

Write down these values:

Value used below

Meaning

Where it comes from

YOUR_TEAM

The team name only

The YOUR_TEAM part of https://YOUR_TEAM.cloudflareaccess.com

YOUR_AUD

Application Audience tag

This specific Access application's settings

net.example.com

Your chosen public hostname

A hostname within your own Cloudflare-managed domain

The AUD is not your account API token, and the team name is not your email
address. Never paste an API secret in the AUD field. Cloudflare's
JWT validation guide
explains how to find the application's AUD.

7. Create the tunnel and matching settings

Back in the project directory:

cd "$HOME/termux-netdhcp"
umask 077
cloudflared tunnel login

Complete the browser login for your Cloudflare account and domain. If a
browser does not open automatically, open the authorization URL printed by
cloudflared yourself. Return to Termux after it reports success.

Create a named, locally managed tunnel:

cloudflared tunnel create netdhcp

Checkpoint: cloudflared prints a UUID and the location of its credentials
JSON. Record the UUID as TUNNEL_UUID. Keep the JSON and cloudflared account
certificate private; they are not files to share for troubleshooting.

In the next commands, replace all four placeholders with your real values:
YOUR_TEAM, YOUR_AUD, net.example.com and TUNNEL_UUID. Do not type angle
brackets around your replacements.

./ndhcp init \
  --team YOUR_TEAM \
  --aud YOUR_AUD \
  --hostname net.example.com

./ndhcp gateway check --config gateway.json

./ndhcp tunnel config \
  --settings gateway.json \
  --id TUNNEL_UUID \
  --output cloudflared.yml

cloudflared tunnel --config "$PWD/cloudflared.yml" ingress validate
cloudflared tunnel route dns TUNNEL_UUID net.example.com

The \ at the end of a line continues a command onto the next line. Copy
the whole command, including its following lines.

Checkpoint: gateway.json and cloudflared.yml exist; the settings check
reports "valid": true; cloudflared accepts the ingress rules and creates
the DNS route. The gateway check validates local settings, not your live
Access policy or account credentials.

The default credentials path is ~/.cloudflared/TUNNEL_UUID.json. If the
create command printed another path, supply it with --credentials when
generating cloudflared.yml.

If a DNS record already uses your hostname, choose a different unused
hostname or deliberately resolve the conflict in your Cloudflare account.
Do not delete unrelated records just to make this command pass.

8. Start the two services

Use two normal-user Termux sessions. Open the second from Termux's
session drawer while keeping the first running.

In session A, run:

cd "$HOME/termux-netdhcp"
./ndhcp gateway run --config gateway.json

Checkpoint: it reports Gateway listening on 127.0.0.1:8787; Access JWT required.
It keeps running instead of returning to the prompt; this is expected.

In session B, run:

cd "$HOME/termux-netdhcp"
./ndhcp tunnel run --config cloudflared.yml

Checkpoint: cloudflared validates the configuration and reports tunnel
connections. Leave both sessions open. If either command exits with an
error, correct that error before proceeding.

9. Verify sign-in and rejection

Open https://net.example.com in your browser, using your actual hostname.
Sign in as a user allowed by the Access policy.

Expected: the default endpoint returns JSON containing:

{
  "authenticated": true,
  "upstream_configured": false
}

The full response has additional fields. At this stage the tool is serving
its authenticated status endpoint; you have not connected another web app.

Open a third Termux session and test the local origin without credentials:

cd "$HOME/termux-netdhcp"
curl -i -H 'Host: net.example.com' http://127.0.0.1:8787/

Replace the Host value with the same public hostname used in gateway.json.
Expected: HTTP 401. This confirms that reaching the local gateway alone
does not bypass token verification.

Also try the public hostname in a private browser window. Without an
authorized sign-in, Access should present a login flow or deny access.
Public edge responses can differ from the local origin's 401.

What the authentication proves: the request has a valid Access
application JWT for your configured issuer and audience. Rate limits and
pattern checks apply afterwards. A valid identity does not prove that a
request is harmless; this is not general malware scanning.

10. Stop or resume

Press Ctrl-C in session A and session B to stop the respective services.
Termux's extra-key row can supply Ctrl. Start them again with the same two
run commands; do not recreate the tunnel each time.

If Android suspends Termux during use, a wake lock may help:

termux-wake-lock

Release it when finished:

termux-wake-unlock

A wake lock is not a guarantee against Android process termination. This
installation does not install an automatic boot service. Stopping it does
not remove the Cloudflare DNS record, tunnel or Access application.

Advanced section A: protect an existing local web app

Have your application listen on 127.0.0.1:8080, using a different port from
the gateway. The gateway supports a literal 127.0.0.1 or ::1 HTTP upstream
with an explicit port. It rejects public or LAN upstream addresses and the
hostname localhost.

For a new configuration, add this argument to init:

./ndhcp init --team YOUR_TEAM --aud YOUR_AUD \
  --hostname net.example.com \
  --upstream http://127.0.0.1:8080

For an existing configuration, preserve your other choices and edit only
the upstream field:

cd "$HOME/termux-netdhcp"
cp -p gateway.json gateway.json.before-upstream
nano gateway.json

Change:

"upstream": null

to:

"upstream": "http://127.0.0.1:8080"

Keep the existing comma if one follows the value. In nano, use Ctrl-O then
Enter to save and Ctrl-X to exit. Validate and restart the gateway:

./ndhcp gateway check --config gateway.json

Configure the application's public base URL as your HTTPS hostname. The
gateway does not rewrite absolute redirect URLs or cookie domains. It
buffers HTTP traffic; WebSockets, SSE, streaming responses and compressed
request bodies are not supported in this version.

Other applications on the phone may be able to reach a loopback upstream
directly. Use upstream authentication or stronger OS isolation where those
local callers are untrusted. An incoming identity header alone is not a
safe substitute for that trust boundary.

Advanced section B: access limits, checks and session management

Run the packaged tests:

cd "$HOME/termux-netdhcp"
.venv/bin/python -m unittest discover -s tests -v

The original build passed 35 automated tests on Linux. Test coverage and
limits are described in TEST_REPORT.md and ALGORITHMS.md.

Settings in gateway.json include:

Setting

Default

Purpose

global_rps / global_burst

30 / 60

Limit all requests before identity verification

identity_rps / identity_burst

5 / 20

Limit each verified user or service

max_body_bytes

1048576

Maximum request body

max_response_bytes

8388608

Maximum buffered upstream response

risk_threshold

5

Score at which a request is blocked

jwt_leeway_seconds

15

Allowed clock skew

allowed_identities

Empty list

Optional additional local identity allowlist

Back up settings before editing. An empty identity list accepts identities
authorized by this Access application. To narrow it locally, use verified
values in the form user:<sub> or service:<common_name>. The default
authenticated status response shows the validated identity for your session.
Do not derive an allowlist from an unsigned email or IP header.

Validate settings and restart the gateway after changes. If you change
team, AUD, hostname or gateway port, regenerate cloudflared.yml from those
settings and restart the tunnel too. Changing the upstream alone does not
require regenerating the tunnel config. init --force recreates defaults,
so editing is preferable when you already have custom settings.

For automated clients, configure an Access Service Auth policy and
service tokens in Cloudflare. The toolkit recognizes the signed service
common_name when the token's sub is empty. Manage service-token secrets
outside the tutorial and shell history.

To keep both processes in a tmux session after validating them in foreground:

pkg install tmux
tmux new-session -d -s netdhcp -n gateway -c "$HOME/termux-netdhcp" \
  'bash ./ndhcp gateway run --config gateway.json'
tmux new-window -t netdhcp -n tunnel -c "$HOME/termux-netdhcp" \
  'bash ./ndhcp tunnel run --config cloudflared.yml'
tmux attach -t netdhcp

Use Ctrl-B then N to switch windows; Ctrl-B then D detaches. Reattach with
tmux attach -t netdhcp. To stop both processes in this dedicated session:

tmux kill-session -t netdhcp

tmux preserves terminal sessions, but cannot prevent Android from killing
the app's processes.

Advanced section C: verify the original download

Optional integrity check for the exact ZIP supplied with toolkit v1.0.0:

sha256sum "$HOME/storage/downloads/termux-netdhcp.zip"

Expected SHA-256:

725001168c063af752b8890747bb16309b898ebdde5c0cb755d083f359a999df

This checksum identifies the original archive. It will differ if the archive
is repackaged or a later release is downloaded.

Advanced section D: optional root-only DHCP server

D1. Requirements

Use a dedicated LAN you control, with no other DHCP server on that LAN.
Android's built-in hotspot may already run one. A local port check cannot
discover every DHCP server elsewhere on the network.

The phone needs real root and the necessary kernel/SELinux permissions.
A proot Linux environment does not grant Android network privileges. If
you do not have these capabilities, use your router for DHCP; the rest of
the toolkit can still run without root.

Install the optional component as the normal Termux user:

cd "$HOME/termux-netdhcp"
bash install-termux.sh --with-dhcp

This enables Termux's root repository and installs dnsmasq. Package
availability can vary by repository and device.

D2. Prepare an actual LAN interface

The following is an example for a dedicated USB Ethernet LAN. Substitute
your real interface and addressing. It is not a configuration to apply to
the Wi-Fi connection you are currently using for Internet access.

Item

Example

Dedicated interface

eth0

LAN subnet

192.168.50.0/24

Phone's DHCP-server address

192.168.50.2

Existing router's address

192.168.50.1

Lease pool

192.168.50.50 through 192.168.50.100

The .1 router in this example must already route that LAN. The phone's
DHCP process does not create Internet routing or NAT.

Inspect the interface:

su -c "'$PREFIX/bin/ip' -4 address show dev eth0"

If this is your dedicated interface, the address is unused on the LAN, and
you have not already configured it, these root commands assign the example
address and bring up the link:

su -c "'$PREFIX/bin/ip' address add 192.168.50.2/24 dev eth0"
su -c "'$PREFIX/bin/ip' link set dev eth0 up"

Skip the add command if the correct address already exists. Interface names
and permissions vary across Android devices. If the OS denies these actions,
resolve the device's supported network setup; disabling SELinux is not a
step in this tutorial.

D3. Generate and review DHCP settings

As the normal Termux user:

cd "$HOME/termux-netdhcp"
./ndhcp dhcp config \
  --interface eth0 \
  --subnet 192.168.50.0/24 \
  --server 192.168.50.2 \
  --router 192.168.50.1 \
  --start 192.168.50.50 \
  --end 192.168.50.100 \
  --lease 12h \
  --output dhcp.conf

cat dhcp.conf

Checkpoint: the pool excludes both the server and router. The file has:

dhcp-option=option:dns-server,1.1.1.1,1.0.0.1

It also has port=0, so dnsmasq's DNS service is disabled. Clients receive
the two resolver addresses through DHCP. Clients may use either resolver;
their order does not force primary/backup behavior or encrypted DNS.

D4. Start and test DHCP

Start only after confirming that no other DHCP server serves this LAN:

su -c "PATH='$PREFIX/bin:/system/bin' '$PREFIX/bin/bash' '$PWD/ndhcp' dhcp serve --config '$PWD/dhcp.conf' --isolated-lan"

The runner checks the interface address and prefix, tests local UDP port 67,
validates the generated directives, tests dnsmasq's syntax and starts it in
the foreground. The DHCP process stays root for Android interface and lease
directory access. Keep the HTTP gateway and cloudflared in normal-user sessions.

Connect a test client to this LAN and request a lease through that client's
network controls. In another session:

cd "$HOME/termux-netdhcp"
./ndhcp dhcp leases --file dhcp-state/dnsmasq.leases

Checkpoint: the client receives a pool address and shows router
192.168.50.1 with the Cloudflare DNS addresses. Check those values on the
client as well as in the lease list. IPv6 configuration is separate and is
not managed by this IPv4 DHCP tool.

Press Ctrl-C in the DHCP session to stop it. If you added the example address
solely for this test, remove that address afterwards with:

su -c "'$PREFIX/bin/ip' address del 192.168.50.2/24 dev eth0"

Remove only an address you added for the test. The tutorial does not change
the router's configuration or Android's existing DHCP client settings.

Troubleshooting

Symptom

Check or next step

ZIP not found

Run ls "$HOME/storage/downloads"; confirm storage permission and use the actual downloaded filename.

cd says directory missing

Check the ZIP's contents with unzip -l; the original archive contains the termux-netdhcp directory.

./ndhcp: Permission denied

Keep the project under $HOME; run chmod 700 ndhcp. bash ndhcp doctor also bypasses the executable bit/shebang.

Package repository errors

Run termux-change-repo, choose a working official mirror, then pkg update. If you use the root repository, check its mirror too.

Missing Python dependency

From the project directory, rerun bash install-termux.sh and read the first installation error.

Cryptography build fails

Check space and package mirrors. The installer prefers a prebuilt package and otherwise installs Rust/C build dependencies.

Doctor cannot list addresses

Android may restrict network inspection. Try the DNS query separately; root DHCP needs additional device permissions.

Both DNS endpoints fail

Check Internet access, phone date/time and whether this network blocks the endpoints. Keep TLS verification enabled.

File exists while generating settings

Use the existing settings, choose a different output filename, or deliberately recreate with --force after a backup.

Team/hostname rejected

Team is the name only, in lowercase. Hostname is a DNS name without https://, a port or a path.

Credentials file missing

Use the UUID and path printed by cloudflared tunnel create; supply --credentials if different from the default.

Public hostname unavailable

Read cloudflared's error, verify tunnel credentials and DNS route, and confirm both foreground processes remain running.

Local HTTP 401

Expected without a token. For an authenticated request, check issuer, AUD, session expiry and any local identity allowlist.

Local HTTP 403

Inspect the audit reason: suspicious-request score or cross-site write check. Review the input and application's intended behavior.

Local HTTP 421

Your Host header does not match gateway.json; use the exact configured hostname.

Local HTTP 429

Global or per-identity request limit reached. Wait for refill and review legitimate request volume before changing limits.

Local HTTP 502

The configured upstream is unavailable, timed out, returned malformed HTTP or exceeded the response limit.

Local HTTP 503

Cloudflare signing keys could not be refreshed. Check connectivity and retry after the 30-second refresh cooldown.

Gateway/tunnel refuses root

Run it in a normal Termux shell. Root is reserved for DHCP service.

DHCP says interface address mismatch

Confirm the selected interface already has the configured server address with the correct subnet prefix.

DHCP says port 67 unavailable

Another local DHCP service or Android restriction may be involved; do not start a competing DHCP server.

No DHCP leases

Verify the client is on the same dedicated LAN, inspect permissions and DHCP logs, and check client network settings.

Services disappear in background

Android may stop Termux processes. Reopen the sessions; a wake lock/tmux can help with operation but does not guarantee uptime.

Completion checkpoints

Version and doctor commands run from the extracted project.

A real DNS query succeeds on the phone.

Cloudflare Access covers the whole chosen hostname and allows the intended users.

Gateway settings and cloudflared ingress validate.

Both services run as the normal Termux user.

An allowed public session returns the authenticated status or intended upstream app.

A local request without an Access JWT receives 401.

If DHCP is used, a real client receives the intended pool, router and DNS settings.

Validation limit: the original toolkit has 35 passing automated Linux
tests, including signed JWT and local HTTP checks. A live Termux installation,
real Cloudflare deployment and actual DHCP lease exchange were not tested
in the creation environment. The phone/network checkpoints above complete
those deployment checks. Pattern scoring can have false positives and
evasions; use application authorization and input validation too.

Official references

Termux installation

Termux package management and mirror troubleshooting

Cloudflare locally managed Tunnel setup

Cloudflare Access JWT validation

Cloudflare Access application and service-token claims

Cloudflare Android DNS setup

dnsmasq manual

For the toolkit's full algorithm details, read ALGORITHMS.md inside the ZIP.
