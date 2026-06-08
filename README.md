Local Web3 Infrastructure Stack

A lightweight, containerized local Ethereum (EVM) environment for Web3 development and testing.

What's inside?

This stack runs on Docker Compose and sets up two main services:

1. Local EVM Node (Anvil):

	• Your local testnet. It automatically generates pre-funded test accounts so you can start building immediately.

	• Port: 8545

	• Note: CORS is completely open (--allow-origin "*") so your local frontend apps and explorers won't throw cross-origin errors.

2. Block Explorer (Otterscan):

	• A clean UI to inspect your local chain. It connects directly to the Anvil node so you can visually track blocks, transactions, and state changes.

	• Port: 8080

Quick Start

Start the network in the background:

[bash]
docker compose up -d


Stop and clean up containers:

[bash]
docker compose down


Testing the connection

You can verify everything is working by sending a test transaction (transferring 1 ETH) directly via your terminal:

[bash]
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{"from":"<SOURCE_ADDRESS>","to":"<DESTINATION_ADDRESS>","value":"0xde0b6b3a7640000"}],"id":1}' -H "Content-Type: application/json" http://localhost:8545


After running this, go to http://localhost:8080 in your browser. You should see the network move from block 0 to block 1, and your transaction details will be there.
