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

* Clone github của hyperledger-caliper-benchmarks về, sau đó tải về phiên bản của Caliper là 0.5.0. Nếu trong git clone không có folder src/ethereum thì tải bản released.
* Tiếp đến cái đặt caliper bằng npm: 
```bash
npm install --only=prod @hyperledger/caliper-cli@0.5.0
```
* Trong thư mục caliper-benchmarks vừa tạo, tạo một file mới tên là config.json(là file cho Caliper biết cần làm gì) nếu bạn muốn tự config network cho caliper.
* Sau khi tạo file json thì nhập thông tin của file json vào **(lưu ý: nhớ bỏ comment vì json không cho phép comment)**.
* Trong phần hướng dẫn này thì mình sẽ sử dụng file thì mình chỉ làm benchmarks trong scenario là simple.
```bash
npx caliper launch manager --caliper-workspace . --caliper-networkconfig ../caliper-benchmarks-0.5.0/networks/besu/1node-clique/networkconfig.json --caliper-benchconfig ../caliper-benchmarks-0.5.0/benchmarks/scenario/simple/config.yaml --caliper-bind-sut ethereum
```
* Sau khi chạy lệnh thì caliper sẽ đo các thông số như Max/min latency, Throughput(TPS) sau đó xuất ra file report.html.

