# BASE-NODE
```
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

```
mkdir ~/base-node
cd ~/base-node
```

Create docker-compose.yml:
```
services:
  base-node:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:latest
    container_name: base-node
    restart: always
    ports:
      - "8545:8545"
      - "8546:8546"
    volumes:
      - ./data:/root/.ethereum
    command:
      [
        "--http",
        "--http.api", "debug,eth,net,web3,txpool",
        "--http.addr", "0.0.0.0",
        "--http.port", "8545",
        "--http.vhosts", "*",
        "--ws",
        "--ws.api", "debug,eth,net,web3,txpool",
        "--ws.addr", "0.0.0.0",
        "--ws.port", "8546",

        "--syncmode=full",
        "--gcmode=full",
        "--cache=8192",

        "--nodiscover",
        "--maxpeers=0",
        "--nat=none",

        "--txpool.nolocals=false",
        "--txpool.pricelimit=1",

        "--metrics",
        "--verbosity=3"
      ]
```

This creates a private-only Base node

No peer discovery

No tx gossip

No mempool leak

Full RPC + WS support

Perfect for arbitrage systems

```
docker compose up -d
```

Check logs:
```
docker logs -f base-node
```
Test your private RPC:
```
curl --location --request POST 'http://YOUR_IP:8545' \
--header 'Content-Type: application/json' \
--data-raw '{
  "jsonrpc":"2.0",
  "method":"eth_blockNumber",
  "params":[],
  "id":1
}'
```

Realtime Mempool Monitoring
Use eth_subscribe("newPendingTransactions") via WebSocket:
```
const ws = new WebSocket("ws://YOUR_IP:8546");
ws.send(JSON.stringify({
  id: 1,
  method: "eth_subscribe",
  params: ["newPendingTransactions"]
}));
```
You will receive every single pending transaction on Base in real‑time.
