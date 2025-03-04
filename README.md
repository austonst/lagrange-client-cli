# Lagrange CLI

This CLI app provides the functionalities needed to run the a Lagrange Attestation Node.

## Lagrange Labs State Committees Attestation Node

Lagrange Labs State Committees provide a mechanism for generating succinct zero-knowledge state proofs for optimistic rollups based on the use of either staked or restaked collateral. Each state committee is a group of attestors/validators that have either staked an optimistic rollup’s native token or dualstaked with EigenLayer. Each state committee node attests to the execution and finality of transaction batches submitted by optimistic sequencers to Ethereum.

Whenever a batch consisting of rollup blocks is considered either safe (OP stack) or has had its corresponding transaction batch settled on Ethereum (Mainnet), each node is required to attest to the batch of blocks using its BLS key.

Broadly, each signature is executed on a tuple containing 3 essential elements:

```
struct batch {
    var batch_header,
    var current_committee,
    var next_committee
}
```

For a full breakdown of the architecture, please refer to the below two documents:

1. [Lagrange Technical Overview Docs](https://lagrange-labs.gitbook.io/lagrange-labs/lagrange-state-committees/commitees-overview)
2. [Lagrange State Committee Deep Dive](https://hackmd.io/@lagrange/lagrange-committee)

## Running a Lagrange Client Node

The below commands will allow a developer to run a node and attest to the state of `Optimism` and `Base` chains. We are operating on two networks:
  - Holesky Testnet
  - Mainnet

### Chains

- Optimism: `10`
- Base: `8453`

### Minimum Recommended Hardware

- VCPUs: `2`
- RAM: `4 GB`
- Storage: `8 GB`
- AWS instance type: `t3.medium`
- Architecture: 64-bit ARM instance

> NOTE: The commands in this README file are for 64-bit ARM AWS ubuntu instance.

### Pre-requisite Installations

1. Golang

    ```bash
    sudo apt-get update
    sudo apt-get -y upgrade
    wget https://go.dev/dl/go1.21.9.linux-amd64.tar.gz
    sudo tar -xvf go1.21.9.linux-amd64.tar.gz -C /usr/local
    echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
    export GOROOT=/usr/local/go
    source ~/.profile
    go version
    ```

2. Docker and Docker Compose

    ```bash
    sudo apt-get update
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo "deb [arch=\"$(dpkg --print-architecture)\" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin make gcc
    echo '{ "log-opts": { "max-size": "10m", "max-file": "100" } }' | sudo tee /etc/docker/daemon.json
    sudo usermod -aG docker $USER
    newgrp docker
    sudo systemctl restart docker
    ```

## Steps

1. Send your EigenLayer operator Ethereum address to Lagrange Labs team for allowlisting. ([Form](https://docs.google.com/forms/d/1oJq1QddKb1Sa_Pe_C1Sa-9p_jN4jBEV2ARI3-M8yb8c/edit))

2. Clone the [Lagrange CLI repository](https://github.com/Lagrange-Labs/client-cli) to your machine.

    ```bash
    git clone https://github.com/Lagrange-Labs/client-cli.git
    ```

3. Set environment variables and download dependencies.

    ```bash
    export CGO_CFLAGS="-O -D__BLST_PORTABLE__"
    export CGO_CFLAGS_ALLOW="-O -D__BLST_PORTABLE__"
    cd client-cli
    go mod download
    ```

4. Create a go binary.

    ```bash
    sudo apt install make gcc

    make build
    ```

5. Create a configuration file for the CLI. Please refer to `config.toml` for the configuration file.

  - `EthereumRPCURL`: Ethereum mainnet RPC endpoint for mainnet, and Holesky RPC endpoint for testnet.
  - `L1RPCEndpoint`: Ethereum mainnet RPC endpoint for both mainnet and Holesky testnet.
  - `BeaconURL`: Beacon mainnet RPC endpoint for both mainnet and Holesky testnet.
  - `L2RPCEndpoint`: Rollup (`Optimism` or `Base`) chain's mainnet RPC endpoint for both mainnet and Holesky testnet. 

    > NOTE: Currently, we only support the `BN254` curve for the `BLSScheme`.

### Commands

- Generate Keystore: generates a new key pair for the given key type and password, and saves it in the keystore file. The key type can be either `ecdsa` or `bls`.

    ```bash
    lagrange-cli generate-keystore -t <Key Type> -p <Password>

    # i.e. ./dist/lagrange-cli generate-keystore -t ecdsa -p password
    ```
- Import Keystore: imports a key pair from the given private key and saves it in the keystore file. The key type can be either `ecdsa` or `bls`.

    ```bash
    lagrange-cli import-keystore -t <Key Type> -p <Password> -k <Private Key>

    # i.e. ./dist/lagrange-cli import-keystore -t bls -p password -k 0x1234567890abcdef...
    ```
- Export Keystore: exports a private key from the keystore file for the given key type and password.

    ```bash
    lagrange-cli export-keystore -t <Key Type> -p <Password> -f <Keystore File Path>

    # i.e. ./dist/lagrange-cli export-keystore -t ecdsa -p password -f ~/.lagrange/keystore/bls_.key
    ```

- Register Operator: registers an operator to the Lagrange State Committee. The network name can be either `mainnet` or `holesky`. The BLS key and Signer address are referenced from the config file.

    ```bash
    lagrange-cli register-operator -c <Config File Path> -n <Network Name>

    # i.e. ./dist/lagrange-cli register-operator -c ./config.toml -n mainnet
    ```
- Deregsiter Operator: deregisters an operator from the Lagrange State Committee. The network name can be either `mainnet` or `holesky`.

    ```bash
    lagrange-cli deregister-operator -c <Config File Path> -n <Network Name>

    # i.e. ./dist/lagrange-cli deregister-operator -c ./config.toml -n mainnet
    ```

- Update BLS Public Key: updates the BLS public key for the operator at the given index. The network name can be either `mainnet` or `holesky`. New BLS key is referenced from the config file.

    ```bash
    lagrange-cli update-bls-pub-key -c <Config File Path> -n <Network Name> -i <Key Index>

    # i.e. ./dist/lagrange-cli update-bls-pub-key -c ./config.toml -n mainnet -i 0
    ```

- Update Signer Address: updates the signer address for the operator. The network name can be either `mainnet` or `holesky`. New signer address is referenced from the config file.

    ```bash
    lagrange-cli update-signer-address -c <Config File Path> -n <Network Name>

    # i.e. ./dist/lagrange-cli update-signer-address -c ./config.toml -n mainnet
    ```

- Add BLS Public Key: adds a new BLS public key for the operator. The network name can be either `mainnet` or `holesky`. New BLS key is referenced from the config file.

    ```bash
    lagrange-cli add-bls-pub-key -c <Config File Path> -n <Network Name>

    # i.e. ./dist/lagrange-cli add-bls-pub-key -c ./config.toml -n mainnet
    ```

- Remove BLS Public Key: removes the BLS public key for the operator at the given index. The network name can be either `mainnet` or `holesky`.

    ```bash
    lagrange-cli remove-bls-pub-key -c <Config File Path> -n <Network Name> -i <Key Index>

    # i.e. ./dist/lagrange-cli remove-bls-pub-key -c ./config.toml -n mainnet -i 0
    ```

- Subscribe Chain: subscribes the operator to the given chain. The network name can be either `mainnet` or `holesky`. The chain name can be either `optimism` or `base`.

    ```bash
    lagrange-cli subscribe-chain -c <Config File Path> -n <Network Name> -r <Chain Name>

    # i.e. ./dist/lagrange-cli subscribe-chain -c ./config.toml -n mainnet -r optimism
    ```

- Unsubscribe Chain: unsubscribes the operator from the given chain. The network name can be either `mainnet` or `holesky`. The chain name can be either `optimism` or `base`.

    ```bash
    lagrange-cli unsubscribe-chain -c <Config File Path> -n <Network Name> -r <Chain Name>

    # i.e. ./dist/lagrange-cli unsubscribe-chain -c ./config.toml -n mainnet -r optimism
    ```

- Generate Config: generates a client config file. The network name can be either `mainnet` or `holesky`. The chain name can be either `optimism` or `base`. The L1 RPC endpoint is the Ethereum mainnet RPC endpoint for both mainnet and Holesky testnet. The L2 (`Optimism` or `Base`) RPC endpoint is the rollup chain's mainnet RPC endpoint for both mainnet and Holesky testnet. The Beacon RPC endpoint is the Beacon mainnet RPC endpoint for both mainnet and Holesky testnet.

    ```bash
    lagrange-cli generate-config -c <Config File Path> -n <Network Name> -r <Chain Name>

    # i.e. ./dist/lagrange-cli generate-config -c ./config.toml -n mainnet -r optimism
    ```

- Deploy Node: creates a docker-compose file and deploys the docker container for the attestation node. The network name can be either `mainnet` or `holesky`.

    ```bash
    lagrange-cli deploy -c <Client Config File Path> -i <Docker Image Name>

    # i.e. ./dist/lagrange-cli deploy -c ~/.lagrange/config/client_mainnet_optimism_.toml -i lagrangelabs/lagrange-node:v0.3.13
    ```

> NOTE: Please take a backup of your BLS and ECDSA key pairs. They are located at `~/.lagrange/keystore/<key_type>_<public_key_prefix>.key`.

> NOTE: We recommend using trusted providers like Alchemy, Quicknode, Infura if you don't run your own node. For the Beacon RPC endpoint, you should check if the given RPC provider supports the Beacon RPC API like `/eth/v1/beacon/genesis`. Quicknode supports this API.

> NOTE: You can check client config files in the `~/.lagrange/config/client_<network_name>_<chain_name>_<bls_pub_key_prefix>.toml` and docker-compose files in the `~/.lagrange/docker-compose_<network_name>_<chain_name>_<bls_pub_key_prefix>.yml`.

### Post-Deployment

- To check the status of the attestation node from the docker container logs, run the below command:

```bash
# to get the container id
docker ps

# if you are not seeing any running containers, then run the below command to see all suspended containers
docker ps -a

# to check the logs of the container
docker logs <container_id>

# to stop the container
cd $HOME/.lagrange && docker compose -f <docker-compose-file> down --remove-orphans

```

- If you experience the rpc provider issue, it can be due to the rate limit of the provider. You can control the `ConcurrentFetchers` in the `config.toml` file to manage the rate limit.

- If you face any issues while running the Lagrange Attestation Node, please reach out to the Lagrange Labs team on Telegram.
