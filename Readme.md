# TRON Bep20 Payment Gateway

Cung cấp kênh thanh toán trên mạng TRON Smart Chain ứng với các loại token Trc20.

## Mục lục

- [Create Account](#create-account)
- [Config](#config)
- [Deployment](#deployment)
- [API](#api)
    * [Complete Player Deposit Transaction Redirect URL](#complete-player-deposit-transaction-redirect-url)
    * [Complete Transferred Tokens Redirect URL](#complete-transferred-tokens-redirect-url)
    * [Get User Address](#get-user-address)
    * [User Count](#user-count)
    * [Get User ID](#get-user-id)
    * [Get User Deposit Transactions Count](#get-user-deposit-transactions-count)
    * [Get User Completed Deposit Transactions](#get-user-completed-deposit-transactions)
    * [Get User Pending Deposit Transactions](#get-user-pending-deposit-transactions)
    * [Get User Withdraw Transactions Count](#get-user-withdraw-transactions-count)
    * [Get User Completed Withdraw Transactions](#get-user-completed-withdraw-transactions)
    * [Get User Pending Withdraw Transactions](#get-user-pending-withdraw-transactions)
    * [Withdraw](#withdraw)
    * [Check Deposit Transaction](#check-deposit-transaction)
    * [Get Deposit Transaction By Tx Hash](#get-deposit-transaction-by-tx-hash)
    * [Get Withdraw Transaction By Tx Hash](#get-withdraw-transaction-by-tx-hash)
    * [Model](#model)
        + [DepositTransaction](#deposittransaction)
        + [WithdrawTransaction](#withdrawtransaction)

<small><i><a href='https://github.com/dangnguyendota'>email: dangnguyendota@gmail.com</a></i></small>

## Create Account 

Tạo tài khoản tronGrid để lấy API Key tại trang sau: https://www.trongrid.io/

**Địa chỉ:**
 - mainnet: https://api.trongrid.io
 - shasta testnet: https://api.shasta.trongrid.io
 - nile testnet: https://nile.trongrid.io

## Config

Trong thư mục `configs/` chứa file `config.example.yaml` là mẫu config, chứa các trường cần có.
Copy file `config.example.yaml` sang file `config.yaml` và sửa lại các trường này.

`config.example.yaml`

```yaml
log: ./logs
amqpURI: "amqp://guest:guest@trx-rabbitmq:5672/"
mongoDNS: "mongodb://trx:trx@trx-mongodb:27017"
hotWalletMnemonic: "order eyebrow caution endorse feed man suffer hurry picnic praise day market"
numberOfConfirmationsToApproveDepositTx: 15
tokens:
  - name: usdt
    decimals: 6
    address: "TXLAQ63Xg1NAzckPwKHvzw7CSEmLMEqcdj"
    approveAmount: 1000000
    minToCollect: 0.001
trongridEndpoint:  https://nile.trongrid.io
trongridAPIKey: a446c38c-8258-4d7f-b4b0-0f917193b8bd
APIPostDepositTransactionURL: https://example.com/user-deposited
APIPostTokensTransferredURL: https://example.com/transferred-tokens
ipWhiteList:
  - localhost
  - 192.168.1.0
APIKey: example-key
password: example-password
watcherBlockRange: 200
```

**Giải thích các trường:**

- _log_ `string` đường dẫn tới thư mực lưu log. (default: `./logs`)
- _amqpURI_ `string` uri connect với rabbitmq, default là `amqp://guest:guest@trx-rabbitmq:5672/` là uri của
  rabbitmq chạy trong trx service, nếu muốn dùng rabbitmq khác chạy bên ngoài thì đặt lại trường này.
- _mongoDNS_ `string` uri tới mongodb, default là `mongodb://trx:trx@trx-mongodb:27017` là uri tới
  mongodb chạy trong trx service, nếu muốn móc sang mongo db khác chạy bên ngoài thì sửa lại trường này.
- _hotWalletMnemonic_ `string` chuỗi mnemonic (hay trên metamask gọi là **Cụm mật khẩu khôi phục bí mật**) của ví tổng,
  ví dụ 1 mnemonic: `mistake wet reward valid gesture drop delay soft say same energy upon`
- _numberOfConfirmationsToApproveDepositTx_ `int` số lượng tối thiểu xác nhận để giao dich có thể coi là hợp lệ, số này
  càng nhỏ thì khả năng cheat của user càng cao, thường sẽ từ 15 trở lên, tuy nhiên nếu để quá cao có thể thời gian của
  người chơi để chờ xác nhận giao dịch sẽ lâu hơn nhiều.
- _tokens_ `array` là mảng các loại token được hỗ trợ
    - _name_ `string` tên duy nhất của token (do mình tự đặt) (ví dụ: usdt, usdc, bnb, busd,...)
    - _decimals_ `int` decimal của token.
    - _address_ `string` địa chỉ của token.
    - _approveAmount_ `int` số token approve cho ví con để ví chính sử dụng chuyển tiền từ ví con đó sang ví chính (để số lớn)
    - _minToCollect_ `int` số lượng token tối thiểu có trong ví con để **trx** thực hiện rút tiền về ví tổng, bởi
      vì mỗi lần rút sẽ mất khoảng `0.2$` phí giao dịch, nên phải giới hạn rút đối với số lượng tiền tối thiểu, tránh số
      tiền quá nhỏ vẫn thực hiện rút dẫn đến tình trạng tiền không đủ bù phí giao dịch. Một user có thể nạp nhiều lần số
      nhỏ, khi đạt đủ số lượng hệ thống mới thực hiện rút tiền về ví tổng nhằm tiết kiệm chi phí.
- _trongridEndpoint_ `string` rpc endpoint của mạng.
- _trongridAPIKey_ `string` api key của tài khoản dùng cho mainnet.
- _APIPostDepositTransactionURL_ `string` đường dẫn
  tới [API Complete Deposit Transaction Redirect URL](#complete-player-deposit-transaction-redirect-url)
- _APIPostTokensTransferredURL_ `string` đường dẫn tới
  API [API Complete Transferred Tokens Redirect URL](#complete-transferred-tokens-redirect-url)
- _ipWhiteList_ `array` các địa chỉ IP hợp lệ có thể gửi request lên, nếu mảng này rỗng thì sẽ accept request của mọi
  địa chỉ IP.
- _APIKey_ `string` **API Key** của hệ thống, phải khớp ở cả 2 đầu **Andromeda Service** và **Game Service**, đây sẽ là
  mã khóa bí mật không được để tiết lộ ra ngoài.
- _password_ `string` là mật khẩu dùng để xác thực đơn giản trong API, nếu password được gán thì mỗi request đều phải
  gửi lên kèm password trong Header.
- _watcherBlockRange_ `int` số lượng block trên 1 lần crawl (tối đa 200, tối thiểu 1)

## Deployment

Andromeda gồm 3 thành phần chính: Cash In, Cash Out và API.
Để khởi chạy toàn hệ thống:

```shell
sudo docker compose up -d 
```

Hoặc chạy từng phần như sau:

**Bước 1:** Tạo file `config.yaml` trong thư mục `configs/` theo hướng dẫn ở phần trên.
**Bước 2:** Chạy API Service

```shell
sh  deploy-API-service.sh
```

**Bước 3:** Chạy Cash In

```shell
sh deploy-CI-services.sh
```

**Bước 4:** Chạy Cash Out

```shell
sh deploy-CO-services.sh
```

Bước 5: Kiểm tra các service đã chạy lên đầy đủ chưa:

```shell
sudo docker compose ps 
```

Kết quả sẽ hiện lên như sau

```shell
NAME                   COMMAND                  SERVICE             STATUS              PORTS
trx-api          "/usr/bin/supervisor…"   api                 running (healthy)   0.0.0.0:9712->9712/tcp, :::9712->9712/tcp
trx-approver     "/usr/bin/supervisor…"   approver            running (healthy)   
trx-collector    "/usr/bin/supervisor…"   collector           running (healthy)   
trx-dispatcher   "/usr/bin/supervisor…"   dispatcher          running (healthy)   
trx-mongodb      "docker-entrypoint.s…"   mongo               running (healthy)   0.0.0.0:9707->27017/tcp, :::9708->27017/tcp
trx-rabbitmq     "docker-entrypoint.s…"   rabbitmq            running (healthy)   25672/tcp
trx-transactor   "/usr/bin/supervisor…"   transactor          running (healthy)   
trx-validator    "/usr/bin/supervisor…"   validator           running (healthy)   
trx-watcher      "/usr/bin/supervisor…"   watcher             running (healthy)  
```

Tình trạng các service đều ở trạng thái `healthy` là ok.

Port của API sẽ là port `9712` và port của mongo db là `9707` username và password là `trx`.

**Hệ thống gồm các thành phần sau đây:**

- **Cở sở dữ liệu:** mongo db, mở port `9707` ra ngoài, _username_: **trx**, _password_: **trx**.
    - _docker compose service_: mongo
    - _image_: mongo
    - _container_: trx-mongo
    - _run_: ```sh restart-mongodb.sh```
- **Message Broker:** rabbitmq
    - _docker compose service_: rabbitmq
    - _image_: rabbitmq:3.8.25-management-alpine
    - _container_: trx-rabbitmq
    - _run_: ```sh restart-rabbitmq.sh```
- **Cash In:** thực hiện ghi nhận các giao dịch nạp tiền vào hệ thống và bắn thông báo sang Game Service.
    - **module watcher** thực hiện lắng nghe các giao dịch trên mạng.
        - _docker compose service_: watcher
        - _image_: trx/watcher:v1.0.0
        - _container_: trx-watcher
        - _run_: ```sh restart-watcher.sh```
    - **module validator** thực hiện xác thực giao dịch có đúng thuộc user của hệ thống hay không
        - _docker compose service_: validator
        - _image_: trx/validator:v1.0.0
        - _container_: trx-validator
        - _run_: ```sh restart-validator.sh```
    - **module approver** thực hiện kiểm tra giao dịch đã đạt đủ số lượng xác nhận từ các node trên mạng chưa, để tránh
      hack.
        - _docker compose service_: approver
        - _image_: trx/approver:v1.0.0
        - _container_: trx-approver
        - _run_: ```sh restart-approver.sh```
    - **module dispatcher** nhận các yêu cầu vào gửi thông báo qua API sang bên Game Service.
        - _docker compose service_: dispatcher
        - _image_: trx/dispatcher:v1.0.0
        - _container_: trx-dispatcher
        - _run_: ```sh restart-dispatcher.sh```
    - **module collector** thực hiện thu hồi token tự động từ các ví con về ví tổng.
        - _docker compose service_: collector
        - _image_: trx/collector:v1.0.0
        - _container_: trx-collector
        - _run_: ```sh restart-collector.sh```
- **Cash Out:** thực hiện lấy các yêu cầu rút tiền về tài khoản người dùng và chuyển tiền vào ví của người dùng.
    - **module transactor** nhận yêu cầu rút tiền từ hệ thống và thực hiện chuyển tiền sang ví của user.
        - _docker compose service_: transactor
        - _image_: trx/transactor:v1.0.0
        - _container_: trx-transactor
        - _run_: ```sh restart-transactor.sh```
- **API Server**: cung cấp các API để Game Service có thể làm việc với **Andromeda**, port `9712`.
    - _docker compose service_: api
    - _image_: trx/api:v1.0.0
    - _container_: trx-api
    - _run_: ```sh restart-api.sh```
- **Data:** thư mục `data/` chứa dữ liệu của database và rabbitmq, không được thay đổi quyền của thư mục và không được
  xóa thư mục nếu không dữ liệu sẽ bị mất.
- **Log:** thư mục `logs/`chứa log của hệ thống, log sẽ ghi theo ngày và sẽ có tên của module ứng với file log đó. Ví dụ
  file `2022-07-19.api.log` chứa log của API vào ngày 19 tháng 7 năm 2022. File `stdout.log` ghi log std.
- **Dọn dẹp Block:** Nếu hệ thống chạy sai hoặc thay đổi Chain thì phải Clean lại bảng _block_ trong DB
  vì bảng này lưu vị trí block gần nhất crawl được từ Chain, mà mỗi chain thì số block khác nhau nên phải clean để tự
  cập nhật lại, thao tác này có thể dẫn đến mất mát dữ liệu trong khoảng thời gian ngắn.
    - _Run:_ ```sh clean-watched-block.sh```

## API

**Lưu ý:** Nếu Password được set thì khi call API bắt buộc phải đặt trường `Authorization` trong header theo cú
pháp `"Password " + password`.
Ví dụ nếu password là **"example"** thì trường Authorization sẽ là `"Password example"`.

Đối với tất cả các API bên dưới thì sẽ trả về các Status Code chung sau:

- **406** `Not Acceptable` đối với địa chỉ IP không nằm trong white list.
- **401** `Unauthorized` đối với password không được gắn trong Header hoặc không trùng với password hệ thống.

### Complete Player Deposit Transaction Redirect URL

Sau khi phát hiện giao dịch nạp tiền và xác thực giao dịch, Andromeda sẽ gửi thông báo sang
Game Service để thông báo giao dịch nạp tiền đã hoàn tất.

Game Service phải cung cấp API để Andromeda gửi thông báo sang, Andromeda sẽ thực hiện tôi đa 3 lần gọi với mỗi
giao dịch ghi nhận được, nếu vượt quá số lượng trên, gói tin sẽ được lưu trữ vào trong cơ sở dữ liệu.

**Thông tin API:**

- **API URL:** tự khai báo ở trong [Config](#config), ứng với API viết bên **Game Service**. (ví
  dụ: https://game.service.com/api/v1/user-deposited)
- **Method:** POST
- **Status Code:** **202** nếu thành công hoặc khác nếu thất bại, nếu trả về status code khác 202 Adromeda sẽ cố gắng
  gửi lại thêm 2 lần nữa, không thành công sẽ lưu vào bảng failed_request.
- **Body**:
    - _user_id_ `int` ID của người chơi
    - _tx_hash_ `string` mã giao dịch
    - _tokens_ `float` số lượng tokens nạp.
    - _token_name_ `string` tên token.
    - _token_address_ `string` địa chỉ token.
    - _check_sum_ `string` bằng `MD5(API_KEY + TxHash)` dùng để xác thực request này có thật sự tới từ **Andromeda
      Service** không, vì nếu tới từ bên ngoài thì sẽ không thể dò ra được `check_sum` tương ứng với `tx_hash` do không
      thể biết được `API_KEY`.

**Lưu ý:**

- **Game Service** phải lưu lại **tx_hash** vào **database** để kiểm tra xem giao dịch này đã được cập nhật trước đó
  chưa, tránh trường hợp bên thứ 3 tấn công dùng lại gói tin này gửi lên nhằm trục lợi nạp nhiều lần từ 1 giao dịch.
- _API URL_ để là `host.docker.internal` thay cho `localhost` nếu **Andromeda Service** và **Game Service** chạy trong
  cùng 1 máy chủ.
- _API URL_ nên để **HTTPS** để tránh kẻ tấn công có thể lấy được thông tin của gói tin.
- **Bắt buộc** phải kiểm tra **check_sum** để xác thực gói tin này có thật sự tới từ **Andromeda Service** hay không.
- Ví dụ _API Key_ là `abc` và _Tx Hash_ là `0x2d157578aee3908b41192bad9af19432eb683ea872152fa3c72a2ba7a4095abb`  thì
  check_sum sẽ là hàm băm MD5 của chuỗi `abc0x2d157578aee3908b41192bad9af19432eb683ea872152fa3c72a2ba7a4095abb` và
  là `ea16990b87483afaab3a4740a47a2c98`
- Có thể thực hiện kiểm tra địa chỉ IP gửi gói tin này đến **Game Service** để tăng thêm tính bảo mật cho hệ thống.

**!Lưu ý: Mỗi khi nhận được request này từ trx, lưu giao dịch vào database và kiểm tra lại mỗi lần nhận được
request
xem tx_hash này đã được insert trước đó chưa để tránh trường hơp hacker có thể tận dụng lại request để gửi nhiều lần để
chuộc lợi từ 1 giao dịch.**

### Complete Transferred Tokens Redirect URL

Sau khi chuyển tiền thành công vào ví người nhận, trx sẽ gửi thông báo sang 1 API Complete Transferred Tokens
Redirect.

Game Service cung cấp API này để trx thông báo sang, nếu trong [config](#config) để trường _
APIPostTokensTransferredURL_
là rỗng thì trx sẽ không thông báo nữa.

**Cấu trúc:**

- **URL**: định nghĩa trong file config
- **Method**: `POST`
- **Status Code:** **202** nếu thành công hoặc khác nếu thất bại, nếu trả về status code khác 202 Adromeda sẽ cố gắng
  gửi lại thêm 2 lần nữa, không thành công sẽ lưu vào bảng failed_request.
- **Body**:
    - _request_id_ `string` request id mà game service gửi lên trong [API Withdraw](#withdraw)
    - _user_id_ `int` ID của người chơi
    - _tx_hash_ `string` mã giao dịch trên blockchain.
    - _tokens_ `float` số tiền đã gửi.
    - _token_name_ `string` tên tiền.
    - _checksum_ `string`  = MD5(API-Key + Request ID) để kiểm tra độ tin cậy của request.

**!Lưu ý: Mỗi khi nhận được request này từ trx, lưu giao dịch vào database và kiểm tra lại mỗi lần nhận được
request
xem request_id này đã được complete trước đó chưa để tránh trường hơp hacker có thể tận dụng lại request để gửi nhiều
lần để
chuộc lợi từ 1 giao dịch.**

### Get User Address

Lấy địa chỉ ví của user.

```http request
GET /api/v1/user/address/:userID
```

Request:

- _userID_ `int` là id của user cần lấy địa chỉ ví, sẽ trả về lỗi nếu userID <= 0.

Ví dụ lấy địa chỉ ví user có id 10.

```http request
GET /api/v1/user/address/10
```

**Response:**

- _user_id_ `int` ID của user
- _address_ `string` địa chỉ ví

**Status Code:**

- **400** `Bad Request` user id truyền lên không phải dạng số hoặc không nằm trong khoảng cho phép từ 1 tới 4294967295.
- **404** `Not Found` lỗi xảy ra khi mnemonic trong config bị sai dẫn đến thuật toán không thể tìm ra địa chỉ ví.
- **200** `OK` success

### User Count

Trả về số lượng ví đã tạo

```http request
GET /api/v1/user/count
```

**Response:**

- _count_ `int` số lượng ví đã tạo.

**Status Code:**

- **404** `Not Found` truy vấn cơ sở dữ liệu thất bại.
- **200** `OK` success

### Get User ID

Trả về User ID ứng với địa chỉ ví

```http request
GET /api/v1/user/id/:address
```

Ví dụ: đối với địa chỉ ví `TKZvhWMEXAJuQw4Bp1Buqj4Sp5LBRhJvwC`

```http request
GET /api/v1/user/id/TKZvhWMEXAJuQw4Bp1Buqj4Sp5LBRhJvwC
```

**Request:**

- **address** `string` địa chỉ ví, trả về lỗi nếu địa chỉ không hợp lệ.

**Response:**

- _user_id_ `int` id của user.
- _address_ `string` địa chỉ ví.

**Status Code:**

- **400** `Bad Request` address không phải là địa chỉ ví.
- **404** `Not Found` không tìm thấy địa chỉ ví này trong cơ sở dữ liệu.
- **200** `OK` success.

### Get User Deposit Transactions Count

Trả về số lượng tổng giao dịch nạp tiền đã thành công của user

```http request
GET /api/v1/deposit/count/:userID
```

**Request:**

- _userID_ `int` ID của người chơi

**Response:**

- _user_id_ `int` id của người chơi
- _count_ `int` số lượng giao dịch
-

### Get User Completed Deposit Transactions

Lấy các giao dịch nạp tiền đã hoàn thành của người chơi.

```http request
GET /api/v1/deposit/completed/:userID?pageSize=1&page=1
```

**Request:**

- _userID_ `int` id của người chơi.
- _pageSize_ `int` số lượng giao dịch tối đa lấy.
- _page_ `int` vị trí của page lấy, ví dụ _page_ = 2, _pageSize_ = 10 thì lấy giao dịch thứ 10 tới thứ 20.

**Response:**

- [DepositTransaction[]](#deposittransaction) danh sách giao dịch.

**Status Code:**

- **400** `Bad Request` user ID, page size, page không phải là số hoặc không hợp lệ.
- **500** `Internal Server Error` không thể thực hiện truy vấn cơ sở dữ liệu.
- **200** `OK` success

### Get User Pending Deposit Transactions

Lấy tất cả các giao dịch nạp tiền đang chờ xác thực.

```http request
GET /api/v1/deposit/pending/:userID
```

**Request:**

- _userID_ `int` id của người chơi.

**Response:**

- [DepositTransaction[]](#deposittransaction) danh sách giao dịch.

**Status Code:**

- **400** `Bad Request` user ID không phải là số hoặc không hợp lệ.
- **500** `Internal Server Error` không thể thực hiện truy vấn cơ sở dữ liệu.
- **200** `OK` success

### Get User Withdraw Transactions Count

Trả về số lượng tổng giao dịch rút tiền đã thành công của user

```http request
GET /api/v1/withdraw/count/:userID
```

**Request:**

- _userID_ `int` ID của người chơi

**Response:**

- _user_id_ `int` id của người chơi
- _count_ `int` số lượng giao dịch

### Get User Completed Withdraw Transactions

Lấy các giao dịch rút tiền đã hoàn thành của người chơi.

```http request
GET /api/v1/withdraw/completed/:userID?pageSize=1&page=1
```

**Request:**

- _userID_ `int` id của người chơi.
- _pageSize_ `int` số lượng giao dịch tối đa lấy.
- _page_ `int` vị trí của page lấy, ví dụ _page_ = 2, _pageSize_ = 10 thì lấy giao dịch thứ 10 tới thứ 20.

**Response:**

- [WithdrawTransaction[]](#withdrawtransaction) danh sách giao dịch.

**Status Code:**

- **400** `Bad Request` user ID, page size, page không phải là số hoặc không hợp lệ.
- **500** `Internal Server Error` không thể thực hiện truy vấn cơ sở dữ liệu.
- **200** `OK` success

### Get User Pending Withdraw Transactions

Lấy tất cả các giao dịch rút tiền đang chờ thực hiện.

```http request
GET /api/v1/withdraw/pending/:userID
```

**Request:**

- _userID_ `int` id của người chơi.

**Response:**

- [WithdrawTransaction[]](#withdrawtransaction) danh sách giao dịch.

**Status Code:**

- **400** `Bad Request` user ID không phải là số hoặc không hợp lệ.
- **500** `Internal Server Error` không thể thực hiện truy vấn cơ sở dữ liệu.
- **200** `OK` success

### Withdraw

Yêu cầu hệ thống thực hiện yêu cầu rút tiền của người chơi và chuyển token sang ví của người chơi.

```http request
POST /api/v1/user/withdraw 
```

**Request Body:**

- _user_id_ `int` id của người chơi
- _address_ `string` địa chỉ ví
- _token_name_ `string` tên token
- _amount_ `float` số lượng token cần rút.
- _request_id_ `string` ID của giao dịch bên game service, ví dụ như ID của giao dịch lưu trong DB bên game service hoặc
  gì
  đó tương tự để hệ thống không thực hiện 1 giao dịch quá 1 lần, trong trường hợp game service gửi nhầm 2 lần 1 gói tin.
- _checksum_ `string` `MD5(API-Key + request_id)` mã checksum dùng để phát hiện gian lận nếu người gửi request này không
  phải Game Service.

**Status Code:**

- **400** `Bad Request` dữ liệu gửi lên không hợp lệ.
- **500** `Internal Server Error` không thể khởi tạo giao dịch.
- **201** `Created` đã khởi tạo thành công giao dịch rút tiền.

### Check Deposit Transaction

API kiểm tra giao dịch thủ công và cập nhật vào database nếu giao dịch hoàn thành 

```http request
POST /api/v1/deposit/check
```

**Request:**
- _user_id_ `int` id người chơi 
- _transaction_hash_ `string` mã giao dịch 

**Response Status Code:**
- _202_ success
- _400_ user id hoặc transaction hash không hợp lệ.
- _406_ giao dịch không tồn tại hoặc ở trạng thái thất bại.
- _404_ không hỗ trợ loại token này.
- _409_ giao dịch đã được hệ thống ghi nhận rồi.
- _500_ không thể khởi tạo giao dịch vào cơ sở dư liệu.
- _422_ giao dịch không hợp lệ (không phải giao dịch chuyển tiền loại này hoặc chuyển tới địa chỉ không chính xác)
- _423_ giao dịch đang ở trạng thái pending vui lòng chờ tới khi giao dịch hoàn tất.

### Get Deposit Transaction By Tx Hash

API lấy giao dịch nạp theo transaction hash.

```http request
GET /api/v1/deposit/:txHash
```

**Response:**
- _tx_hash_ `string` mã giao dịch 
- _user_id_ `string` id của người nạp 
- _contract_address_ `string` địa chỉ contract dùng để chuyển tiền.
- _from_address_ `string` địa chỉ của người dùng.
- _to_address_ `string`  địa chỉ ví con người dùng nạp vào.
- _block_number_ `int`  
- _block_hash_ `string`
- _value_ `string` 
- _token_name_ `string` tên token.
- _token_address_ `string` địa chỉ token.
- _tokens_ `float` số lượng tiền nạp vào  
- _created_at_ `int` thời điểm crawl được dữ liệu về.

**Status Code:**
- `404` không tìm thấy transaction.
- `200` success.

### Get Withdraw Transaction By Tx Hash

API lấy giao dịch rút theo transaction hash.

```http request
GET /api/v1/withdraw/:txHash
```

**Response:**
- _request_id_ `string`
- _tx_hash_ `string` mã giao dịch
- _user_id_ `string` id của người nạp
- _to_address_ `string`  địa chỉ ví con người dùng muốn rút tới.
- _token_name_ `string` tên token.
- _token_address_ `string` địa chỉ token.
- _tokens_ `float` số lượng tiền rút.
- _created_at_ `int` thời điểm crawl được dữ liệu về.

**Status Code:**
- `404` không tìm thấy transaction.
- `200` success.

### Model

#### DepositTransaction

- _id_ `string` ID của record.
- _tx_hash_ `string` mã giao dịch.
- _user_id_ `int` ID của user.
- _contract_address_ `string` địa chỉ contract.
- _from_address_ `string` địa chỉ ví gửi.
- _to_address_ `string` địa chỉ ví con nhận tiền.
- _block_number_ `int` số thứ tự block.
- _block_hash_ `string` mã block.
- _value_ `string` tiền khi chưa chia cho decimal.
- _token_name_ `string` tên của token khai báo trong file config.
- _token_address_ `string` địa chỉ của token.
- _tokens_ `float` số tiền gửi.
- _is_approved_ `bool` trường này bằng true nếu giao dịch đã được xác thực thành công, còn false nếu giao dịch đang đợi
  hoàn thành.
- _is_collected_ `bool` nếu tiền đã được chuyển về ví tổng thì = true.
- _updated_at_ `int` unix timestamp .
- _created_at_ `int` unix timestamp.
- _max_confirmations_ `int` số lượng tối đa xác nhận cần cho giao dịch này (chỉ dùng trong trường hợp pending
  transaction)
- _confirmations_ `int` số lượng xác nhận hiện tại (nếu = _max_confirmations_ thì giao dịch đã thành công), trong giao
  diện lịch sử giao dịch sẽ hiển thị số (confirmations / max confirmations) đối với các giao dịch đang thực hiện, để
  người dùng có thể biết là giao dịch này tiến độ hoàn thành đang là bao nhiêu %.

#### WithdrawTransaction

- _id_ `string` ID của record.
- _request_id_ `string` id của request này ứng với bên game server, để map 2 transaction với nhau.
- _user_id_ `int` id của user.
- _tokens_ `float` số tiền yêu cầu rút.
- _token_name_ `string` tên token.
- _token_address_ `string` địa chỉ token.
- _to_address_ `string` địa chỉ ví nhận tiền.
- _is_transferred_ `bool` true nếu tiền đã được gửi sang ví người dùng.
- _tx_hash_ `string` mã giao dịch (nếu _is_transferred_ = true).
- _created_at_ `int` unix timestamp
- _updated_at_ `int` unix timestamp 
