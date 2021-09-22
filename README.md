<h1 align="center">
  <img src="https://github.com/Dreamacro/clash/raw/master/docs/logo.png" alt="Clash" width="200">
  <br>Clash<br>
</h1>

<h4 align="center">A rule-based tunnel in Go.</h4>

<p align="center">
  <a href="https://github.com/Dreamacro/clash/actions">
    <img src="https://img.shields.io/github/workflow/status/Dreamacro/clash/Go?style=flat-square" alt="Github Actions">
  </a>
  <a href="https://goreportcard.com/report/github.com/Dreamacro/clash">
    <img src="https://goreportcard.com/badge/github.com/Dreamacro/clash?style=flat-square">
  </a>
  <img src="https://img.shields.io/github/go-mod/go-version/Dreamacro/clash?style=flat-square">
  <a href="https://github.com/Dreamacro/clash/releases">
    <img src="https://img.shields.io/github/release/Dreamacro/clash/all.svg?style=flat-square">
  </a>
  <a href="https://github.com/Dreamacro/clash/releases/tag/premium">
    <img src="https://img.shields.io/badge/release-Premium-00b4f0?style=flat-square">
  </a>
</p>

## Features

- Local HTTP/HTTPS/SOCKS server with authentication support
- VMess, Shadowsocks, Trojan, Snell protocol support for remote connections
- Built-in DNS server that aims to minimize DNS pollution attack impact, supports DoH/DoT upstream and fake IP.
- Rules based off domains, GEOIP, IPCIDR or Process to forward packets to different nodes
- Remote groups allow users to implement powerful rules. Supports automatic fallback, load balancing or auto select node based off latency
- Remote providers, allowing users to get node lists remotely instead of hardcoding in config
- Netfilter TCP redirecting. Deploy Clash on your Internet gateway with `iptables`.
- Comprehensive HTTP RESTful API controller

## Getting Started
Documentations are now moved to [GitHub Wiki](https://github.com/Dreamacro/clash/wiki).

## Advanced usage for this fork branch
### TUN configuration
Support macOS,Linux and Windows.

For Windows, you should download the [Wintun](https://www.wintun.net) driver and copy `wintun.dll` into the System32 directory.
```yaml
# Enable the TUN listener
tun:
  enable: true
  stack: system # system or gvisor
  dns-listen: 0.0.0.0:53 # additional dns server listen on TUN
  auto-route: true # auto set global route
```
### Rules configuration
- Support rule `GEOSITE`.
- Support `multiport` condition for rule `SRC-PORT` and `DST-PORT`.
- Support not match condition for rule `GEOIP`.
- Support `network` condition for all rules.

The `GEOSITE` and `GEOIP` databases via https://github.com/Loyalsoldier/v2ray-rules-dat.
```yaml
rules:
  # network condition for rules
  - DOMAIN-SUFFIX,bilibili.com,DIRECT,tcp
  - DOMAIN-SUFFIX,bilibili.com,REJECT,udp
    
  # multiport condition for rules SRC-PORT and DST-PORT
  - DST-PORT,123/136/137-139,DIRECT,udp
  
  # rule GEOSITE
  - GEOSITE,category-ads-all,REJECT
  - GEOSITE,icloud@cn,DIRECT
  - GEOSITE,apple@cn,DIRECT
  - GEOSITE,apple-cn,DIRECT
  - GEOSITE,microsoft@cn,DIRECT
  - GEOSITE,facebook,PROXY
  - GEOSITE,youtube,PROXY
  - GEOSITE,geolocation-cn,DIRECT
  - GEOSITE,gfw,PROXY
  - GEOSITE,greatfire,PROXY
  #- GEOSITE,geolocation-!cn,PROXY

  - GEOIP,telegram,PROXY,no-resolve
  - GEOIP,private,DIRECT,no-resolve
  - GEOIP,cn,DIRECT
    
  # Not match condition for rule GEOIP
  #- GEOIP,!cn,PROXY

  - MATCH,PROXY
```

### Proxies configuration
Support outbound transport protocol `VLESS`.

The XTLS only support TCP transport by the XRAY-CORE.
```yaml
proxies:
  - name: "vless-tcp"
    type: vless
    server: server
    port: 443
    uuid: uuid
    network: tcp
    servername: example.com # AKA SNI
    # udp: true
    # flow: xtls-rprx-direct # xtls-rprx-origin  # enable XTLS
    # skip-cert-verify: true
    
  - name: "vless-ws"
    type: vless
    server: server
    port: 443
    uuid: uuid
    udp: true
    network: ws
    servername: example.com # priority over wss host
    # skip-cert-verify: true
    ws-path: /path
    ws-headers:
      Host: example.com

  - name: "vless-h2"
    type: vless
    server: server
    port: 443
    uuid: uuid
    network: h2
    servername: example.com
    # skip-cert-verify: true
    h2-opts:
      host:
        - http.example.com
        - http-alt.example.com
      path: /

  - name: "vless-http"
    type: vless
    server: server
    port: 443
    uuid: uuid
    # udp: true
    network: http
    servername: example.com
    # skip-cert-verify: true
    http-opts:
      method: "GET"
      path:
        - '/'
        - '/video'
      headers:
        Connection:
          - keep-alive

  - name: vless-grpc
    server: server
    port: 443
    type: vless
    uuid: uuid
    network: grpc
    servername: example.com
    # skip-cert-verify: true
    grpc-opts:
      grpc-service-name: "example"
```

### IPTABLES auto-configuration
Only work on Linux OS who support `iptables`, Clash will auto-configuration iptables for tproxy listener when `tproxy-port` value isn't zero.

If `TPROXY` is enabled, the `TUN` must be disabled.
```yaml
# Enable the TPROXY listener
tproxy-port: 9898
# Disable the TUN listener
tun:
  enable: false
```
Create user given name `clash`.

Run Clash by user `clash` as a daemon.

Create the systemd configuration file at /etc/systemd/system/clash.service:
```shell
[Unit]
Description=Clash daemon, A rule-based proxy in Go.
After=network.target

[Service]
Type=simple
User=clash
Group=clash
CapabilityBoundingSet=cap_net_admin
AmbientCapabilities=cap_net_admin
Restart=always
ExecStart=/usr/local/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
```
Launch clashd on system startup with:
```shell
$ systemctl enable clash
```
Launch clashd immediately with:
```shell
$ systemctl start clash
```

### Display Process name
Add field `Process` to `Metadata` and prepare to get process name for Restful API `GET /connections`.

To display process name in GUI please use https://yaling888.github.io/yacd/.

## Premium Release
[Release](https://github.com/Dreamacro/clash/releases/tag/premium)

## Development
If you want to build an application that uses clash as a library, check out the the [GitHub Wiki](https://github.com/Dreamacro/clash/wiki/use-clash-as-a-library)

## Credits

* [riobard/go-shadowsocks2](https://github.com/riobard/go-shadowsocks2)
* [v2ray/v2ray-core](https://github.com/v2ray/v2ray-core)
* [WireGuard/wireguard-go](https://github.com/WireGuard/wireguard-go)

## License

This software is released under the GPL-3.0 license.

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2FDreamacro%2Fclash.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2FDreamacro%2Fclash?ref=badge_large)
