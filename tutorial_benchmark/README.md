# Hướng dẫn benchmark cơ bản Hyperledger Caliper trên Besu bằng quorum-dev-quickstart
## Tải docker compose, quorum-dev-quickstart, nodejs 16 về
```bash
docker compose version //Kiểm tra phiên bản docker compose
node -v //Kiểm tra phiên bản node js
npx quorum-dev-quickstart
```
* Trong thư mục config chứa source code Besu, chọn file config.toml và thêm cờ (flag).
```bash
tx-pool-limit-by-account-percentage="1" 
//Giới hạn số tx mỗi tài khoản = 1% mempool
```
* Sau khi thêm cờ vào file config thì chạy docker compose bằng file run.sh.

* Clone github của hyperledger-caliper-benchmarks về, sau đó tải về phiên bản mới nhất của Caliper là 0.5.0.
```bash
npm install --only=prod @hyperledger/caliper-cli@0.5.0
```
* Trong thư mục caliper-benchmarks vừa tạo, tạo một file mới tên là config.json, là file cho Caliper biết cần làm gì.
* Sau khi tạo file json thì nhập thông tin của file json vào **(lưu ý: nhớ bỏ comment vì json không cho phép comment)**.
```json
{
  "caliper": {
    // Khai báo blockchain backend mà Caliper sẽ benchmark
    // Ở đây là "ethereum" (có thể là Besu, GoQuorum, Geth, v.v.)
    "blockchain": "ethereum"
  },
  "ethereum": {
    // Địa chỉ WebSocket endpoint của node Ethereum hoặc Besu
    // Nếu bạn chạy Quorum Dev Quickstart thì thường là ws://localhost:8546
    "url": "ws://localhost:8546",

    // Địa chỉ của tài khoản dùng để deploy smart contract
    "contractDeployerAddress": "0xc0A8e4D217eB85b812aeb1226fAb6F588943C2C2",

    // Private key tương ứng với địa chỉ trên, để ký giao dịch deploy
    "contractDeployerAddressPrivateKey": "0x45a915e4d0601499eb4365960e6a7a45f33493093061116b197e3240065ff2d8",

    // Địa chỉ mặc định được dùng để gửi các giao dịch (TX)
    "fromAddress": "0xc0A8e4D217eB85b812aeb1226fAb6F588943C2C2",

    // Seed dùng để sinh khóa — thường không cần thiết nếu có private key
    "fromAddressSeed": "0x8b12a749f7606741db0a4b5c912e846f346101fce9d7a632431b0bae3fc9060",

    // Private key của tài khoản gửi giao dịch
    "fromAddressPrivateKey": "0x45a915e4d0601499eb4365960e6a7a45f33493093061116b197e3240065ff2d8",

    // Số block cần chờ để Caliper xem giao dịch là đã xác nhận
    // Dùng trong môi trường thật để đảm bảo TX không bị reorg
    "transactionConfirmationBlocks": 12,

    // Khai báo các smart contract mà benchmark sẽ tương tác
    "contracts": {
      // Tên hợp đồng là "simple"
      "simple": {
        // Đường dẫn tới file chứa ABI + bytecode của contract "Simple"
        // File này được tạo sau khi bạn compile bằng Truffle/Hardhat
        "path": "src/ethereum/simple/simple.json",

        // Cấu hình gas limit cho từng loại hành động Caliper sẽ thực hiện
        "gas": {
          // Gas dùng cho hàm "open" (thường là tạo tài khoản / mở kênh)
          "open": 45000,
          // Gas dùng cho hàm "query" (đọc dữ liệu)
          "query": 100000,
          // Gas dùng cho hàm "transfer" (gửi giao dịch)
          "transfer": 70000
        },

        // ABI (Application Binary Interface) — định nghĩa các hàm contract
        // Ở đây để trống [] vì Caliper sẽ đọc từ file simple.json ở trên
        "abi": []
      }
    }
  }
}
```
* Nếu muốn tự deploy smart contract của mình thì bỏ thuộc tính ```path``` đi và thay bằng ```address```. Ở đây mình để mặc định là path tới simple.json
* Để điều chỉnh thông số (parameters) ta vào file config.yaml theo đường dẫn ```benchmarks/scenario/simple/config.yaml```, để mô phổng hệ thống interbank thì mình sẽ điều chỉnh số worker thành 10 **(lưu ý: nên chọn worker dựa theo số nhân của máy nên là nCPU/2 hoặc nCPU/1.5)**.
* Đây là thông số của mình dùng để mô phổng:
```yaml
simpleArgs: &simple-args
  initialMoney: 10000         # Số dư ban đầu của mỗi tài khoản
  moneyToTransfer: 100        # Số tiền mỗi giao dịch chuyển
  numberOfAccounts: &number-of-accounts 1000  # Tổng số tài khoản được tạo

test:
  name: interbank-simulation
  description: >-
    Benchmark mô phỏng giao dịch liên ngân hàng trên mạng consortium Besu bằng Hyperledger Caliper.
    Bao gồm 3 giai đoạn: mở tài khoản, truy vấn, và chuyển tiền giữa các ngân hàng.
  workers:
    number: 10  # 10 worker ~ 10 ngân hàng song song (tối ưu cho CPU 16 nhân)

  rounds:
    # 🏦 Vòng 1: mở tài khoản
    - label: open
      description: >-
        Tạo 1000 tài khoản cho các ngân hàng, mỗi tài khoản có số dư ban đầu 10,000.
      txNumber: *number-of-accounts
      rateControl:
        type: fixed-rate
        opts:
          tps: 200           # Tổng TPS = 200 (chia đều cho 10 worker, mỗi worker ~20 TPS)
      workload:
        module: benchmarks/scenario/simple/open.js
        arguments: *simple-args

    # 📊 Vòng 2: truy vấn tài khoản
    - label: query
      description: >-
        Đo hiệu năng truy vấn số dư tài khoản trong mạng consortium.
      txNumber: *number-of-accounts
      rateControl:
        type: fixed-rate
        opts:
          tps: 400           # Tổng TPS = 400 (mỗi worker ~40 TPS)
      workload:
        module: benchmarks/scenario/simple/query.js
        arguments: *simple-args

    # 💸 Vòng 3: chuyển tiền liên ngân hàng
    - label: transfer
      description: >-
        Mô phỏng chuyển tiền giữa các ngân hàng, mỗi giao dịch 100 đơn vị tiền.
      txNumber: 500
      rateControl:
        type: fixed-rate
        opts:
          tps: 50            # Tổng TPS = 50 (mỗi worker ~5 TPS)
      workload:
        module: benchmarks/scenario/simple/transfer.js
        arguments:
          << : *simple-args
          money: 100
```