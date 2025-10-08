# Hướng dẫn triển khai Private Besu Network và Benchmark với Hyperledger Caliper
## 1️⃣ Tạo Private Besu Network với Docker Compose
### 1.1 Cấu trúc thư mục
```bash
Besu_private_network_learning/
├─ caliper-benchmark/
│  ├─ benchmark.yaml
│  ├─ network.yaml
│  └─ simpleTransfer.js
├─ data-central/
├─ data-bank-a/
├─ data-bank-b/
├─ data-bank-c/
├─ data-bank-d/
└─ docker-compose.yml

```

### 1.2 docker-compose.yml mẫu (5 node: central + 4 bank)
```bash
version: '3.7'
services:
  central-bank:
    image: hyperledger/besu:latest
    container_name: central-bank
    command: >
      besu --network-id 1337
           --genesis-file=/opt/besu/genesis.json
           --data-path=/var/lib/besu
           --rpc-http-enabled
           --rpc-http-host=0.0.0.0
           --rpc-http-port=8545
           --rpc-ws-enabled
           --rpc-ws-port=8546
           --host-allowlist=*
           --min-gas-price=0
    ports:
      - 8545:8545
      - 8546:8546
      - 30303:30303
    volumes:
      - ./data-central:/var/lib/besu
      - ./genesis.json:/opt/besu/genesis.json

  bank-a:
    image: hyperledger/besu:latest
    container_name: bank-a
    command: >
      besu --network-id 1337
           --genesis-file=/opt/besu/genesis.json
           --data-path=/var/lib/besu
           --rpc-http-enabled
           --rpc-http-port=8545
           --host-allowlist=*
    ports:
      - 8547:8545
      - 30304:30303
    volumes:
      - ./data-bank-a:/var/lib/besu
      - ./genesis.json:/opt/besu/genesis.json

  bank-b:
    image: hyperledger/besu:latest
    container_name: bank-b
    command: >
      besu --network-id 1337
           --genesis-file=/opt/besu/genesis.json
           --data-path=/var/lib/besu
           --rpc-http-enabled
           --rpc-http-port=8545
           --host-allowlist=*
    ports:
      - 8548:8545
      - 30305:30303
    volumes:
      - ./data-bank-b:/var/lib/besu
      - ./genesis.json:/opt/besu/genesis.json

  bank-c:
    image: hyperledger/besu:latest
    container_name: bank-c
    command: >
      besu --network-id 1337
           --genesis-file=/opt/besu/genesis.json
           --data-path=/var/lib/besu
           --rpc-http-enabled
           --rpc-http-port=8545
           --host-allowlist=*
    ports:
      - 8549:8545
      - 30306:30303
    volumes:
      - ./data-bank-c:/var/lib/besu
      - ./genesis.json:/opt/besu/genesis.json

  bank-d:
    image: hyperledger/besu:latest
    container_name: bank-d
    command: >
      besu --network-id 1337
           --genesis-file=/opt/besu/genesis.json
           --data-path=/var/lib/besu
           --rpc-http-enabled
           --rpc-http-port=8545
           --host-allowlist=*
    ports:
      - 8550:8545
      - 30307:30303
    volumes:
      - ./data-bank-d:/var/lib/besu
      - ./genesis.json:/opt/besu/genesis.json
```
### 1.3 Genesis file (genesis.json) mẫu
```
{
  "config": {
    "chainId": 1337,
    "constantinopleFixBlock": 0,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "clique": {
      "period": 2,
      "epoch": 30000
    }
  },
  "nonce": "0x0",
  "timestamp": "0x5F5B8D80",
  "extraData": "0x",
  "gasLimit": "0x47b760",
  "difficulty": "0x1",
  "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "coinbase": "0x0000000000000000000000000000000000000000",
  "alloc": {
    "0xSenderAddress": { "balance": "1000000000000000000000" },
    "0xReceiverAddress": { "balance": "1000000000000000000000" }
  }
}
```
* **Lưu ý: Replace 0xSenderAddress và 0xReceiverAddress bằng account bạn tạo ra.**

## 2️⃣ Chạy network
```bash
docker-compose up -d
```
Kiểm tra container đang chạy:
```bash
docker ps
```

## 3️⃣ Cài đặt Hyperledger Caliper
```bash
npm init -y
npm install --save-dev @hyperledger/caliper-cli @hyperledger/caliper-core
```
## 4️⃣ Bind Caliper với Besu
```bash
npx caliper bind --caliper-bind-sut besu:latest
```
## 5️⃣ Tạo benchmark
* Vào folder Caliper-Benchmark
### 5.1 benchmark.yaml
```bash
name: simple-transfer
description: Simple ETH transfer
workers:
  type: local
  number: 1
rounds:
  - label: transfer
    txNumber: 10
    rateControl:
      type: fixed-rate
      opts:
        tps: 1
    workload:
      module: ./simpleTransfer.js
      arguments:
        value: 1000000000000000000
```
### 5.2 network.yaml
```bash
name: besu-network
caliper:
  blockchain: besu
  version: latest
  smartContract:
    language: solidity
    file: ./contracts/SimpleToken.sol
    name: SimpleToken
    version: 1.0
    constructor:
      args: []
    artifact: build/contracts/SimpleToken.json
  deploy:
    init:
      txTimeout: 300
nodes:
  central:
    url: http://localhost:8545
  bank-a:
    url: http://localhost:8547
  bank-b:
    url: http://localhost:8548
  bank-c:
    url: http://localhost:8549
  bank-d:
    url: http://localhost:8550
```
## 6️⃣ Tạo simpleTransfer.js
```bash
'use strict';

module.exports.info  = 'Simple ETH transfer benchmark';

module.exports.init = async function(blockchain, context, args) {
    return;
};

module.exports.run = async function(blockchain, context, args) {
    let tx = {
        from: '0xSenderAddress', // replace bằng account sender
        to: '0xReceiverAddress', // replace bằng account receiver
        value: args.value,
        gas: 21000
    };
    await blockchain.sendTransaction(tx);
};

module.exports.end = async function(blockchain, context, args) {
    return;
};
```
## 7️⃣ Chạy benchmark
```bash
npx caliper launch manager \
  --caliper-benchconfig caliper-benchmark/benchmark.yaml \
  --caliper-networkconfig caliper-benchmark/network.yaml
```
* Nếu xuất hiện lỗi Cannot find module 'besu', chắc chắn bạn đã chạy npx caliper bind --caliper-bind-sut besu:latest trong cùng thư mục.
* Mọi node phải đang Up và healthy.
