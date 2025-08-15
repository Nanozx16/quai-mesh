# quai-mesh

Mesh Ethereum
This repository contains a sample implementation of Mesh API for the Ethereum blockchain.

Build Status    

Build once. Integrate your blockchain everywhere.

MESH-ETHEREUM IS CONSIDERED ALPHA SOFTWARE. USE AT YOUR OWN RISK.

This project is available open source under the terms of the [Apache 2.0 License](https://opensource.org/licenses/Apache-2.0).

Overview
The mesh-ethereum repository provides an implementation sample of the Mesh API for Ethereum in Golang. We created this repository for developers of Ethereum-like (a.k.a., account-based) blockchains, who may find it easier to fork this implementation sample than write one from scratch.

Mesh is an open-source specification and set of tools that makes integrating with blockchains simpler, faster, and more reliable. The Mesh API is specified in the OpenAPI 3.0 format.

You can craft requests and responses with auto-generated code using Swagger Codegen or OpenAPI Generator. These requests and responses must be human-readable (easy to debug and understand), and able to be used in servers and browsers.

Jump to:

How to Use This Repo
Docker Deployment
Testing
Documentation
Related Projects
How to Use This Repo
Fork the repo.
Start playing with the code.
Deploy a node in Docker to begin testing.
System Requirements
RAM: 16 MB minimum

We tested mesh-ethereum on an AWS c5.2xlarge instance. This instance type has 8 vCPU and 16 GB of RAM. If you use a computer with less than 16 GB of RAM, it is possible that mesh-ethereum will exit with an OOM error.

Network Settings
To increase the load mesh-ethereum can handle, we recommend these actions:

Tune your OS settings to allow for more connections. On a linux-based OS, run the following commands to do so (source):
sysctl -w net.ipv4.tcp_tw_reuse=1
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
sysctl -w net.ipv4.tcp_max_syn_backlog=10000
sysctl -w net.core.somaxconn=10000
sysctl -p (when done)
We have not tested mesh-ethereum with net.ipv4.tcp_tw_recycle and do not recommend enabling it.

Modify your open file settings to 100000. You can do this on a linux-based OS with the command: ulimit -n 100000.
Memory-Mapped Files
mesh-ethereum uses memory-mapped files to persist data in the indexer. As a result, you must run mesh-ethereum on a 64-bit architecture (the virtual address space easily exceeds 100s of GBs).

If you receive a kernel OOM, you may need to increase the allocated size of swap space on your OS. There is a great tutorial for how to do this on Linux here.

Features
Comprehensive tracking of all ETH balance changes
Stateless, offline, curve-based transaction construction (with address checksum validation)
Atomic balance lookups using go-ethereum's GraphQL Endpoint
Idempotent access to all transaction traces and receipts
Development
Helpful commands for development:

Install dependencies
make deps
Run Tests
make test
Lint the Source Code
make lint
Security Check
make salus
Build a Docker Image from the Local Context
make build-local
Generate a Coverage Report
make coverage-local
Image Installation
Running the following commands will create a Docker image called mesh-ethereum:latest.

Installing from GitHub
To download the pre-built Docker image from the latest release, run:

curl -sSfL https://raw.githubusercontent.com/coinbase/mesh-ethereum/master/install.sh | sh -s
Do not try to install mesh-ethereum using GitHub Packages!

Installing from Source
After cloning this repository, run:

make build-local
Configuring the Environment Variables
Required Arguments
MODE Type: String Options: ONLINE, OFFLINE Default: None

MODE determines if Mesh can make outbound connections.

NETWORK Type: String Options: MAINNET, ROPSTEN, RINKEBY, GOERLI or TESTNET Default: ROPSTEN, but only for backwards compatibility if you use TESTNET

NETWORK is the Ethereum network to launch or communicate with.

PORT Type: Integer Options: 8080, any compatible port number Default: None

PORT is the port to use for Mesh.

Optional Arguments
GETH Type: String Options: A node URL Default: None

GETH points to a remote geth node instead of initializing one

SKIP_GETH_ADMIN Type: Boolean Options: TRUE, FALSE Default: FALSE

SKIP_GETH_ADMIN instructs Mesh to not use the geth admin RPC calls. This is typically disabled by hosted blockchain node services.

Run Docker
Running the commands below will start a Docker container in detached mode, with a data directory at <working directory>/ethereum-data and the Mesh API accessible at port 8080.

Example Commands
You can run these commands from the command line. If you cloned the repository, you can use the make commands shown after the examples.

Mainnet:Online

Uncloned repo:

docker run -d --rm --ulimit "nofile=100000:100000" -v "$(pwd)/ethereum-data:/data" -e "MODE=ONLINE" -e "NETWORK=MAINNET" -e "PORT=8080" -p 8080:8080 -p 30303:30303 mesh-ethereum:latest
Cloned repo:

make run-mainnet-online
Mainnet:Online (Remote)

Uncloned repo:

docker run -d --rm --ulimit "nofile=100000:100000" -e "MODE=ONLINE" -e "NETWORK=MAINNET" -e "PORT=8080" -e "GETH=<NODE URL>" -p 8080:8080 -p 30303:30303 mesh-ethereum:latest
Cloned repo:

make run-mainnet-remote geth=<NODE URL>
Mainnet:Offline

Uncloned repo:

docker run -d --rm -e "MODE=OFFLINE" -e "NETWORK=MAINNET" -e "PORT=8081" -p 8081:8081 mesh-ethereum:latest
Cloned repo:

make run-mainnet-offline
Testnet:Online

Uncloned repo:

docker run -d --rm --ulimit "nofile=100000:100000" -v "$(pwd)/ethereum-data:/data" -e "MODE=ONLINE" -e "NETWORK=TESTNET" -e "PORT=8080" -p 8080:8080 -p 30303:30303 mesh-ethereum:latest
Cloned repo:

make run-testnet-online
// to send some funds into your testnet account you can use the following commands
geth attach http://127.0.0.1:8545
> eth.sendTransaction({from: eth.coinbase, to: "0x9C639954BC9956598Df734994378A36f73cfba0C", value: web3.toWei(50, "ether")})
Testnet:Online (Remote)

Uncloned repo:

docker run -d --rm --ulimit "nofile=100000:100000" -e "MODE=ONLINE" -e "NETWORK=TESTNET" -e "PORT=8080" -e "GETH=<NODE URL>" -p 8080:8080 -p 30303:30303 mesh-ethereum:latest
Cloned repo:

make run-testnet-remote geth=<NODE URL>
Testnet:Offline

Uncloned repo:

docker run -d --rm -e "MODE=OFFLINE" -e "NETWORK=TESTNET" -e "PORT=8081" -p 8081:8081 mesh-ethereum:latest
Cloned repo:

make run-testnet-offline
Test the Implementation with mesh-cli
To validate mesh-ethereum, install mesh-cli and run one of the following commands:

mesh-cli check:data --configuration-file mesh-cli-conf/testnet/config.json - This command validates that the Data API implementation is correct using the ethereum testnet node. It also ensures that the implementation does not miss any balance-changing operations.
mesh-cli check:construction --configuration-file mesh-cli-conf/testnet/config.json - This command validates the Construction API implementation. It also verifies transaction construction, signing, and submissions to the testnet network.
mesh-cli check:data --configuration-file mesh-cli-conf/mainnet/config.json - This command validates that the Data API implementation is correct using the ethereum mainnet node. It also ensures that the implementation does not miss any balance-changing operations.
Read the How to Test your Mesh Implementation documentation for additional details.

Contributing
You may contribute to the mesh-ethereum project in various ways:

Asking Questions
Providing Feedback
Reporting Issues
Read our Contributing documentation for more information.

You can also find community implementations for a variety of blockchains in the mesh-ecosystem repository.

Documentation
You can find the Mesh API documentation here.

Check out the Getting Started section to start diving into Mesh.

Related Projects
mesh-geth-sdk — This SDK helps accelerate Mesh API implementation on go-ethereum based chains.
mesh-sdk-go — The mesh-sdk-go SDK provides a collection of packages used for interaction with the Mesh API specification.
mesh-specifications — Much of the SDKs’ code is generated from this repository.
mesh-cli — Use the mesh-cli tool to test your Mesh API implementation. The tool also provides the ability to look up block contents and account balances.
Other Implementation Samples
You can find community implementations for a variety of blockchains in the mesh-ecosystem repository.

License
This project is available open source under the terms of the Apache 2.0 License.

© 2022 Coinbase
