

<div align="center">

#  👨🏻‍💻 **Ritual Infernet Node Guide** 👨🏻‍💻

</div>



## Device / System Requirements 💻

![image](https://github.com/user-attachments/assets/76006807-e83d-4c33-9d9d-8f7c95c8cdeb)
- Ubuntu 20.04 / 22.04 (recommended)  
- Minimum: 2 vCPU, 4 GB RAM (recommended)  
- Stable internet connection

---

## Pre-Requirements 🛠

- Docker Desktop (or Docker + Docker Compose on VPS)  
- An EVM wallet with minimum ~$5 BASE ETH  
- Base Mainnet RPC URL (e.g. from ALCHEMY)  
- Basic CLI familiarity (nano, bash)

Install base apt packages:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt -qy install curl git nano jq lz4 build-essential screen
```

---

## Install Docker & Docker Compose (VPS)

```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update && sudo apt install -y docker-ce && sudo systemctl enable --now docker
sudo usermod -aG docker $USER && newgrp docker

sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r .tag_name)/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker --version && docker-compose --version
```

---

## Allow Incoming connections on Ports (VPS only)

```bash
sudo apt install ufw -y
sudo ufw allow 22
sudo ufw allow 3000
sudo ufw allow 4000
sudo ufw allow 6379
sudo ufw allow 8545
sudo ufw allow ssh
sudo ufw enable
```

---

## Cloning The Starter Repository

```bash
git clone https://github.com/ritual-net/infernet-container-starter
cd infernet-container-starter
```

---

## Running hello-world container

🔺 Before running, ensure Docker is started in background (local PC):

```bash
docker pull ritualnetwork/hello-world-infernet:latest
project=hello-world make deploy-container
```

Open a new terminal/Termius tab for further steps.

---

## Edit Node Configuration Files

Follow these exact steps to replace files and paste the contents shown. Replace placeholders (ENTER_API_KEY, ENTER_YOUR_PVT_KEYS) with your actual values. Always prefix private keys with `0x` when required. Do NOT commit secrets.

---

### 1️⃣ Edit: ~/infernet-container-starter/deploy/config.json

Run:

```bash
rm ~/infernet-container-starter/deploy/config.json && nano ~/infernet-container-starter/deploy/config.json
```

Paste the JSON below (replace placeholders):

```json
{
    "log_path": "infernet_node.log",
    "server": {
        "port": 4000,
        "rate_limit": {
            "num_requests": 100,
            "period": 100
        }
    },
    "chain": {
        "enabled": true,
        "trail_head_blocks": 3,
        "rpc_url": "ENTER_API_KEY",
        "registry_address": "0x3B1554f346DFe5c482Bb4BA31b880c1C18412170",
        "wallet": {
          "max_gas_limit": 4000000,
          "private_key": "ENTER_YOUR_PVT_KEYS",
          "allowed_sim_errors": []
        },
        "snapshot_sync": {
          "sleep": 3,
          "batch_size": 500,
          "starting_sub_id": 240000,
          "sync_period": 30
        }
    },
    "startup_wait": 1.0,
    
    "redis": {
        "host": "redis",
        "port": 6379
    },
    "forward_stats": true,
    "containers": [
        {
            "id": "hello-world",
            "image": "ritualnetwork/hello-world-infernet:latest",
            "external": true,
            "port": "3000",
            "allowed_delegate_addresses": [],
            "allowed_addresses": [],
            "allowed_ips": [],
            "command": "--bind=0.0.0.0:3000 --workers=2",
            "env": {},
            "volumes": [],
            "accepted_payments": {},
            "generates_proofs": false
        }
    ]
}
```

Save: Ctrl+X → Y → Enter

---

### 2️⃣ Edit: ~/infernet-container-starter/projects/hello-world/container/config.json

Run:

```bash
rm ~/infernet-container-starter/projects/hello-world/container/config.json && nano ~/infernet-container-starter/projects/hello-world/container/config.json
```

Paste the JSON below (replace placeholders):

```json
{
    "log_path": "infernet_node.log",
    "server": {
        "port": 4000,
        "rate_limit": {
            "num_requests": 100,
            "period": 100
        }
    },
    "chain": {
        "enabled": true,
        "trail_head_blocks": 3,
        "rpc_url": "ENTER_API_KEY",
        "registry_address": "0x3B1554f346DFe5c482Bb4BA31b880c1C18412170",
        "wallet": {
          "max_gas_limit": 4000000,
          "private_key": "ENTER_YOUR_PVT_KEYS",
          "allowed_sim_errors": []
        },
        "snapshot_sync": {
          "sleep": 3,
          "batch_size": 500,
          "starting_sub_id": 240000,
          "sync_period": 30
        }
    },
    "startup_wait": 1.0,
   
    "redis": {
        "host": "redis",
        "port": 6379
    },
    "forward_stats": true,
    "containers": [
        {
            "id": "hello-world",
            "image": "ritualnetwork/hello-world-infernet:latest",
            "external": true,
            "port": "3000",
            "allowed_delegate_addresses": [],
            "allowed_addresses": [],
            "allowed_ips": [],
            "command": "--bind=0.0.0.0:3000 --workers=2",
            "env": {},
            "volumes": [],
            "accepted_payments": {},
            "generates_proofs": false
        }
    ]
}
```

Save: Ctrl+X → Y → Enter

---

### 3️⃣ Edit: ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol

Run:

```bash
rm ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol && nano ~/infernet-container-starter/projects/hello-world/contracts/script/Deploy.s.sol
```

Paste the Solidity script:

```solidity
// SPDX-License-Identifier: BSD-3-Clause-Clear
pragma solidity ^0.8.13;

import {Script, console2} from "forge-std/Script.sol";
import {SaysGM} from "../src/SaysGM.sol";

contract Deploy is Script {
    function run() public {
        // Setup wallet
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        // Log address
        address deployerAddress = vm.addr(deployerPrivateKey);
        console2.log("Loaded deployer: ", deployerAddress);

        address registry = 0x3B1554f346DFe5c482Bb4BA31b880c1C18412170;
        // Create consumer
        SaysGM saysGm = new SaysGM(registry);
        console2.log("Deployed SaysHello: ", address(saysGm));

        // Execute
        vm.stopBroadcast();
        vm.broadcast();
    }
}
```

Save: Ctrl+X → Y → Enter

---

### 4️⃣ Edit Makefile: ~/infernet-container-starter/projects/hello-world/contracts/Makefile

Run:

```bash
rm ~/infernet-container-starter/projects/hello-world/contracts/Makefile && nano ~/infernet-container-starter/projects/hello-world/contracts/Makefile
```

Paste the Makefile:

```makefile
# phony targets are targets that don't actually create a file
.phony: deploy

# anvil's third default address
sender := ENTER_YOUR_PVT_KEYS
RPC_URL := ENTER_API_KEY

# deploying the contract
deploy:
	@PRIVATE_KEY=$(sender) forge script script/Deploy.s.sol:Deploy --broadcast --rpc-url $(RPC_URL)

# calling sayGM()
call-contract:
	@PRIVATE_KEY=$(sender) forge script script/CallContract.s.sol:CallContract --broadcast --rpc-url $(RPC_URL)
```

- Replace `ENTER_YOUR_PVT_KEYS` with your private key (prefix with `0x`).  
- Replace `ENTER_API_KEY` with your RPC URL (e.g. Alchemy URL).

Save: Ctrl+X → Y → Enter

---

### 5️⃣ Edit: ~/infernet-container-starter/deploy/docker-compose.yaml

Run:

```bash
rm ~/infernet-container-starter/deploy/docker-compose.yaml && nano ~/infernet-container-starter/deploy/docker-compose.yaml
```

Paste:

```yaml
services:
  node:
    image: ritualnetwork/infernet-node:1.4.0
    ports:
      - "0.0.0.0:4000:4000"
    volumes:
      - ./config.json:/app/config.json
      - node-logs:/logs
      - /var/run/docker.sock:/var/run/docker.sock
    tty: true
    networks:
      - network
    depends_on:
      - redis
      - infernet-anvil
    restart:
      on-failure
    extra_hosts:
      - "host.docker.internal:host-gateway"
    stop_grace_period: 1m
    container_name: infernet-node

  redis:
    image: redis:7.4.0
    ports:
      - "6379:6379"
    networks:
      - network
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
      - redis-data:/data
    restart:
      on-failure
    container_name: infernet-redis

  fluentbit:
    image: fluent/fluent-bit:3.1.4
    expose:
      - "24224"
    environment:
      - FLUENTBIT_CONFIG_PATH=/fluent-bit/etc/fluent-bit.conf
    volumes:
      - ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf
      - /var/log:/var/log:ro
    networks:
      - network
    restart:
      on-failure
    container_name: infernet-fluentbit

  infernet-anvil:
    image: ritualnetwork/infernet-anvil:1.0.0
    command: --host 0.0.0.0 --port 3000 --load-state infernet_deployed.json -b 1
    ports:
      - "8545:3000"
    networks:
      - network
    container_name: infernet-anvil

networks:
  network:

volumes:
  node-logs:
  redis-data:
```

Save: Ctrl+X → Y → Enter

---

## Stop Docker Compose (if running)

```bash
cd $HOME
docker compose -f infernet-container-starter/deploy/docker-compose.yaml stop
```

---

## Installing Foundry

```bash
curl -L https://foundry.paradigm.xyz | bash
echo 'export PATH="$HOME/.foundry/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
foundryup
```

---

## Install libraries (forge-std & infernet-sdk)

```bash
cd ~/infernet-container-starter/projects/hello-world/contracts && \
rm -rf lib/forge-std lib/infernet-sdk && \
forge install foundry-rs/forge-std && \
forge install ritual-net/infernet-sdk && \
ls lib/forge-std && \
ls lib/infernet-sdk
```

---

## Initialize & Start Docker Compose

```bash
cd $HOME
docker compose -f infernet-container-starter/deploy/docker-compose.yaml up -d
```

👉 Verify that containers are running:

```bash
docker ps
```

(Expect node, redis, fluentbit, infernet-anvil, plus any project containers)

If any container fails, view logs:

```bash
docker compose -f infernet-container-starter/deploy/docker-compose.yaml logs -f infernet-node
```

---

## Deploy Consumer Contract (SaysGM)

```bash
cd ~/infernet-container-starter
project=hello-world make deploy-contracts
```

When deploy finishes, save the deployed contract address shown in the output.

---

## Call Contract

1. Open call script:

```bash
nano ~/infernet-container-starter/projects/hello-world/contracts/script/CallContract.s.sol
```

2. Edit the script to set the SaysGM deployed contract address where indicated. Save: Ctrl+X → Y → Enter

3. Run:

```bash
project=hello-world make call-contract
```

You should see transactions (if using a Mainnet RPC + funded account).

---

## Next-Day Start (Restart)

```bash
docker compose -f infernet-container-starter/deploy/docker-compose.yaml down
docker compose -f infernet-container-starter/deploy/docker-compose.yaml up -d
```

---

## Delete Whole Node Data (Reset & Start Over)

If something went wrong and you want to start from scratch:

```bash
cd $HOME
docker compose -f infernet-container-starter/deploy/docker-compose.yaml stop
sudo rm -rf ~/infernet-container-starter
# Then re-clone the starter repo and follow steps again
```

---

## Useful Notes & Tips

- Replace placeholders: ENTER_API_KEY and ENTER_YOUR_PVT_KEYS (use `0x...`) before starting.  
- Do NOT commit private keys or secrets to Git. Use environment variables or a .env file with proper permissions.  
- If `forge` is not found, ensure Foundry PATH has been added and terminal restarted or `source ~/.bashrc`.  
- Use `docker compose -f ... logs -f <service>` to tail logs for debugging.  
- If permission errors with Docker: ensure your user is in the `docker` group or run commands with `sudo`.

---



---

Thank you — Happy Coding 💗
