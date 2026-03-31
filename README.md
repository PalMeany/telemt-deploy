# telemt-deploy
### Simple set of commands that will help you deploy your own telemt proxy server in 5-10 minutes. For dummies!

So you've decided to run your own proxy for telegram...

## First thing you'll need is a VPS/VDS server 
### or any virtual server whatsoever

# I AM IN PROGRESS OF WRITING THIS SO HERE ARE COMMANDS TO START FAST (semi-auto)
```
apt update && apt upgrade -y && apt install btop jq nload nano -y
wget -qO- "https://github.com/telemt/telemt/releases/latest/download/telemt-$(uname -m)-linux-$(ldd --version 2>&1 | grep -iq musl && echo musl || echo gnu).tar.gz" | tar -xz
mv telemt /bin
chmod +x /bin/telemt
mkdir /etc/telemt
cat > /etc/telemt/telemt.toml << 'EOF'
[general]
ad_tag = "Ad tag you get from @MTProxybot"
use_middle_proxy = true
me2dc_fallback = true
fast_mode = true

[general.modes]
classic = false
secure = false
tls = true

[general.links]
show = "*"

[server]
port = 443 # Any port you like, 443 recommended
max_connections = 0

[server.api]
enabled = true
listen = "0.0.0.0:9091"
whitelist = ["127.0.0.0/8"]
minimal_runtime_enabled = false
minimal_runtime_cache_ttl_ms = 1000

[censorship]
tls_domain = "" # Some domain you will be masking as
mask_host = "" # IP that the domain above leads to
mask_port = 443
mask = true
tls_emulation = true
tls_front_dir = "tlsfront"

[access.users]
public = "any 32 characters hex string"
EOF
useradd -d /opt/telemt -m -r -U telemt
chown -R telemt:telemt /etc/telemt
cat > /etc/systemd/system/telemt.service << 'EOF'
[Unit]
Description=Telemt
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=telemt
Group=telemt
WorkingDirectory=/opt/telemt
ExecStart=/bin/telemt /etc/telemt/telemt.toml
Restart=on-failure
LimitNOFILE=65536
AmbientCapabilities=CAP_NET_BIND_SERVICE
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl start telemt
systemctl enable telemt
curl http://localhost:9091/v1/users | jq
```
