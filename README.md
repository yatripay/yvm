# YatriPay Virtual Machine (YVM)

The official node client for the **YatriPay** blockchain, an EVM-compatible network secured by
Clique Proof-of-Authority (PoA).

YVM is based on [go-ethereum](https://github.com/ethereum/go-ethereum) **v1.13.15 (Ontamalca)**,
the last upstream release that supports Clique PoA.

## Network Information

| Parameter      | Value                                                            |
| -------------- | ---------------------------------------------------------------- |
| Network Name   | YatriPay                                                         |
| Chain ID       | `8934`                                                           |
| Consensus      | Clique Proof-of-Authority (3s block time)                        |
| RPC URL        | [https://mnrpc.yatripay.com](https://mnrpc.yatripay.com)         |
| WSS URL        | `wss://mnrpc.yatripay.com/ws`                                    |
| Block Explorer | [https://yvmexplorer.yatripay.com](https://yvmexplorer.yatripay.com) |
| Website        | [https://yatripay.com](https://yatripay.com)                     |

### Add YatriPay to MetaMask

1. Open MetaMask → **Settings** → **Networks** → **Add network** → **Add a network manually**.
2. Enter the values below and save:
   - **Network name:** YatriPay
   - **New RPC URL:** `https://mnrpc.yatripay.com`
   - **Chain ID:** `8934`
   - **Block explorer URL:** `https://yvmexplorer.yatripay.com`

## Building from Source

Requirements: [Go](https://go.dev/dl/) 1.21 or later and a C compiler (e.g. `build-essential` on Debian/Ubuntu).

```shell
git clone https://github.com/yatripay/yvm.git
cd yvm
make geth
```

The binary is written to `build/bin/geth`. Optionally install it system-wide:

```shell
sudo install -m 0755 build/bin/geth /usr/local/bin/geth
geth version   # should report 1.13.15-stable
```

## Running a Node

The YatriPay genesis file, [`yatripay.json`](./yatripay.json), is included in the root of this
repository. Every node must be initialised with it before first start.

### 1. Initialise the data directory (first run only)

```shell
geth --datadir "$HOME/.yatripay" init yatripay.json
```

### 2. Start the node

```shell
geth \
  --datadir "$HOME/.yatripay" \
  --networkid 8934 \
  --port 19090 \
  --syncmode full \
  --bootnodes "enode://07b121b83114dcfe963f13a66f7555c50be6cfa1b8f3b1036945b59d14ad2478e3982a089199e73f85a2777a109b9bf599fceac52fd1194085fac89f26c50dc2@13.233.118.201:19090"
```

Port `19090` (TCP **and** UDP) must be reachable for peer-to-peer connectivity. No other port
needs to be opened to the internet.

### 3. (Optional) Enable local RPC

If your applications need JSON-RPC access, bind it to localhost and expose only the namespaces
you need:

```shell
  --http --http.addr 127.0.0.1 --http.port 8545 --http.api eth,net,web3 \
  --ws   --ws.addr   127.0.0.1 --ws.port   8546 --ws.api   eth,net,web3
```

To serve RPC publicly, put the node behind a reverse proxy (e.g. Nginx or Caddy) with TLS and
rate limiting rather than binding geth directly to `0.0.0.0`.

### Running as a service (recommended)

Instead of running the node inside `screen`, use `systemd` so that it restarts automatically and
runs under an unprivileged user:

```shell
sudo useradd --system --create-home --home-dir /var/lib/yatripay --shell /usr/sbin/nologin yatripay
sudo cp yatripay.json /var/lib/yatripay/
sudo -u yatripay geth --datadir /var/lib/yatripay/data init /var/lib/yatripay/yatripay.json
```

Create `/etc/systemd/system/yatripay.service`:

```ini
[Unit]
Description=YatriPay Node (YVM)
After=network-online.target
Wants=network-online.target

[Service]
User=yatripay
Group=yatripay
ExecStart=/usr/local/bin/geth \
  --datadir /var/lib/yatripay/data \
  --networkid 8934 \
  --port 19090 \
  --syncmode full \
  --bootnodes enode://07b121b83114dcfe963f13a66f7555c50be6cfa1b8f3b1036945b59d14ad2478e3982a089199e73f85a2777a109b9bf599fceac52fd1194085fac89f26c50dc2@13.233.118.201:19090
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
NoNewPrivileges=true
ProtectSystem=full
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Then enable and follow the logs:

```shell
sudo systemctl daemon-reload
sudo systemctl enable --now yatripay
journalctl -u yatripay -f
```

### Attaching a console

```shell
geth attach "$HOME/.yatripay/geth.ipc"          # or /var/lib/yatripay/data/geth.ipc for the service
> net.peerCount
> eth.blockNumber
```

The IPC socket is only accessible to the local user, so it is the safest way to administer a node.

## Security Recommendations

- **Never** expose HTTP/WS RPC on `0.0.0.0` without a firewall or authenticated reverse proxy.
- **Never** enable the `personal`, `admin`, `debug` or `miner` RPC namespaces on a public endpoint.
- Do not keep unlocked accounts (`--unlock`, `--allow-insecure-unlock`) on internet-facing nodes.
- Back up `<datadir>/keystore` and its password securely if the node holds any keys.
- Keep the data directory (`.yatripay`) out of version control; it is already listed in `.gitignore`.

## License

YVM is derived from go-ethereum and inherits its licensing:

- The library code (everything outside the `cmd` directory) is licensed under the
  [GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html),
  included in this repository as `COPYING.LESSER`.
- The binaries (everything inside the `cmd` directory) are licensed under the
  [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html),
  included in this repository as `COPYING`.
