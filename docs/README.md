<h1 align="center">
    <img src="https://bucket-aicycle.s3.ap-southeast-1.amazonaws.com/logo.png" style="width: 400px;height: 100px;" alt="jsdeliver"> 
</h1>

## **Phiên bản hiện tại: 1.0.1**
| Phiên bản | Mô tả | Chi tiết thay đổi |
| ---- | ----- |-------|
| 1.0.0 | Các API ban đầu |
| 1.0.1 | Thêm chi tiết lỗi đã được cover cho API gọi Engine | Mục chi tiết lỗi đầu ra |
| 1.0.2 | Thêm API trả về ảnh có vẽ sẵn mask | Chi tiết API |
| 1.0.3 | Thêm trường oldImageId để xóa ảnh cũ cho API call engine |
| 1.0.4 | Thay đổi API lấy link upload ảnh (không bắt buộc truyền filePaths và trả thêm 1 trường filePath ở response dùng để call engine) | Chi tiết field |
| 2.0.0 | Thay đổi API gọi engine cho phần cấp đơn (bắt buộc truyền claimFolderId để call engine) | Chi tiết field |


## **1. Đặc tả tài liệu kết nối**
### **1.1 Các thông tin cơ bản**
|||
|----|----|
| Method | |
| API Url | |
| API Headers | |

### **1.2 Các mã HTTP Code trả về**
- 200: OK
- 400: Dữ liệu chưa đúng từ phía Client, client xem xét lại về dữ liệu đầu vào.
- 401: Lỗi Authenticate
- 403: Lỗi không có quyền truy cập API
- 5XX: Lỗi xuất phát từ phía máy chủ từ AICycle, liên hê đội ngũ hỗ trợ của AICycle cùng errorId để giải quyết.

### **1.3 Base Url:**
https://api.aicycle.ai/insurance

**API Key:** Liên hệ đội ngũ support tích hợp của AICycle để được cấp apiKey cho tổ chức.

## **2. APIs tích hợp ValueMe**
### 2.1: Flow tích hợp giữa các đối tác và AICycle
![alt text](https://dyta7vmv7sqle.cloudfront.net/AICycle-flow-client.png)
### **2.2: API định giá xe v3 (sử dụng CodeBook hãng hiệu và phiên bản của đối tác)**
#### a. Thông tin cơ bản

||                                                          |
|----|----------------------------------------------------------|
| Method | POST                                                     |
| API Url | https://api.aicycle.ai/insurance/valuation/v3/car-valuate |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`              |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**         |
|-----------------|-------------------|--------------|------------------|-------------|-------------------|
| **versionCode**     | **Code phiên bản xe** | **Bắt buộc**     | **Text**             | **1,255**       | **BMW02006** |
| year            | Năm sản xuất      |   Tùy chọn  | Number           | 1,9999      | 2020              |
| vinCode            | Thông tin số khung (số VIN) |   Tùy chọn  | Text           | 1,9999      | VNICODETOY211       |
| gearBox            | Hộp số  |   Tùy chọn  | Text           | 1,9999      | AT       |
| wheelDrive            | Dẫn động |   Tùy chọn  | Text           | 1,9999      | 4WD  |
| engineCapacity    | Dung tích động cơ |   Tùy chọn  | Float       | 1,9999      | 1.5  |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/valuation/v3/car-valuate' \
--header 'Authorization: Bearer <API Key>' \
--header 'Content-Type: application/json' \
--data '{
    "versionCode": "BMW02006",
    "year": 2021
}'
```
#### c. Chi tiết đầu ra

| **HTTP Status Code**   | **Mô tả**                                                         |
|----------------------|------------|
| 200        | Định giá thành công |
| 400        | Định giá không thành công (do sai dữ liệu đầu vào) |
| 401        | Sai thông tin xác thực |
| 500        | Lỗi từ hệ thống AICycle (liên hệ AIC Customer Support với) |

**Loại đầu ra khi định giá không thành công (HTTP 4xx)**: Response body
Khi các mã codebook không tồn tại:
```
{
    "message": "This version not support versionCode (TOYAVA0111NK)",
    "name": "BadRequestError",
    "status": 400,
    "traceId": "6ae1188803e4aebc9bb7ade579994fc3"
}
```
Khi codebook đúng mà sai năm:
```
{
    "message": "This version not support year (2020)",
    "name": "BadRequestError",
    "status": 400,
    "errors": null,
    "listAvailableYears": [
        2006,
        2007,
        2008,
        2009,
        2010,
        2011,
        2012,
        2013,
        2013,
        2014,
        2014,
        2015
    ],
    "traceId": "d097a03a9e03e50ef7925b72bdeef9a8"
}
```
Khi sai token (401):
```
{
    "message": "Api token is required.",
    "traceId": "1716f059b256f097771ceba42667f3c6"
}
```

**Loại đầu ra khi định giá thành công (HTTP 200)**: Response body
```
[<ValuationResult>]
```
*Chi tiết Object item `ValuationResult`*

| **Tên Tham số**    | **Mô tả**                                                         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**         |
|--------------------|-------------------------------------------------------------------|--------------|------------------|---------|-------------------|
| companyCode        | Mã hãng xe của tổ chức                                            | Tùy chọn     | Text             | 1,255   | code-hang-xe      |
| modelCode          | Mã hiệu xe của tổ chức                                            | Tùy chọn     | Text             | 1,255   | code-hieu-xe      |
| versionCode        | Mã phiên bản xe của tổ chức                                       | Bắt buộc     | Text             | 1,255   | code-phien-ban-xe |
| year               | Năm sản xuất                                                      | Bắt buộc     | Number           | 1,9999  | 2020              |
| minPrice           | Giá min của xe                                                    | Bắt buộc     | Number           | 1,999999999 | 100000000         |
| maxPrice           | Giá max của xe                                                    | Bắt buộc     | Number           | 1,999999999 | 300000000         |
| carValue           | Giá trị xe                                                        | Bắt buộc     | Number           | 1,999999999 | 300000000         |
| isWarning          | Thông tin về dòng xe này trên thị trường có đang thiếu hay không? | Bắt buộc     | Boolean          | n       | true              |
| listedPrice        | Giá xe niêm yết                                                   | Bắt buộc     | Number           | 1,999999999        | 320000000         |
| minListedPrice     | Giá trị min của giá niêm yết                                      | Bắt buộc     | Number           | 1,999999999        | 305000000         |
| maxListedPrice     | Giá trị max của giá niêm yết                                      | Bắt buộc     | Number           | 1,999999999        | 330000000         |
| hanoiOnRoadPrice   | Gía lăn bánh tại Hà Nội                                           | Bắt buộc     | Number           | 1,999999999        | 323000000         |
| hcmOnRoadPrice     | Gía lăn bánh tại TP HCM                                           | Bắt buộc     | Number           | 1,999999999        | 323000000         |
| generalOnRoadPrice | Gía lăn bánh tại các tỉnh khác                                    | Bắt buộc     | Number           | 1,999999999        | 323000000         |
| carFuelName | Nhiên liệu                                                        | Bắt buộc     | Text             | 1,255        | Xăng              |
| numOfSeat | Số chỗ ngồi                                                       | Bắt buộc     | Number           | 1,9999        | 5                 |
| listAvailableYears | Danh sách các năm gợi ý hợp lệ cho CodeBook hiện tại              | Bắt buộc     | Array[Number]    | 1,999999999        | [2020,2021]       |
#### d. Ví dụ đầu ra
```
[
    {
        "companyCode": "<companyCode>",
        "modelCode": "<modelCode>",
        "versionCode": "<versionCode>",
        "year": <year>,
        "maxPrice": 679943600,
        "carValue": 601720000,
        "minPrice": 559599600,
        "isWarning": true,
        "listedPrice": 710000000,
        "minListedPrice": 674500000,
        "maxListedPrice": 816500000,
        "hanoiOnRoadPrice": 732380000,
        "generalOnRoadPrice": 713380000,
        "hcmOnRoadPrice": 732380000,
        "carFuelName": "Xăng",
        "numOfSeat": 5,
        "listAvailableYears": [
            2006,
            2007,
            2008,
            2009,
            2010,
            2011,
            2012,
            2013,
            2013,
            2014,
            2014,
            2015
        ]
    }
]
```
### **2.3: API lấy danh sách hãng xe của AICycle (car company)**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/car-info/company |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Query

| **Tên Tham số**  | **Mô tả**                     | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|------------------|-------------------------------|--------------|------------------|-------------|-----------|
| vehicleBrandName | Tên hãng xe                   | Tùy chọn     | Text             | 1, 255      | TOYOTA    |
| limit            | Limit số lượng bản ghi trả về | Tùy chọn     | Number           | 1,9999      | 30        |
| offset           | Bỏ qua 1 số bản ghi           | Tùy chọn     | Number           | 0,9999      | 0         |

**Ví dụ**
```
curl --location --request GET 'https://stage-api-insurance.aicycle.ai/car-info/company?limit=1000' \
--header 'Authorization: Bearer <API Key>'

```
#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số**     | **Mô tả**             |**Bắt buộc**| **Kiểu dữ liệu**      | **Min,Max** | **Ví dụ**             |
|---------------------|-----------------------|---|-----------------------|-------------|-----------------------|
| count               | Số lượng bản ghi      |Bắt buộc| Number                | 1,9999      | 30                    |
| limit               | Giới hạn bản ghi      |Bắt buộc| Number                | 1,9999      | 100                   |
| offset              | Bỏ qua 1 số bản ghi   |Bắt buộc| Number                | 0,9999      | 0                     |
| records             | Chi tiết các hãng xe  |Bắt buộc| Array[carCompanyInfo] | n           | []                    |


*Chi tiết Object item  `carCompanyInfo`*

| **Tên Tham số**  | **Mô tả**          |**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**| **Ví dụ**    |
|------------------|--------------------|---|---|---|--------------|
| carCompanyId     | Mã hãng xe         |Bắt buộc|Text|1,255| uuid-hang-xe |
| vehicleBrandName | Tên hãng xe        |Bắt buộc|Text|1,255| TOYOTA       |

#### d. Ví dụ đầu ra
```
{
    "count": 114,
    "limit": 1000,
    "offset": 0,
    "records": [
        {
            "carCompanyId": "uuid-hang-xe",
            "vehicleBrandName": "TOYOTA"
        }
    ]
}
```

### **2.4: API lấy danh sách hiệu xe của AICycle (car model)**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/car-info/model |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Query

| **Tên Tham số** | **Mô tả**   |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**    |
|-----------------|-------------|---|------------------|-------------|--------------|
| name            | Tên hiệu xe |Tùy chọn| Text             | 1,255       | VIOS         |
| carCompanyId    | Id hãng xe  |Tùy chọn| Text             | 1,255       | uuid-hieu-xe |
| limit            | Limit số lượng bản ghi trả về | Tùy chọn     | Number           | 1,9999      | 30        |
| offset           | Bỏ qua 1 số bản ghi           | Tùy chọn     | Number           | 0,9999      | 0         |

**Ví dụ**
```
curl --location --request GET 'https://stage-api-insurance.aicycle.ai/car-info/model?carCompanyId=<uuid-hang-xe>&limit=100' \
--header 'Authorization: Bearer <API Key>'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu**    | **Min,Max** | **Ví dụ**             |
|---------------------|----------------------|---|---------------------|-------------|-----------------------|
| count               | Số lượng bản ghi     |Bắt buộc| Number              | 1,9999      | 30                    |
| limit               | Giới hạn bản ghi     |Bắt buộc| Number              | 1,9999      | 100                   |
| offset              | Bỏ qua 1 số bản ghi  |Bắt buộc| Number              | 0,9999      | 0                     |
| records             | Chi tiết các hiệu xe |Bắt buộc| Array[carModelInfo] | n           | []                    |


*Chi tiết Object item  `carModelInfo`*

| **Tên Tham số** | **Mô tả**   |**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**| **Ví dụ**    |
|-----------------|-------------|---|---|---|--------------|
| carModelId      | Mã hiệu xe  |Bắt buộc|Text|1,255| uuid-hieu-xe |
| carModelName    | Tên hiệu xe |Bắt buộc|Text|1,255| VIOS         |

#### d. Ví dụ đầu ra
```
{
    "count": 114,
    "limit": 1000,
    "offset": 0,
    "records": [
        {
            "carModelId": "uuid-hieu-xe",
            "carModelName": "VIOS"
        }
    ]
}
```

### **2.5: API lấy danh sách các năm sản xuất (manufactured year) theo hãng xe và hiệu xe**
#### a. Thông tin cơ bản
|||
|----|----------------------------------------------------------------------------------------------------------------|
| Method | GET                                                                                                            |
| API Url | https://api.aicycle.ai/insurance/car-info/company/{carCompanyId}/model/{carModelId}/manufactured-years |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                                                                    |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Param

| **Tên Tham số** | **Mô tả**  |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**    |
|-----------------|------------|---|------------------|-------------|--------------|
| carCompanyId    | Id hãng xe |Bắt buộc| Text             | 1,255       | uuid-hang-xe |
| carModelId   | Id hiệu xe |Bắt buộc| Text             | 1,255       | uuid-hieu-xe |

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/car-info/company/{carCompanyId}/model/{carModelId}/manufactured-years' \
--header 'Authorization: Bearer <API Key>'
```

#### c. Ví dụ đầu ra
```
[
    2023,
    2022,
    2021,
    2020,
    2019,
    2018,
    2017,
    2016,
    2015,
    2014,
    2013,
    2012,
    2011
]
```

### **2.6: API lấy ra các phiên bản xe (car version) theo hãng xe và hiệu xe**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/car-info/company/{carCompanyId}/model/{carModelId}/versions |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Param

| **Tên Tham số** | **Mô tả**  |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**    |
|-----------------|------------|---|------------------|-------------|--------------|
| carCompanyId    | Id hãng xe |Bắt buộc| Text             | 1,255       | uuid-hang-xe |
| carModelId   | Id hiệu xe |Bắt buộc| Text             | 1,255       | uuid-hieu-xe |

**Loại đầu vào**: Query

| **Tên Tham số** | **Mô tả**    | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|--------------|--------------|------------------|-------------|-----------|
| year            | Năm sản xuất | Tùy chọn     | Number           | 1,9999      | 2001      |

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/car-info/company/{carCompanyId}/model/{carModelId}/versions?year=2011' \
--header 'Authorization: Bearer <API Key>'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**                   |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**             |
|-----------------|-----------------------------|---|------------------|-------------|-----------------------|
| carVersionId    | Id phiên bản xe             |Bắt buộc| Text             | 1,255       | uuid-phien-ban-xe     |
| carVersionName  | Tên phiên bản xe            |Bắt buộc| Text             | 1,255       | VIOS 1.5.G            |
| carCompanyId    | Id hãng xe                  |Bắt buộc| Text             | 1,255       | uuid-hang-xe          |
| carModelId      | Id hiệu xe                  |Bắt buộc| Text             | 1,255       | uuid-hieu-xe          |
| carVersionKey   | Key unique của phiên bản xe |Bắt buộc| Text             | 1,255       | key-version           |
| year            | Năm sản xuất                |Bắt buộc| Number           | 1,999999    | 2011                  |

#### d. Ví dụ đầu ra
```
[
    {
        "carVersionId": "<carVersionId>",
        "carVersionName": "1.5 G",
        "carCompanyId": "<carCompanyId>",
        "carModelId": "<carModelId>",
        "carVersionKey": "toyota-vios-1.5.g-2011",
        "year": 2011
    }
]
```

### **2.7: API yêu cầu mở mới codebook**
#### a. Thông tin cơ bản

||                                                    |
|----|----------------------------------------------------|
| Method | POST                                               |
| API Url | https://api.aicycle.ai/insurance/codebook-requests |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`        |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-------------|-----------|
| carCompanyName    | Tên hãng xe       | Bắt buộc     | Text             | 1,255       | TOYOTA    |
| carCompanyCode   | Code hãng xe      | Tùy chọn     | Text             | 1,255       | TOY       |
| carModelName   | Tên hiệu xe       | Bắt buộc     | Text             | 1,255       | FORTUNER  |
| carModelCode   | Code hiệu xe      | Tùy chọn     | Text             | 1,255       | TOY03     |
| carVersionName   | Tên phiên bản     | Bắt buộc     | Text             | 1,255       | 3.0 V     |
| carVersionCode   | Code phiên bản    | Tùy chọn     | Text             | 1,255       | TOY03011  |
| option   | Option riêng      | Tùy chọn     | Text             | 1,255       | PIN       |
| carType   | Kiểu xe           | Tùy chọn     | Text             | 1,255       | SUV       |
| carGearBox   | Hộp số            | Tùy chọn     | Text             | 1,255       | AT        |
| engineCapacity   | Dung tích động cơ | Tùy chọn     | Number             | 1,9999       | 2.7       |
| numOfSeat   | Số chỗ ngồi       | Tùy chọn     | Number             | 1,9999       | 7         |
| carFuel   | Nhiên liệu        | Tùy chọn     | Text             | 1,255       | Xăng      |
| carWheelDrive   | Dẫn động          | Tùy chọn     | Text             | 1,255       | RFD       |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/codebook-requests' \
--header 'Authorization: Bearer <API Key> \
--header 'Content-Type: application/json' \
--data '{
    "carCompanyName": "TOYOTA",
    "carCompanyCode": "TOY",
    "carModelName": "FORTUNER",
    "carModelCode": "TOY03",
    "carVersionName": "3.0 V",
    "carVersionCode": "TOY03011",
    "option": "PIN",
    "carType": "SUV",
    "carGearBox": "AT",
    "engineCapacity": 2.7,
    "numOfSeat": 7,
    "carFuel": "Xăng",
    "carWheelDrive": "RFD"
}'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: StatusCode

| **Code**       | **Mô tả**                   |
|----------------|-----------------------------|
| 201   | Tạo request codebook thành công             |
| 400   | Lỗi request từ client, review lại request và thử lại             |
| 401   | Lỗi thông tin xác thực hoặc ủy quyền             |
| 500   | Lỗi từ hệ thống API             |

### **2.8: API định giá xe v1 (sử dụng CodeBook hãng hiệu và phiên bản của AICycle) (deprecated)**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/valuation/car-valuate |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**           | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**          |
|-----------------|---------------------|--------------|------------------|-----------|--------------------|
| carCompanyId    | Id hãng xe          | Bắt buộc     | Text             | 1,255     | uuid-hang-xe       |
| carModelId      | Id hiệu xe          | Bắt buộc     | Text             | 1,255     | uuid-hieu-xe       |
| carVersionId    | Id phiên bản xe     | Bắt buộc     | Text             | 1,255     | uuid-phien-ban-xe  |
| year            | Năm sản xuất        | Bắt buộc     | Number           | 1,9999    | 2020               |
| option          | Option riêng của xe | Optional     | Text             | 1,255     | pin                |
| carStatus       | Tình trạng xe       | Optional     | Text             | 1,255     | old                |

**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/valuation/car-valuate' \
--header 'Authorization: Bearer <API Key>' \
--header 'Content-Type: application/json' \
--data-raw '{
    "carCompanyId": "uuid-hang-xe",
    "carModelId": "uuid-hieu-xe",
    "carVersionId": "uuid-phien-ban-xe",
    "year": 2020
}'
```
#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**                                                         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**         |
|-----------------|-------------------------------------------------------------------|--------------|------------------|-------------|-------------------|
| carCompanyId    | Id hãng xe                                                        | Bắt buộc     | Text             | 1,255       | uuid-hang-xe      |
| carModelId      | Id hiệu xe                                                        | Bắt buộc     | Text             | 1,255       | uuid-hieu-xe      |
| carVersionId    | Id phiên bản xe                                                   | Bắt buộc     | Text             | 1,255       | uuid-phien-ban-xe |
| year            | Năm sản xuất                                                      | Bắt buộc     | Number           | 1,9999      | 2020              |
| minPrice        | Giá min của xe                                                    | Bắt buộc     | Number           | 1,999999999 | 100000000         |
| maxPrice        | Giá max của xe                                                    | Bắt buộc     | Number           | 1,999999999 | 300000000         |
| carValue        | Giá trị xe                                                        | Bắt buộc     | Number           | 1,999999999 | 300000000         |
| isWarning       | Thông tin về dòng xe này trên thị trường có đang thiếu hay không? | Bắt buộc     | Boolean          | n           | true              |
| listedPrice        | Giá xe niêm yết                                                   | Bắt buộc     | Number           | 1,999999999        | 320000000         |
| minListedPrice     | Giá trị min của giá niêm yết                                      | Bắt buộc     | Number          | 1,999999999        | 305000000         |
| maxListedPrice     | Giá trị max của giá niêm yết                                      | Bắt buộc     | Number          | 1,999999999        | 330000000         |
| hanoiOnRoadPrice   | Gía lăn bánh tại Hà Nội                                           | Bắt buộc     | Number          | 1,999999999        | 323000000         |
| hcmOnRoadPrice     | Gía lăn bánh tại TP HCM                                           | Bắt buộc     | Number          | 1,999999999        | 323000000         |
| generalOnRoadPrice | Gía lăn bánh tại các tỉnh khác                                    | Bắt buộc     | Number          | 1,999999999        | 323000000         |
#### d. Ví dụ đầu ra
```
[
    {
        "carVersionId": "<carVersionId>",
        "carCompanyId": "<carCompanyId>",
        "carModelId": "<carModelId>",
        "year": <year>,
        "maxPrice": 679943600,
        "carValue": 601720000,
        "minPrice": 559599600,
        "isWarning": true,
        "listedPrice": 710000000,
        "minListedPrice": 674500000,
        "maxListedPrice": 816500000,
        "hanoiOnRoadPrice": 732380000,
        "generalOnRoadPrice": 713380000,
        "hcmOnRoadPrice": 732380000
    }
]
```

## **3. APIs tích hợp BuyMe/ClaimMe**
### **3.1: Flow diagram tích hợp cho BuyMe realtime**
![Alt text](https://dyta7vmv7sqle.cloudfront.net/AICycle-flow.png "a title")


### **3.2: Tạo mới hồ sơ (claimFolder)**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/claimfolders |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**| **Mô tả**                  |**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|----------------------------|---|---|---|---|
|claimName| Mã hồ sơ của đối tác       |Bắt buộc|Text|1,255|Folder-1|
|vehicleBrandName| Tên hãng xe của hồ sơ      ||Text|1,255|mazda|
|vehicleModel| Tên hiệu xe của hồ sơ      ||Text|1,255|mazda.3_sedan|
|vehicleSpec| Tên phiên bản xe của hồ sơ ||Text|1,255|luxury_1_5l_at|
|vehicleLicensePlates| Biển số xe của hồ sơ       ||Text|1,255|30H84142|

**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/claimfolders' \
--header 'authorization: Bearer $$apiKey$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimName":"34AAAAA",
    "vehicleBrandName": "mazda",
    "vehicleModel": "mazda.3_sedan",
    "vehicleSpec": "luxury_1_5l_at",
    "vehicleLicensePlates": "30H84142"
}'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimId|Id của claim folder|Bắt buộc|Text|1,255|123|

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "msg": "Create claim folder success",
    "data": [
        {
            "claimId": "189",
            "claimName": "34AAAAA",
            "vehicleBrandName": "Toyota Vios"
        }
    ]
}
```
### **3.3: API lấy link upload ảnh lên s3 AICycle**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/images/url |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|filePaths|Đường dẫn file trên S3|Optional|Array[Text]|1,100|["INSURANCE_CLAIM/image-1.jpg", "INSURANCE_CLAIM/image-2.jpg"]|

**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/images/url' \
--header 'Authorization: Bearer $$API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "filePaths": ['INSURANCE_CLAIM/image-1.jpg', 'INSURANCE_CLAIM/image-2.jpg']
}'
```


#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|urls[]|Array Urls upload và lấy image trên s3|Bắt buộc|Object|1,255|INSURANCE_CLAIM/image-1.jpg|
|urls[n].fetchUrl|Url lấy ảnh|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/image-1.jpg|
|urls[n].uploadUrl|Url upload ảnh|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/image-1.jpg|
|urls[n].filePath|filePath dùng để call engine|Bắt buộc|Text|1,255|INSURANCE/image-1.jpg|

#### d. Ví dụ đầu ra
```
{
    "urls": [
        {
            "fetchUrl": "https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/quang-6.jpg",
            "uploadUrl": "https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/quang-6.jpg",
            “filePath": “INSURANCE/image-1.jpg”
        },
        {
            "fetchUrl": "https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/quang-6.jpg",
            "uploadUrl": "https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/quang-6.jpg",
            “filePath": “INSURANCE/image-2.jpg”
        }
    ],
    "msg": "Get url image success",
    "status": "success"
}
```


**Chú ý**
> Sau khi call api lấy link `uploadUrl` để up ảnh. Trên postman dùng link đó với method là PUT, body dạng binary và select file ảnh trên máy để upload. Khi nhận được status là 200 (upload thành công), dùng đường link `fetchUrl` đó paste lên browser để xem kết quả

### **3.4: API Call Engine AICycle BuyMe V2 (new version)**
##### a. Thông tin cơ bản

||                                                    |
|----|----------------------------------------------------|
| Method | POST                                               |
| API Url | https://api.aicycle.ai/insurance/v2/buy-me/process |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`        |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                    |
|-----------------|--------------------------------------------------------------------------|----------|------------------|-------------|------------------------------|
| claimId         | Id của Hồ sơ                                                             | Bắt buộc | Number           | 1,999999    | 123 |
| imageName       | Tên ảnh                                                                  | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| filePath        | Path ảnh trên s3 (lấy từ kết quả trả về  của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| position        | Slug Loại ảnh (Toàn cảnh, trung cảnh, cận cảnh)                          | Bắt buộc | Text             | 1,255       | toan-canh-afh4l5 |
| direction       | Slug Hướng ảnh                                                           | Bắt buộc | Text             | 1,255       | 45-trai-truoc-C1xM02 |
| oldImageId      | Id ảnh cũ muốn xóa                                                       | Optional | Number           | 1,9999      | 3 |
| vehicleType     | Loại xe muốn engine detect (xe tải, xe con)                              | Optional | ENUM(truck, car) | 1,255       | truck                       |

*Lưu ý*
> **Các giá trị của `position`**
>
>| positionName | positionSlug |
>|--------------|-----------|
>| Toàn cảnh    | toan-canh-afh4l5          |
>| Trung cảnh   | trung-canh-0s8mnb          |
>| Cận cảnh     | can-canh-czu5jp          |

> **Các giá trị của `direction`**
>
>| directionName    | directionSlug |
>|------------------|---|
>| Trước            | truoc-sT9qgX  |
>| 45° Phải - Trước | 45-phai-truoc-UoYzs6  |
>| 45° Trái - Trước | 45-trai-truoc-C1xM02  |
>| Sau              | sau-htBwjB  |
>| 45° Phải - Sau   | 45-phai-sau-fRzY3r  |
>| 45° Trái - Sau   | 45-trai-sau-1q3G3J  |
>| Phải - Trước     | phai-truoc-eYWg1d  |
>| Trái - Trước     | trai-truoc-r6BEZd  |
>| Phải - Sau       | phai-sau-v1hAm6  |
>| Trái - Sau       | trai-sau-t8QgFO  |

**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/v2/buy-me/process' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimId": 1,
    "imageName": "INSURANCE/1666145614576/1666145614447.jpg",
    "filePath": "INSURANCE/1666145614576/1666145614447.jpg",
    "position": "toan-canh-afh4l5",
    "direction": "45-trai-truoc-C1xM02",
     “oldImageId": 123
}'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng trái (String)>,
    "isPhotoValid": <Ảnh có hợp lệ hay không (Boolean)>,
    "traceId": <ID dùng để giải quyết khi liên hệ AICycle (UUID)>,
    "errorCodeFromEngine": <ErrorCode (Number)>,
    "message": <String>,
    "imageId": <ID của ảnh (Number)>,
    "result": [<Image>]
}
```
*Chi tiết Object item  `Image`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| imgSize             | Kích thước ảnh       |Bắt buộc| Array[number]    | n           | [1920, 1080] |
| imgUrl              | Link ảnh gốc         |Bắt buộc| Text             | 1,255       | {{s3Link}} |
| errorCodeFromEngine | Mã lỗi ảnh           |Bắt buộc| Number           | 1,999999    | 0       |
| message             | Chi tiết lỗi ảnh     |Bắt buộc| Text             | 1,255       | Ảnh chụp qua màn hình |
| damages             | Các hỏng hóc của ảnh |Bắt buộc| Array[damage]    | n           |         |
| carParts            | Các bộ phận của ảnh  |Bắt buộc| Array[carPart]   | n           |    |

*Chi tiết Object item  `carPart`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| carPartKey      |Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
| position        |Vị trí bộ phận|Bắt buộc|Text|1,255|Trái - Trước|
| name            |Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
| maskUrl         |Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Chi tiết Object item `damage`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| name            |Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
| maskUrl         |mask hỏng hóc|Bắt buộc|Text|1,255|...|

*Chi tiết Object item `extraInfo`*

| **Tên Tham số** | **Mô tả**     |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|---------------|---|------------------|-------------|--------|
| plateNumber     | Biển số xe    |Bắt buộc| Text             | 1,255       | 30A 9999 |
| carCompany      | Hãng xe       |Bắt buộc| Text             | 1,255       | TOYOTA |
| carModel        | Hiệu xe       |Bắt buộc| Text             | 1,255       | Vios   |
| carColor        | Hiệu xe       |Bắt buộc| [Number]         | 1,999999    | [220,219,215] |
| imagePosition   | Slug loại ảnh |Bắt buộc| Text             | 1,255       |   trung-canh-0s8mnb     |
| imageDirection  | Slug góc chụp   |Bắt buộc| Text             | 1,255       | trai-truoc-r6BEZd      |


***Chi tiết Bảng Mã lỗi cùng httpStatus trả về của `errorCodeFromEngine`***

*Chú thích*
> Với những lỗi có level là error sẽ chỉ trả ra mã lỗi và message lỗi, những lỗi level warning sẽ trả ra mã lỗi, message, và car_part, car_damages như bình thường

|**Mã lỗi**|**HTTP Status**|**Response body**|**Level lỗi**|
|----|----|----|----|
|0|200|Ảnh hợp lệ||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Ảnh chụp bị mờ. Vui lòng chụp lại"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Không download được ảnh"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Ảnh chụp bị tối"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Ảnh chụp qua màn hình. Vui lòng chụp lại"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Trả ra error của engine"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Ảnh chụp bị lóa"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Ảnh không đúng góc chụp. Vui lòng chụp lại"}`|Error|
|378224|400|`{"errorCodeFromEngine": 378224, "message": "Ảnh không phải xe tải"}`|Error|
|178434|400|`{"errorCodeFromEngine": 178434, "message": "Ảnh không phải xe con"}`|Error|

#### d. Ví dụ đầu ra

```
{
    "status": "success",
    "isPhotoValid": true,
    "traceId": "80540cf80585486f63432dcdf680fd20",
    "errorCodeFromEngine": 0,
    "message": "",
    "imageId": 4808,
    "result": [
        {
            "imgSize": [1920, 1080],
            "extraInfor": {
                "plateNumber": "",
                "chassisNumber": null,
                "carCompany": "",
                "carModel": "",
                "carColor": [222, 220,216],
                "imagePosition": "trung-canh-0s8mnb",
                "imageDirection": "trai-truoc-r6BEZd"
            },
            "damages": [
                {
                    "name": "Móp, bẹp(thụng)",
                    "damageKey": "mop-bep-jtep4m",
                    "location": "",
                    "score": 1,
                    "box": [ 0.34, 0.14, 0.79,0.77],
                    "maskPath": "nfU_HFiSOi4wEkP1ycUwv.png",
                    "isPart": false,
                    "maskUrl": "{{s3Url}}"
                },
                ...
            ],
            "carParts": [
                {
                    "name": "Trụ kính cánh cửa",
                    "carPartKey": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
                    "position": "Trái - Trước",
                    "score": 0.83660888671875,
                    "box": [
                        0.81875,
                        0,
                        0.9994791666666667,
                        0.2
                    ],
                    "maskPath": "vG4wVxBeWvLF5MFGMAfq9.png",
                    "isPart": true,
                    "damages": [
                        {
                            "name": "Trầy, xước",
                            "damageKey": "yfMzer07THdYoCI1SM2LN",
    
                            "score": 1,
                            "box": [
                                0.2171875,
                                0.07685185185185185,
                                0.9989583333333333,
                                0.9425925925925925
                            ],
                            "maskPath": "favy1yqepBr_JM8tAJm2v.png",
                            "isPart": false,
                            "overlapRate": 0.00047147571877818216,
                            "maskUrl": "{{s3Url}}",
                        },
                        ...
                    ],
                },
                ...
            ]
        }
    ]
}
```

### **3.5: API lấy kết quả Hồ sơ (nhóm theo bộ phận)**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/claimfolders/claim-results/{claimId} |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimId|Id của Hồ sơ|Bắt buộc|Number|1,999999|123|

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/claimfolders/claim-results/2450' \
--header 'authorization: Bearer $$API_KEY$$' \
--data-raw ''
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body:
```
{
    "results": [<VehiclePart>],
    "summary": <Tổng số tiền thiệt hại theo VND (number)>
}
```
*Chi tiết Object item `VehiclePart`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|vehiclePartExcelId|Mã bộ phận|Bắt buộc|Text|1,255|ba-do-soc-truoc-WUBZvD|
|vehiclePartName|Tên bộ phận|Bắt buộc|Text|1,255|Ba đờ sốc trước|
|location|Vị trí bộ phận|Bắt buộc|Text|1,255|Trước|
|damages|Chi tiết hỏng hóc|Bắt buộc|Array[damage]|n|[]|
|images|Ảnh|Bắt buộc|Array[image]|n|[]|

*Chi tiết Object item `damage`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|damageTypeName|Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
|damagePercentage|Phần trăm hỏng hóc|Bắt buộc|Number|0,1|0.05|
|damageArea|Diện tích hỏng hóc|Bắt buộc|Number|0,1|0.05|

*Chi tiết Object item `image`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|imageUrl|Url ảnh|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|
|imageRange|Loại ảnh|Bắt buộc|Text|1,255|Toàn cảnh|
|location | Nơi ảnh được chụp | Bắt buộc | TEXT | 1,255 | Duy Tân, Cầu Giấy, Hà Nội |
|requestedTime | Thời gian chụp ảnh | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
| uploadedTime | Thời gian ảnh được upload lên hệ thống | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
|uploadLocation | Nơi upload ảnh để thực hiện claim | Bắt buộc | TEXT | 1,255 | Hai Bà Trưng, Hà Nội |
|damageMasks|Chi tiết hỏng hóc của ảnh|Bắt buộc|Array[damageMask]|n|[]|

*Chi tiết Object item `damageMask`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|damageTypeName|Loại hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
|maskUrl|Url mask hỏng hóc|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

#### d. Ví dụ đầu ra

```
{
    "result": [
        {
            "vehiclePartExcelId": "ba-do-soc-truoc-WUBZvD",
            "vehiclePartName": "Ba đờ sốc trước",
            "location": "Trước",
            "createdDate": "2022/10/24",
            "damages": [
                {
                    "damageTypeName": "Trầy (xước)",
                    "damagePercentage": 0.0419343353811561,
                    "damageArea": 0.419343353811561,
                    "damageTypeColor": "#FFEC05"
                }
            ],
            "images": [
                {
                    "imageId": 10579,
                    "filePath": "INSURANCE_CLAIM/1652252930377/image_picker7865199709592835107.jpg",
                    "imageUrl": "https://s3-sgn09.fptcloud.com/aicycle/INSURANCE_CLAIM/1652252930377/image_picker7865199709592835107.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0080fa5f9d06c7ad85c7%2F20221024%2Fsgn09%2Fs3%2Faws4_request&X-Amz-Date=20221024T105823Z&X-Amz-Expires=7200&X-Amz-Signature=1e0ad191019668c5e77af28aa301f00f3d5f22a98882a1024660da2b9d946897&X-Amz-SignedHeaders=host",
                    "imageRange": "Toàn cảnh",
                    "damageExist": true,
                    "errorType": null,
                    "errorNote": null,
                    "location": "Duy Tân, Cầu Giấy, Hà Nội",
                    "requestedTime": "2023-03-24T03:28:29.414Z",
                    "uploadedTime": "2023-03-27T03:04:53.001Z",
                    "uploadLocation": "Quận Cầu Giấy, Hà Nội",
                    "timeProcess": 2.678,
                    "damageMasks": [
                        {
                            "maskPath": "8EskO3gPp1DYO00BOcBPq.png",
                            "maskUrl": "https://s3-sgn09.fptcloud.com/aicycle/INSURANCE_RESULT/8EskO3gPp1DYO00BOcBPq.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0080fa5f9d06c7ad85c7%2F20221024%2Fsgn09%2Fs3%2Faws4_request&X-Amz-Date=20221024T105823Z&X-Amz-Expires=7200&X-Amz-Signature=d531e05207a16858319bf560331cb684e20e728abd84c516c4d2918d5139dbef&X-Amz-SignedHeaders=host",
                            "vehiclePartName": "Ba đờ sốc trước",
                            "damageTypeUuid": "yfMzer07THdYoCI1SM2LN",
                            "damageTypeName": "Trầy (xước)",
                            "damageTypeColor": "#FFEC05",
                            "boxes": [
                                0,
                                0,
                                1,
                                1
                            ],
                            "isPart": false,
                            "userCreated": false
                        }
                    ]
                }
            ],
            "price": 1000000,
            "laborCost": 0,
            "totalCost": 1000000,
            "area": 10,
            "repairPlan": "Sửa chữa"
        }
    ],
    "summary": 1000000
}
```

### **3.6: API lấy kết quả Hồ sơ (nhóm theo ảnh)**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/claimfolders/{claimId}/get-image-results |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimId|Id của Hồ sơ|Bắt buộc|Number|1,999999|123|

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/claimfolders/{claimId}/get-image-results' \
--header 'authorization: Bearer $$API_KEY$$' \
--data-raw ''
```
#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "results": {
        images: [<Image>]
    }
}
```

*Chi tiết Object item `Image`*

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu**  | **Min,Max** | **Ví dụ**   |
|-----------------|-------------------|---|-------------------|-------------|-------------|
| url             | Link ảnh          |Bắt buộc| Text              | 1,255       | {{imgUrl}}  |
| imageSize       | Kích thước ảnh    |Bắt buộc| Array[number]     | n           | [1920,1080] |
| directionName   | Góc chụp          |Bắt buộc| Text              | 1,255       | Trước       |
| imageRangeName  | Loại ảnh          |Bắt buộc| Text              | 1,255       | Toàn cảnh   |
|location | Nơi ảnh được chụp | Bắt buộc | TEXT | 1,255 | Duy Tân, Cầu Giấy, Hà Nội |
|requestedTime | Thời gian chụp ảnh | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
| uploadedTime | Thời gian ảnh được upload lên hệ thống | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
|uploadLocation | Nơi upload ảnh để thực hiện claim | Bắt buộc | TEXT | 1,255 | Hai Bà Trưng, Hà Nội |
| damageInfo      | Chi tiết hỏng hóc |Bắt buộc| Array[damageInfo] | n           | []          |

*Chi tiết Object item `damageInfo`*

| **Tên Tham số** | **Mô tả**                     | **Bắt buộc** | **Kiểu dữ liệu**    | **Min,Max** | **Ví dụ**       |
|-----------------|-------------------------------|--------------|---------------------|-------------|-----------------|
| vehiclePartName | Tên bộ phận                   | Bắt buộc     | TEXT                | 1,255       | Ba đờ sốc trước |
| price           | Giá bộ phận                   | Bắt buộc     | Number              | 1,9999      | 1000000         |
| location        | Vị trí bộ phận                | Bắt buộc     | TEXT                | 1,255       | Trước           |
| damageDetail    | Chi tiết hỏng hóc của bộ phận | Bắt buộc     | Array[damageDetail] | n           | []              |

*Chi tiết Object item `damageDetail`*

| **Tên Tham số**  | **Mô tả**          | **Bắt buộc** | **Kiểu dữ liệu**    | **Min,Max** | **Ví dụ**   |
|------------------|--------------------|--------------|---------------------|-------------|-------------|
| url              | Link mask hỏng hóc | Bắt buộc     | TEXT                | 1,255       | {{maskUrl}} |
| damageTypeName   | Loại hỏng hóc      | Bắt buộc     | TEXT                | 1,255       | Trầy (xước) |
| damagePercentage | Phần trăm hỏng hóc | Bắt buộc     | Number              | 1,9999      | 0.08        |

#### d. Ví dụ đầu ra

```
{
    "result": {
        "images": [
            {
                "imageId": 16532,
                "imageName": "1679373021917.jpg",
                "filePath": "INSURANCE/1679373021956/1679373021917.jpg",
                "url": "{{imageUrl}}",
                "imageSize": [
                    1920,
                    1080
                ],
                "directionName": "45° Trái - Sau",
                "imageRangeName": "Toàn cảnh",
                "timeProcess": 1.693,
                "timeAppUpload": 1.501,
                "requestedTime": "2023-03-24T03:28:29.414Z",
                "uploadedTime": "2023-03-27T03:04:53.001Z",
                "uploadLocation": "Quận Cầu Giấy, Hà Nội",
                "location": "Duy Tân, Cầu Giấy, Hà Nội",
                "errorType": null,
                "errorNote": null,
                "traceId": "265e70b629c05b353b1968693a4ca051",
                "damageInfo": [
                    {
                        "vehiclePartName": "Pa vô lê trái",
                        "vehiclePartExcelId": "pavole-trai-tEm7AB",
                        "price": 450000,
                        "laborCost": 0,
                        "totalCost": 450000,
                        "area": 70,
                        "location": "Trái",
                        "repairPlan": "Sửa chữa",
                        "damageDetail": [
                            {
                                "filePath": "fxVIBETAr_Buc1h6ngzC4.png",
                                "url": "{{maskUrl}}",
                                "damageTypeName": "Trầy (xước)",
                                "damageTypeColor": "#FFEC05",
                                "damageUuid": "yfMzer07THdYoCI1SM2LN",
                                "boxes": [
                                    0.3125,
                                    0.6731481481481482,
                                    0.32135416666666666,
                                    0.6953703703703704
                                ],
                                "damageArea": 0.6255235966733421,
                                "damagePercentage": 0.008936051381047744
                            }
                        ]
                    }
                ]
            }
     ]
   }
 }       
```

### **3.7: API kiểm tra các ảnh trong Hồ sơ**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/checkCar/{claimId} |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

| **Tên Tham số** | **Mô tả** | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-----------|--------------|------------------|-------------|-----------|
| claimId         | Id Hồ sơ | Bắt buộc     | Number           | 1,999999    | 1111      |

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**             |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**    |
|-----------------|-----------------------|--------------|------------------|-------------|--------------|
| state           | trạng thái của folder | Bắt buộc     | Number           | 1,999999    | 1            |
| message         | thông báo chi tiết về trạng thái của folder            | Bắt buộc     | Text             | 1,255       | Các ảnh thuộc cùng một xe |

#### d. Chi tiết các state
**Loại đầu ra**: Response body

| **state** | **Mô tả** |
|-----------| ----------- |
| 0         |trạng thái mặc định, các ảnh thuộc cùng một xe|
| 1         |chỉ có 1 ảnh trong folder|
| 2         |có ít nhất 1 ảnh trong folder chụp không đúng góc|
| 3         |các ảnh trong folder thuộc cùng một xe|
| 4         |các ảnh trong folder không thuộc cùng một xe|
| 5         |không nhận diện được biển số nào trong folder|
| 6         |thư mục ảnh rỗng|

#### e. Ví dụ đầu ra
```
{
    state: 3,
    message: 'Các ảnh cùng một xe'
 } 

```

#### f. Ví dụ curl
```
curl --location --request GET 'https://api.aicycle.ai/insurance/checkCar/1111' \
--header 'Authorization: Bearer {API Key}'

```


### **3.8: API callback lưu kết quả từ khách hàng**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/claimfolders/{claimFolderId}/results-callback |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| claimFolderId   |Id của Hồ sơ|Bắt buộc|Number|1,999999|123|

**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả** | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------|--------|---------|------|-------|-----|
| carPlate | Biển số xe | Optional  | TEXT | 1,255 | 30E 64737 |
| carCompany | Hãng xe | Optional | TEXT | 1,255 | HYUNDAI |
| carModel | Mẫu xe | Optional | TEXT | 1,255 | i10 |
| damages | Danh sách các hỏng hóc | Optional| Array[damage] | n     | [] |

*Chi tiết Object item `damage`*

|**Tên Tham số**|**Mô tả**| **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---|---|----------|------|-----|----------|
|damageId|ID hỏng hóc (định danh hỏng hóc ở hệ thống khách hàng) (nếu có)| Optional | TEXT | 1,255 | 123      |
|damageType|Loại hỏng hóc (theo định nghĩa của khách hàng) (nếu có)| Optional | TEXT | 1,255 | Trầy (Xước) |
|damagePart|Bộ phận ghi nhận hỏng hóc (theo định nghĩa của khách hàng) (nếu có)| Optional | TEXT | 1,255 |Capo trước|
|damageName|Tên hỏng hóc được định nghĩa phía client (tùy chọn)| Optional |TEXT|1,255|Trầy (Xước) Capo Trước|


**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/claimfolders/52/results-callback' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data-raw '{
  "carPlate": "30A 115.23",
  "carCompany": "HYUNDAI",
  "carModel": "i10",
  "damages": [
    {
      "damageId": '123',
      "damageType": "Trầy (Xước)",
      "damagePart": "Capo trước",
      "damageName": "Trầy Capo Trước"
    }
  ]
}'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**:Response body

|**Tên Tham số**| **Mô tả**        | **Bắt buộc** | **Kiểu dữ liệu** | **Min, Max**           | **Ví dụ**              |
|---|------------------|--------------|------|------------------|------------------------|
|claimId| Tên folder claim | Bắt buộc     | Number | 1,999999         | 123                    
|carPlate| Biển số xe       | Bắt buộc     | TEXT | 1,255            | 30E 64737             
|carCompany| Hãng xe          | Bắt buộc     | TEXT | 1,255            | HYUNDAI              
|carModel| Hiệu xe          | Bắt buộc     |TEXT| 1,255            | i10                    
|damages| Danh sách các hỏng hóc     | Bắt buộc     |Array[damage]| n                | []                     
|ownerOrganizationId| Id tổ chức      | Bắt buộc     |Number| 1,999999            | 1 



*Chi tiết Object item `damage`*


|**Tên Tham số**|**Mô tả**| **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---|---|--------------|------|-----|----------|
|damageId|ID hỏng hóc| Bắt buộc     | TEXT | 1,255 | 123      |
|damageType|Loại hỏng hóc| Bắt buộc     | TEXT | 1,255 | Trầy (Xước) |
|damagePart|Bộ phận ghi nhận hỏng hóc| Bắt buộc     | TEXT | 1,255 |Capo trước|
|damageName|Tên hỏng hóc| Bắt buộc     |TEXT|1,255|Trầy (Xước) Capo Trước| 

#### d. Ví dụ đầu ra

```
{
    "claimId": 123,
    "carPlate": "30A 11523",
    "carCompany": "HYUNDAI",
    "carModel": "i10",
    "damages": [
        {
            "damageId": '123',
            "damageType": "Trầy (Xước)",
            "damagePart": "Capo trước",
            "damageName": "Trầy Capo Trước"
        }
    ],
    "ownerOrganizationId": 2
}
```

### 3.9 APIs tích hợp Claim Me
##### a. Thông tin cơ bản

||                                                      |
|----|------------------------------------------------------|
| Method | POST                                                 |
| API Url | https://api.aicycle.ai/insurance/v2/claim-me/process |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`          |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                   |
|-----------------|--------------------------------------------------------------------------|----------|------------------|-------------|-----------------------------|
| claimId         | Id của folder                                                            | Bắt buộc | Number           | 1,999999    | 123                         |
| imageName       | Tên ảnh                                                                  | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| filePath        | Path ảnh trên s3 (lấy từ kết quả trả về  của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| position        | Slug Loại ảnh (Toàn cảnh, trung cảnh, cận cảnh)                          | Bắt buộc | Text             | 1,255       | toan-canh-afh4l5            |
| direction       | Slug Hướng ảnh                                                           | Bắt buộc | Text             | 1,255       | 45-trai-truoc-C1xM02        |
| oldImageId      | Id ảnh cũ muốn xóa                                                       | Optional | Number           | 1,9999      | 3                           |
| vehicleType     | Loại xe muốn engine detect (xe tải, xe con)                              | Optional | ENUM(truck, car) | 1,255       | truck                       |





*Lưu ý*
> **Các giá trị của `position`**
>
>| positionName | positionSlug |
>|--------------|-----------|
>| Toàn cảnh    | toan-canh-afh4l5          |
>| Trung cảnh   | trung-canh-0s8mnb          |
>| Cận cảnh     | can-canh-czu5jp          |

> **Các giá trị của `direction`**
>
>| directionName    | directionSlug |
>|------------------|---|
>| Trước            | truoc-sT9qgX  |
>| 45° Phải - Trước | 45-phai-truoc-UoYzs6  |
>| 45° Trái - Trước | 45-trai-truoc-C1xM02  |
>| Sau              | sau-htBwjB  |
>| 45° Phải - Sau   | 45-phai-sau-fRzY3r  |
>| 45° Trái - Sau   | 45-trai-sau-1q3G3J  |
>| Phải - Trước     | phai-truoc-eYWg1d  |
>| Trái - Trước     | trai-truoc-r6BEZd  |
>| Phải - Sau       | phai-sau-v1hAm6  |
>| Trái - Sau       | trai-sau-t8QgFO  |

**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/v2/claim-me/process' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimId": 1,
    "imageName": "INSURANCE/1666145614576/1666145614447.jpg",
    "filePath": "INSURANCE/1666145614576/1666145614447.jpg",
    "position": "toan-canh-afh4l5",
    "direction": "45-trai-truoc-C1xM02",
     “oldImageId": 123
}'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "isPhotoValid": <Ảnh có hợp lệ hay không (Boolean)>,
    "traceId": <ID dùng khi liên hệ với AICycle (UUID)>,
    "errorCodeFromEngine": <Mã lỗi trả về từ AICycle Engine (Number)>,
    "message": <Message lỗi trả về từ AICycle Engine (String)>,
    "imageId": <ID của ảnh (Number)>,
    "result": [<Image>]
}
```

*Chi tiết Object item `Image`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| imgSize             | Kích thước ảnh       |Bắt buộc| Array[number]    | n           | [1920, 1080] |
| imgUrl              | Link ảnh gốc         |Bắt buộc| Text             | 1,255       | {{s3Link}} |
| errorCodeFromEngine | Mã lỗi ảnh           |Bắt buộc| Number           | 1,999999    | 0       |
| message             | Chi tiết lỗi ảnh     |Bắt buộc| Text             | 1,255       | Ảnh chụp qua màn hình |
| damages             | Các hỏng hóc của ảnh |Bắt buộc| Array[damage]    | n           |         |
| carParts            | Các bộ phận của ảnh  |Bắt buộc| Array[carPart]   | n           |         |

*Chi tiết Object item  `carPart`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| carPartKey      |Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
| position        |Vị trí bộ phận|Bắt buộc|Text|1,255|Trái - Trước|
| name            |Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
| maskUrl         |Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Chi tiết Object item `damage`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| name            |Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
| maskUrl         |mask hỏng hóc|Bắt buộc|Text|1,255|...|




***Chi tiết Bảng Mã lỗi cùng httpStatus trả về của `errorCodeFromEngine`***

*Chú thích*
> Với những lỗi có level là error sẽ chỉ trả ra mã lỗi và message lỗi, những lỗi level warning sẽ trả ra mã lỗi, message, và car_part, car_damages như bình thường

|**Mã lỗi**|**HTTP Status**|**Response body**|**Level lỗi**|
|----|----|----|----|
|0|200|Ảnh hợp lệ||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Ảnh chụp bị mờ. Vui lòng chụp lại"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Không download được ảnh"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Ảnh chụp bị tối"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Ảnh chụp qua màn hình. Vui lòng chụp lại"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Trả ra error của engine"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Ảnh chụp bị lóa"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Ảnh không đúng góc chụp. Vui lòng chụp lại"}`|Error|
|378224|400|`{"errorCodeFromEngine": 378224, "message": "Ảnh không phải xe tải"}`|Error|
|178434|400|`{"errorCodeFromEngine": 178434, "message": "Ảnh không phải xe con"}`|Error|

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "isPhotoValid": true,
    "traceId": "80540cf80585486f63432dcdf680fd20",
    "errorCodeFromEngine": 0,
    "message": "",
    "imageId": 4808,
    "result": [
        {
            "imgSize": [1920, 1080],
            "damages": [
                {
                    "name": "Móp, bẹp(thụng)",
                    "damageKey": "mop-bep-jtep4m",
                    "location": "",
                    "score": 1,
                    "box": [ 0.34, 0.14, 0.79,0.77],
                    "maskPath": "nfU_HFiSOi4wEkP1ycUwv.png",
                    "isPart": false,
                    "maskUrl": "{{s3Url}}"
                },
                ...
            ],
            "carParts": [
                {
                    "name": "Trụ kính cánh cửa",
                    "carPartKey": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
                    "position": "Trái - Trước",
                    "score": 0.83660888671875,
                    "box": [
                        0.81875,
                        0,
                        0.9994791666666667,
                        0.2
                    ],
                    "maskPath": "vG4wVxBeWvLF5MFGMAfq9.png",
                    "isPart": true,
                    "damages": [
                        {
                            "name": "Trầy, xước",
                            "damageKey": "yfMzer07THdYoCI1SM2LN",
    
                            "score": 1,
                            "box": [
                                0.2171875,
                                0.07685185185185185,
                                0.9989583333333333,
                                0.9425925925925925
                            ],
                            "maskPath": "favy1yqepBr_JM8tAJm2v.png",
                            "isPart": false,
                            "overlapRate": 0.00047147571877818216,
                            "maskUrl": "{{s3Url}}",
                        },
                        ...
                    ],
                },
                ...
            ]
        }
    ]
}
```

### 3.10 APIs tích hợp Buy Me V1 (deprecated)
##### a. Thông tin cơ bản

||                                                                    |
|----|--------------------------------------------------------------------|
| Method | POST                                                               |
| API Url | https://api.aicycle.ai/insurance/claimimages/triton-assessment-box |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                        |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                   |
|-----------------|--------------------------------------------------------------------------|----------|------------------|-------------|-----------------------------|
| claimId         | Id của folder                                                            | Bắt buộc | Number           | 1,999999    | 123                         |
| imageName       | Tên ảnh                                                                  | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| filePath        | Path ảnh trên s3 (lấy từ kết quả trả về  của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| imageRangeId    | Id Loại ảnh (Toàn cảnh, trung cảnh, cận cảnh)                            | Bắt buộc | Number           | 1,999999       | 1                           |
| partDirectionId | Id Hướng ảnh                                                             | Bắt buộc | Number             | 1,999999        | 2                           |
| oldImageId      | Id ảnh cũ muốn xóa                                                       | Optional | Number           | 1,9999      | 3                           |
| vehicleType     | Loại xe muốn engine detect (xe tải, xe con)                              | Optional | ENUM(truck, car) | 1,255       | truck                       |





*Lưu ý*
> **Các giá trị của `imageRangeId`**
>
>| imageRangeName | positionSlug |
>|----------------|--------------|
>| Toàn cảnh      | 1            |
>| Trung cảnh     | 2            |
>| Cận cảnh       | 3            |

> **Các giá trị của `direction`**
>
>| directionName    | directionSlug |
>|------------------|---------------|
>| Trước            | 2             |
>| 45° Phải - Trước | 3             |
>| 45° Trái - Trước | 4             |
>| Sau              | 5             |
>| 45° Phải - Sau   | 6             |
>| 45° Trái - Sau   | 7             |
>| Phải - Trước     | 8             |
>| Trái - Trước     | 9             |
>| Phải - Sau       | 10            |
>| Trái - Sau       | 11            |

**Ví dụ**
```
curl --location --request POST 'https://api.aicycle.ai/insurance/v2/claim-me/process' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimId": 1,
    "imageName": "INSURANCE/1666145614576/1666145614447.jpg",
    "filePath": "INSURANCE/1666145614576/1666145614447.jpg",
    "imageRangeId": 1,
    "partDirectionId": 2,
     “oldImageId": 123
}'
```

***Chi tiết Bảng Mã lỗi cùng httpStatus trả về của `errorCodeFromEngine`***

*Chú thích*
> Với những lỗi có level là error sẽ chỉ trả ra mã lỗi và message lỗi, những lỗi level warning sẽ trả ra mã lỗi, message, và car_part, car_damages như bình thường

|**Mã lỗi**|**HTTP Status**|**Response body**|**Level lỗi**|
|----|----|----|----|
|0|200|Ảnh hợp lệ||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Ảnh chụp bị mờ. Vui lòng chụp lại"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Không download được ảnh"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Ảnh chụp bị tối"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Ảnh chụp qua màn hình. Vui lòng chụp lại"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Trả ra error của engine"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Ảnh chụp bị lóa"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Ảnh không đúng góc chụp. Vui lòng chụp lại"}`|Error|
|378224|400|`{"errorCodeFromEngine": 378224, "message": "Ảnh không phải xe tải"}`|Error|
|178434|400|`{"errorCodeFromEngine": 178434, "message": "Ảnh không phải xe con"}`|Error|

#### c. Ví dụ đầu ra
```
{
    "status": "success",
    "isCarExistInProfile": false,
    "isPhotoValid": true,
    "traceId": "80540cf80585486f63432dcdf680fd20",
     “errorCodeFromEngine”: 0,
     “message: “: “”,
    "imageId": 4808,
    "result": [
        {
            "img_size": [1920, 1080],
            "extra_infor": {
                "plate_number": "",
                "chassis_number": null,
                "car_company": "",
                "car_model": "",
                "car_color": [222, 220,216],
                "corner": "Trái - Trước",
                "imagePosition": 1
            },
            "car_damages": [
                {
                    "class": "Móp, bẹp(thụng)",
                    "class_uuid": "zmMJ5xgjmUpqmHd99UNq3",
                    "location": "",
                    "score": 1,
                    "box": [ 0.34, 0.14, 0.79,0.77],
                    "mask_path": "nfU_HFiSOi4wEkP1ycUwv.png",
                    "is_part": false,
                    "mask_url": "{{s3Url}}"
                },
	    ….
            ],
            "car_parts": [
                {
                    "class": "Trụ kính cánh cửa",
                    "class_uuid": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
                    "location": "Trái - Trước",
                    "score": 0.83660888671875,
                    "box": [
                        0.81875,
                        0,
                        0.9994791666666667,
                        0.2
                    ],
                    "mask_path": "vG4wVxBeWvLF5MFGMAfq9.png",
                    "is_part": true,
                    "damages": [
                        {
                            "class": "Trầy, xước",
                            "class_uuid": "yfMzer07THdYoCI1SM2LN",
                            "location": "",
                            "score": 1,
                            "box": [
                                0.2171875,
                                0.07685185185185185,
                                0.9989583333333333,
                                0.9425925925925925
                            ],
                            "mask_path": "favy1yqepBr_JM8tAJm2v.png",
                            "is_part": false,
                            "overlap_rate": 0.00047147571877818216,
                            "claimId": 1,
                            "imageId": "4808",
                            "isMaskDuplicate": false,
                            "mask_url": "{{s3Url}}",
                            "damage_type_name": "Trầy (xước)",
                            "damage_type_color": "#FFEC05"
                        },
		…..
                    ],
                    "car_part_name": "Khung kính cánh cửa trước trái",
                    "rawLocation": "Trái - Trước",
                    "car_part_color": "#21E0C1",
                    "mask_url": "{{s3Url}}"
                },
	…..
            ],
            "img_url": "{{s3Url}}"
        }
    ]
}

```


### 3.11 APIs tích hợp Claim Me theo form data
##### a. Thông tin cơ bản

||                                                      |
|----|------------------------------------------------------|
| Method | POST                                                 |
| API Url | https://api.aicycle.ai/insurance/v2/claim-me/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`          |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                   |
|-----------------|--------------------------------------------------------------------------|----------|------------------|-------------|-----------------------------|
| img         | File ảnh xe                                                            | Bắt buộc | File           | n    |                          |
| claimFolderId       | Id Folder của AIcycle                                                                   | Optional | Number             | 1,99999       | 123 |
| externalSessionId        | Id folder (giấy chứng nhận)  của đối tác | Optional | Text             | n       | folderId |
| directionSlug        | Slug Hướng ảnh                          | Optional | Text             | 1,255       | 45-trai-truoc-C1xM02             |
| isValidate       |       Flag bật tắt validate góc chụp                                                     | Optional | Boolean             | n       |     true  |





*Lưu ý*
> **Các giá trị của `position`**
>
>| positionName | positionSlug |
>|--------------|-----------|
>| Toàn cảnh    | toan-canh-afh4l5          |
>| Trung cảnh   | trung-canh-0s8mnb          |
>| Cận cảnh     | can-canh-czu5jp          |

> **Các giá trị của `direction`**
>
>| directionName    | directionSlug |
>|------------------|---|
>| Trước            | truoc-sT9qgX  |
>| 45° Phải - Trước | 45-phai-truoc-UoYzs6  |
>| 45° Trái - Trước | 45-trai-truoc-C1xM02  |
>| Sau              | sau-htBwjB  |
>| 45° Phải - Sau   | 45-phai-sau-fRzY3r  |
>| 45° Trái - Sau   | 45-trai-sau-1q3G3J  |
>| Phải - Trước     | phai-truoc-eYWg1d  |
>| Trái - Trước     | trai-truoc-r6BEZd  |
>| Phải - Sau       | phai-sau-v1hAm6  |
>| Trái - Sau       | trai-sau-t8QgFO  |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/v2/claim-me/upload' \
--header 'Authorization: Bearer ${APIKEY}' \
--form '${fileImage}' \
--form 'externalSessionId="${externalId}"' \
--form 'directionSlug="45-trai-truoc-C1xM02"' \
--form 'isValidate="true"'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "isPhotoValid": <Ảnh có hợp lệ hay không (Boolean)>,
    "traceId": <ID dùng khi liên hệ với AICycle (UUID)>,
    "errorCodeFromEngine": <Mã lỗi trả về từ AICycle Engine (Number)>,
    "message": <Message lỗi trả về từ AICycle Engine (String)>,
    "imageId": <ID của ảnh (Number)>,
    "result": [<Image>]
}
```

*Chi tiết Object item `Image`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| imgSize             | Kích thước ảnh       |Bắt buộc| Array[number]    | n           | [1920, 1080] |
| imgUrl              | Link ảnh gốc         |Bắt buộc| Text             | 1,255       | {{s3Link}} |
| errorCodeFromEngine | Mã lỗi ảnh           |Bắt buộc| Number           | 1,999999    | 0       |
| message             | Chi tiết lỗi ảnh     |Bắt buộc| Text             | 1,255       | Ảnh chụp qua màn hình |
| damages             | Các hỏng hóc của ảnh |Bắt buộc| Array[damage]    | n           |         |
| carParts            | Các bộ phận của ảnh  |Bắt buộc| Array[carPart]   | n           |         |
| imgDrawUrl              | Link ảnh đã vẽ sẵn vết hỏng         |Bắt buộc| Text             | 1,255       | {{s3Link}} |

*Chi tiết Object item  `carPart`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| carPartKey      |Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
| position        |Vị trí bộ phận|Bắt buộc|Text|1,255|Trái - Trước|
| name            |Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
| maskUrl         |Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Chi tiết Object item `damage`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| name            |Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
| maskUrl         |mask hỏng hóc|Bắt buộc|Text|1,255|...|




***Chi tiết Bảng Mã lỗi cùng httpStatus trả về của `errorCodeFromEngine`***

*Chú thích*
> Với những lỗi có level là error sẽ chỉ trả ra mã lỗi và message lỗi, những lỗi level warning sẽ trả ra mã lỗi, message, và car_part, car_damages như bình thường

|**Mã lỗi**|**HTTP Status**|**Response body**|**Level lỗi**|
|----|----|----|----|
|0|200|Ảnh hợp lệ||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Ảnh chụp bị mờ. Vui lòng chụp lại"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Không download được ảnh"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Ảnh chụp bị tối"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Ảnh chụp qua màn hình. Vui lòng chụp lại"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Trả ra error của engine"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Ảnh chụp bị lóa"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Ảnh không đúng góc chụp. Vui lòng chụp lại"}`|Error|
|378224|400|`{"errorCodeFromEngine": 378224, "message": "Ảnh không phải xe tải"}`|Error|
|178434|400|`{"errorCodeFromEngine": 178434, "message": "Ảnh không phải xe con"}`|Error|

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "isPhotoValid": true,
    "traceId": "80540cf80585486f63432dcdf680fd20",
    "errorCodeFromEngine": 0,
    "message": "",
    "imageId": 4808,
    "result": [
        {
            "imgSize": [1920, 1080],
            "imageUrl": {{s3Link}},
            "imageDrawUrl": {{s3Link}}
            "damages": [
                {
                    "name": "Móp, bẹp(thụng)",
                    "damageKey": "mop-bep-jtep4m",
                    "location": "",
                    "score": 1,
                    "box": [ 0.34, 0.14, 0.79,0.77],
                    "maskPath": "nfU_HFiSOi4wEkP1ycUwv.png",
                    "isPart": false,
                    "maskUrl": "{{s3Url}}"
                },
                ...
            ],
            "carParts": [
                {
                    "name": "Trụ kính cánh cửa",
                    "carPartKey": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
                    "position": "Trái - Trước",
                    "score": 0.83660888671875,
                    "box": [
                        0.81875,
                        0,
                        0.9994791666666667,
                        0.2
                    ],
                    "maskPath": "vG4wVxBeWvLF5MFGMAfq9.png",
                    "isPart": true,
                    "damages": [
                        {
                            "name": "Trầy, xước",
                            "damageKey": "yfMzer07THdYoCI1SM2LN",
    
                            "score": 1,
                            "box": [
                                0.2171875,
                                0.07685185185185185,
                                0.9989583333333333,
                                0.9425925925925925
                            ],
                            "maskPath": "favy1yqepBr_JM8tAJm2v.png",
                            "isPart": false,
                            "overlapRate": 0.00047147571877818216,
                            "maskUrl": "{{s3Url}}",
                        },
                        ...
                    ],
                },
                ...
            ]
        }
    ]
}
```


### 3.12 APIs tích hợp Buy Me theo form data
##### a. Thông tin cơ bản

||                                                      |
|----|------------------------------------------------------|
| Method | POST                                                 |
| API Url | https://api.aicycle.ai/insurance/v2/buy-me/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`          |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                   |
|-----------------|--------------------------------------------------------------------------|----------|------------------|-------------|-----------------------------|
| img         | File ảnh xe                                                            | Bắt buộc | File           | n    |                          |
| claimFolderId       | Id Folder của AIcycle                                                                   | Optional | Number             | 1,99999       | 123 |
| externalSessionId        | Id folder (giấy chứng nhận)  của đối tác | Optional | Text             | n       | folderId |
| directionSlug        | Slug Hướng ảnh                          | Optional (bắt buộc nếu bật flag isValidate) | Text             | 1,255       | 45-trai-truoc-C1xM02             |
| isValidate       |       Flag bật tắt validate góc chụp                                                     | Optional | Boolean             | n       |     true  |
| carCompany       |       Mã (tên) hãng xe của khách hàng cần validate                                                 | Optional (bắt buộc nếu khách hàng yêu cầu validate hãng, hiệu, biển số) | Text             | 1,255       |     KIA  |
| carModel       |       Mã (tên) hiệu xe của khách hàng cần validate                                                 | Optional (bắt buộc nếu khách hàng yêu cầu validate hãng, hiệu, biển số) | Text             | 1,255       |     Morning  |
| licensePlate       |       Biển số của khách hàng cần validate                                                 | Optional (bắt buộc nếu khách hàng yêu cầu validate hãng, hiệu, biển số) | Text             | 1,255       |     30A99999  |





*Lưu ý*
> **Các giá trị của `position`**
>
>| positionName | positionSlug |
>|--------------|-----------|
>| Toàn cảnh    | toan-canh-afh4l5          |
>| Trung cảnh   | trung-canh-0s8mnb          |
>| Cận cảnh     | can-canh-czu5jp          |

> **Các giá trị của `direction`**
>
>| directionName    | directionSlug |
>|------------------|---|
>| Trước            | truoc-sT9qgX  |
>| 45° Phải - Trước | 45-phai-truoc-UoYzs6  |
>| 45° Trái - Trước | 45-trai-truoc-C1xM02  |
>| Sau              | sau-htBwjB  |
>| 45° Phải - Sau   | 45-phai-sau-fRzY3r  |
>| 45° Trái - Sau   | 45-trai-sau-1q3G3J  |
>| Phải - Trước     | phai-truoc-eYWg1d  |
>| Trái - Trước     | trai-truoc-r6BEZd  |
>| Phải - Sau       | phai-sau-v1hAm6  |
>| Trái - Sau       | trai-sau-t8QgFO  |
>| Tem đăng kiểm       | tem-dang-kiem-LC81Ar  |
>| Taplo       | tap-lo-H4SHs1  |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/v2/buy-me/upload' \
--header 'Authorization: Bearer ${APIKEY}' \
--form '${fileImage}' \
--form 'externalSessionId="${externalId}"' \
--form 'directionSlug="45-trai-truoc-C1xM02"' \
--form 'isValidate="true"'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "isPhotoValid": <Ảnh có hợp lệ hay không (Boolean)>,
    "traceId": <ID dùng khi liên hệ với AICycle (UUID)>,
    "errorCodeFromEngine": <Mã lỗi trả về từ AICycle Engine (Number)>,
    "message": <Message lỗi trả về từ AICycle Engine (String)>,
    "imageId": <ID của ảnh (Number)>,
    "result": [<Image>],
    "stateFolderCode": <Trạng thái của folder hiện tại (có cùng 1 xe hay không) (Number)>
    "stateFolderMessage": <Trạng thái của folder hiện tại (có cùng 1 xe hay không) (String)>
}
```

*Chi tiết Object item `Image`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| imgSize             | Kích thước ảnh       |Bắt buộc| Array[number]    | n           | [1920, 1080] |
| imgUrl              | Link ảnh gốc         |Bắt buộc| Text             | 1,255       | {{s3Link}} |
| errorCodeFromEngine | Mã lỗi ảnh           |Bắt buộc| Number           | 1,999999    | 0       |
| message             | Chi tiết lỗi ảnh     |Bắt buộc| Text             | 1,255       | Ảnh chụp qua màn hình |
| damages             | Các hỏng hóc của ảnh |Bắt buộc| Array[damage]    | n           |         |
| carParts            | Các bộ phận của ảnh  |Bắt buộc| Array[carPart]   | n           |         |
| imgDrawUrl              | Link ảnh đã vẽ sẵn vết hỏng         |Bắt buộc| Text             | 1,255       | {{s3Link}} |
| extraInfor              | Các thông tin thêm của xe         |Bắt buộc| Object             | n       |  |


*Chi tiết Object item  `carPart`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| carPartKey      |Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
| position        |Vị trí bộ phận|Bắt buộc|Text|1,255|Trái - Trước|
| name            |Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
| maskUrl         |Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Chi tiết Object item `damage`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| name            |Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
| maskUrl         |mask hỏng hóc|Bắt buộc|Text|1,255|...|

*Chi tiết Object item `extraInfor`*

| **Tên Tham số** |**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|-----------------|---|---|---|---|---|
| plateNumber            |Biển số xe|Bắt buộc|Text|1,255|30A 99999|
| carCompany         |Hãng xe|Bắt buộc|Text|1,255|PEUGEOT|
| carModel         |Hiệu xe|Bắt buộc|Text|1,255|3008|
| carShape         |Loại xe|Bắt buộc|Text|1,255|SUV_CROSSOVER|
| carColor        |Màu xe|Bắt buộc|Array[number]|n|[ 200,136,72]|



***Chi tiết Bảng Mã lỗi cùng httpStatus trả về của `errorCodeFromEngine`***

*Chú thích*
> Với những lỗi có level là error sẽ chỉ trả ra mã lỗi và message lỗi, những lỗi level warning sẽ trả ra mã lỗi, message, và car_part, car_damages như bình thường

|**Mã lỗi**|**HTTP Status**|**Response body**|**Level lỗi**|
|----|----|----|----|
|0|200|Ảnh hợp lệ||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Ảnh chụp bị mờ. Vui lòng chụp lại"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Không download được ảnh"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Ảnh chụp bị tối"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Ảnh chụp qua màn hình. Vui lòng chụp lại"}`|Error|
|47565|500|`{"errorCodeFromEngine": 47565, "message": "Trả ra error của engine"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Không thể nhận diện ô tô trong ảnh. Vui lòng chụp lại"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Ảnh chụp bị lóa"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Ảnh không đúng góc chụp. Vui lòng chụp lại"}`|Error|
|10012|408|`{"errorCodeFromEngine": 10012, "message": "Request Timeout"}`|Error|
|4343|400|`{"errorCodeFromEngine": 4343, "message": "Ảnh không phải tem đăng kiểm"}`|Error|
|1588|400|`{"errorCodeFromEngine": 1588, "message": "Ảnh không phải taplo"}`|Error|

***Chi tiết Bảng Mã và message của `stateFolderCode`***
>| stateFolderCode    | message |
>|------------------|---|
>| 1         |chỉ có 1 ảnh trong folder|
>| 3         |các ảnh trong folder thuộc cùng một xe|
>| 4         |các ảnh trong folder không thuộc cùng một xe|
>| 12            | Hồ sơ có thể có ảnh không đảm bảo chất lượng. Khách hàng đã up ${numOfImageErr}. Vui lòng check lại hồ sơ.  |
>| 13 | Hồ sơ có thể được chụp trong không gian tối, khách hàng đã up ${numOfImageErr} ảnh tối. Vui lòng check lại hồ sơ.  |
>| 14 | Hồ sơ có thể có ảnh mờ, khách hàng đã up ${numOfImageErr} ảnh mờ. Vui lòng check lại hồ sơ.  |
>| 15              | Hồ sơ có thể có ảnh lóa, khách hàng đã up ${numOfImageErr} ảnh lóa. Vui lòng check lại hồ sơ.  |
>| 16   | Hồ sơ có dấu hiệu gian lận, khách hàng đã up ${numOfImageErr} ảnh chụp qua màn hình. Vui lòng check lại hồ sơ.  |
>| 17   | Folder hiện tại chưa có ảnh xe  |

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "isPhotoValid": true,
    "traceId": "80540cf80585486f63432dcdf680fd20",
    "errorCodeFromEngine": 0,
    "message": "",
    "imageId": 4808,
    "stateFolderCode": 3,
    "stateFolderMessage": "Các ảnh cùng một xe",
    "result": {
            "imgSize": [1920, 1080],
            "imageUrl": {{s3Link}},
            "imageDrawUrl": {{s3Link}},
            "extraInfor": {
              "imageDirection": "45-phai-truoc-UoYzs6",
              "additionalCornerInfor": "45° Phải - Trước",
              "imagePosition": "toan-canh-afh4l5",
              "plateNumber": "30F 04872",
              "chassisNumber": null,
              "carCompany": "PEUGEOT",
              "carModel": "3008",
              "carShape": "SUV_CROSSOVER",
              "carColor": [
                  200,
                  136,
                  72
              ]
            }, 
            "damages": [
                {
                    "name": "Móp, bẹp(thụng)",
                    "damageKey": "mop-bep-jtep4m",
                    "location": "",
                    "score": 1,
                    "box": [ 0.34, 0.14, 0.79,0.77],
                    "maskPath": "nfU_HFiSOi4wEkP1ycUwv.png",
                    "isPart": false,
                    "maskUrl": "{{s3Url}}"
                },
                ...
            ],
            "carParts": [
                {
                    "name": "Trụ kính cánh cửa",
                    "carPartKey": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
                    "position": "Trái - Trước",
                    "score": 0.83660888671875,
                    "box": [
                        0.81875,
                        0,
                        0.9994791666666667,
                        0.2
                    ],
                    "maskPath": "vG4wVxBeWvLF5MFGMAfq9.png",
                    "isPart": true,
                    "damages": [
                        {
                            "name": "Trầy, xước",
                            "damageKey": "yfMzer07THdYoCI1SM2LN",
    
                            "score": 1,
                            "box": [
                                0.2171875,
                                0.07685185185185185,
                                0.9989583333333333,
                                0.9425925925925925
                            ],
                            "maskPath": "favy1yqepBr_JM8tAJm2v.png",
                            "isPart": false,
                            "overlapRate": 0.00047147571877818216,
                            "maskUrl": "{{s3Url}}",
                        },
                        ...
                    ],
                },
                ...
            ]
        }
}
```

#### e. Chi tiết đầu ra với ảnh góc tem đăng kiểm

**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "errorCodeFromEngine": <Mã lỗi trả về từ AICycle Engine (Number)>,
    "message": <Message lỗi trả về từ AICycle Engine (String)>,
    "stampImageId": <ID của ảnh tem đăng kiểm (Number)>,
    "result": {
      "imgUrl": <Url ảnh tem đăng kiểm (String)>,
      "stampId": <Id của ảnh tem đăng kiểm (String)>,
      "licensePlate": <Biển số xe của tem đăng kiểm (String)>,
      "expiredMonth": <Ngày hết hạn theo tháng (String)>,
      "expiredDate: <Ngày hết hạn theo ngày (String)>
    },
    "stateFolderCode": <Trạng thái của folder hiện tại (có cùng 1 xe hay không) (Number)>
    "stateFolderMessage": <Trạng thái của folder hiện tại (có cùng 1 xe hay không) (String)>
}
```

#### f. Chi tiết đầu ra với ảnh taplo

**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "errorCodeFromEngine": <Mã lỗi trả về từ AICycle Engine (Number)>,
    "message": <Message lỗi trả về từ AICycle Engine (String)>,
    "taploImageId": <ID của ảnh taplo (Number)>,
    "result": {
      "imgUrl": <Url ảnh tem đăng kiểm (String)>,
      "taploIconInfo": [
        {
          "iconName": <Tên icon mà AI đọc được (String)>,
          "iconBoundingBox": <Bounding Box của Icon hiện tại (Array<Number>)>
        }
      ]
    }
}
```



### **3.13: API lấy kết quả Hồ sơ (nhóm theo ảnh) V2**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/v2/claimfolders/{externalSessionId}/external-image-result |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|externalSessionId|Id của giấy chứng nhận (folder của khách hàng)|Bắt buộc|Text|1,255|folder1|

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/v2/claimfolders/{externalSessionId}/external-image-result' \
--header 'authorization: Bearer $$API_KEY$$' \
--data-raw ''
```
#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "results": {
        images: [<Image>]
    }
}
```

*Chi tiết Object item `Image`*

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu**  | **Min,Max** | **Ví dụ**   |
|-----------------|-------------------|---|-------------------|-------------|-------------|
| url             | Link ảnh          |Bắt buộc| Text              | 1,255       | {{imgUrl}}  |
| imageSize       | Kích thước ảnh    |Bắt buộc| Array[number]     | n           | [1920,1080] |
| directionName   | Góc chụp          |Bắt buộc| Text              | 1,255       | Trước       |
| imageRangeName  | Loại ảnh          |Bắt buộc| Text              | 1,255       | Toàn cảnh   |
|location | Nơi ảnh được chụp | Bắt buộc | TEXT | 1,255 | Duy Tân, Cầu Giấy, Hà Nội |
|requestedTime | Thời gian chụp ảnh | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
| uploadedTime | Thời gian ảnh được upload lên hệ thống | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
|uploadLocation | Nơi upload ảnh để thực hiện claim | Bắt buộc | TEXT | 1,255 | Hai Bà Trưng, Hà Nội |
| damageInfo      | Chi tiết hỏng hóc |Bắt buộc| Array[damageInfo] | n           | []          |
| imageDrawUrl      | Ảnh vẽ sẵn vết hỏng |Bắt buộc| Text | 1, 255           | {{s3Link}}          |

*Chi tiết Object item `damageInfo`*

| **Tên Tham số** | **Mô tả**                     | **Bắt buộc** | **Kiểu dữ liệu**    | **Min,Max** | **Ví dụ**       |
|-----------------|-------------------------------|--------------|---------------------|-------------|-----------------|
| vehiclePartName | Tên bộ phận                   | Bắt buộc     | TEXT                | 1,255       | Ba đờ sốc trước |
| price           | Giá bộ phận                   | Bắt buộc     | Number              | 1,9999      | 1000000         |
| location        | Vị trí bộ phận                | Bắt buộc     | TEXT                | 1,255       | Trước           |
| damageDetail    | Chi tiết hỏng hóc của bộ phận | Bắt buộc     | Array[damageDetail] | n           | []              |

*Chi tiết Object item `damageDetail`*

| **Tên Tham số**  | **Mô tả**          | **Bắt buộc** | **Kiểu dữ liệu**    | **Min,Max** | **Ví dụ**   |
|------------------|--------------------|--------------|---------------------|-------------|-------------|
| url              | Link mask hỏng hóc | Bắt buộc     | TEXT                | 1,255       | {{maskUrl}} |
| damageTypeName   | Loại hỏng hóc      | Bắt buộc     | TEXT                | 1,255       | Trầy (xước) |
| damagePercentage | Phần trăm hỏng hóc | Bắt buộc     | Number              | 1,9999      | 0.08        |

#### d. Ví dụ đầu ra

```
{
    "result": {
        "images": [
            {
                "imageId": 16532,
                "imageName": "1679373021917.jpg",
                "filePath": "INSURANCE/1679373021956/1679373021917.jpg",
                "url": "{{imageUrl}}",
                "imageDrawUrl": "{{imageUrl}}",
                "imageSize": [
                    1920,
                    1080
                ],
                "directionName": "45° Trái - Sau",
                "imageRangeName": "Toàn cảnh",
                "timeProcess": 1.693,
                "timeAppUpload": 1.501,
                "requestedTime": "2023-03-24T03:28:29.414Z",
                "uploadedTime": "2023-03-27T03:04:53.001Z",
                "uploadLocation": "Quận Cầu Giấy, Hà Nội",
                "location": "Duy Tân, Cầu Giấy, Hà Nội",
                "errorType": null,
                "errorNote": null,
                "traceId": "265e70b629c05b353b1968693a4ca051",
                "damageInfo": [
                    {
                        "vehiclePartName": "Pa vô lê trái",
                        "vehiclePartExcelId": "pavole-trai-tEm7AB",
                        "price": 450000,
                        "laborCost": 0,
                        "totalCost": 450000,
                        "area": 70,
                        "location": "Trái",
                        "repairPlan": "Sửa chữa",
                        "damageDetail": [
                            {
                                "filePath": "fxVIBETAr_Buc1h6ngzC4.png",
                                "url": "{{maskUrl}}",
                                "damageTypeName": "Trầy (xước)",
                                "damageTypeColor": "#FFEC05",
                                "damageUuid": "yfMzer07THdYoCI1SM2LN",
                                "boxes": [
                                    0.3125,
                                    0.6731481481481482,
                                    0.32135416666666666,
                                    0.6953703703703704
                                ],
                                "damageArea": 0.6255235966733421,
                                "damagePercentage": 0.008936051381047744
                            }
                        ]
                    }
                ]
            }
     ]
   }
 }       
```

### **3.14: API lấy kết quả Hồ sơ (nhóm theo bộ phận) V2**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/v2/claimfolders/{externalSessionId}/external-segment-result |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|externalSessionId|Id hồ sơ (giấy chứng nhận) của khách hàng|Bắt buộc|Text|1,255|folder1|

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/v2/claimfolders/${externalSessionId}/external-segment-result' \
--header 'authorization: Bearer $$API_KEY$$' \
--data-raw ''
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body:
```
{
    ${VehiclePart}
}
```
*Chi tiết Object item `VehiclePart`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|vehiclePartExcelId|Mã bộ phận|Bắt buộc|Text|1,255|ba-do-soc-truoc-WUBZvD|
|vehiclePartName|Tên bộ phận|Bắt buộc|Text|1,255|Ba đờ sốc trước|
|location|Vị trí bộ phận|Bắt buộc|Text|1,255|Trước|
|damages|Chi tiết hỏng hóc|Bắt buộc|Array[damage]|n|[]|
|images|Ảnh|Bắt buộc|Array[image]|n|[]|

*Chi tiết Object item `damage`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|damageTypeName|Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
|damagePercentage|Phần trăm hỏng hóc|Bắt buộc|Number|0,1|0.05|
|damageArea|Diện tích hỏng hóc|Bắt buộc|Number|0,1|0.05|

*Chi tiết Object item `image`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|imageUrl|Url ảnh|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|
|imageRange|Loại ảnh|Bắt buộc|Text|1,255|Toàn cảnh|
|location | Nơi ảnh được chụp | Bắt buộc | TEXT | 1,255 | Duy Tân, Cầu Giấy, Hà Nội |
|requestedTime | Thời gian chụp ảnh | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
| uploadedTime | Thời gian ảnh được upload lên hệ thống | Bắt buộc | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
|uploadLocation | Nơi upload ảnh để thực hiện claim | Bắt buộc | TEXT | 1,255 | Hai Bà Trưng, Hà Nội |
|damageMasks|Chi tiết hỏng hóc của ảnh|Bắt buộc|Array[damageMask]|n|[]|

*Chi tiết Object item `damageMask`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|damageTypeName|Loại hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
|maskUrl|Url mask hỏng hóc|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

#### d. Ví dụ đầu ra

```
{
    [
        {
            "vehiclePartExcelId": "ba-do-soc-truoc-WUBZvD",
            "vehiclePartName": "Ba đờ sốc trước",
            "location": "Trước",
            "createdDate": "2022/10/24",
            "damages": [
                {
                    "damageTypeName": "Trầy (xước)",
                    "damagePercentage": 0.0419343353811561,
                    "damageArea": 0.419343353811561,
                    "damageTypeColor": "#FFEC05"
                }
            ],
            "images": [
                {
                    "imageId": 10579,
                    "filePath": "INSURANCE_CLAIM/1652252930377/image_picker7865199709592835107.jpg",
                    "imageUrl": "https://s3-sgn09.fptcloud.com/aicycle/INSURANCE_CLAIM/1652252930377/image_picker7865199709592835107.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0080fa5f9d06c7ad85c7%2F20221024%2Fsgn09%2Fs3%2Faws4_request&X-Amz-Date=20221024T105823Z&X-Amz-Expires=7200&X-Amz-Signature=1e0ad191019668c5e77af28aa301f00f3d5f22a98882a1024660da2b9d946897&X-Amz-SignedHeaders=host",
                    "imageRange": "Toàn cảnh",
                    "damageExist": true,
                    "errorType": null,
                    "errorNote": null,
                    "location": "Duy Tân, Cầu Giấy, Hà Nội",
                    "requestedTime": "2023-03-24T03:28:29.414Z",
                    "uploadedTime": "2023-03-27T03:04:53.001Z",
                    "uploadLocation": "Quận Cầu Giấy, Hà Nội",
                    "timeProcess": 2.678,
                    "damageMasks": [
                        {
                            "maskPath": "8EskO3gPp1DYO00BOcBPq.png",
                            "maskUrl": "https://s3-sgn09.fptcloud.com/aicycle/INSURANCE_RESULT/8EskO3gPp1DYO00BOcBPq.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=0080fa5f9d06c7ad85c7%2F20221024%2Fsgn09%2Fs3%2Faws4_request&X-Amz-Date=20221024T105823Z&X-Amz-Expires=7200&X-Amz-Signature=d531e05207a16858319bf560331cb684e20e728abd84c516c4d2918d5139dbef&X-Amz-SignedHeaders=host",
                            "vehiclePartName": "Ba đờ sốc trước",
                            "damageTypeUuid": "yfMzer07THdYoCI1SM2LN",
                            "damageTypeName": "Trầy (xước)",
                            "damageTypeColor": "#FFEC05",
                            "boxes": [
                                0,
                                0,
                                1,
                                1
                            ],
                            "isPart": false,
                            "userCreated": false
                        }
                    ]
                }
            ],
            "price": 1000000,
            "laborCost": 0,
            "totalCost": 1000000,
            "area": 10,
            "repairPlan": "Sửa chữa"
        }
    ]
}
```


### **3.15: API verify hồ sơ**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | GET |
| API Url | https://api.aicycle.ai/insurance/insurance/checkCar/${claimId} |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimId|Id hồ sơ do AICycle trả ra|Bắt buộc|Number|1,99999|123|

**Ví dụ**
```
curl --location --request GET 'https://api.aicycle.ai/insurance/insurance/checkCar/${claimId}' \
--header 'authorization: Bearer $$API_KEY$$'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body:
```
{
  "state": <Number>(Mã trạng thái của hồ sơ),
  "message": <String>(Message thông báo trạng thái của hồ sơ)
}
```
> **Các giá trị của `state` và `message`**
>
>| stateId    | message |
>|------------------|---|
>| 0         |trạng thái mặc định (các ảnh thuộc cùng một xe)|
>| 1         |chỉ có 1 ảnh trong folder|
>| 2         |có ít nhất 1 ảnh trong folder chụp không đúng góc|
>| 3         |các ảnh trong folder thuộc cùng một xe|
>| 4         |các ảnh trong folder không thuộc cùng một xe|
>| 5         |không nhận diện được biển số nào trong folder|
>| 6         |thư mục ảnh rỗng|
>| 12            | Hồ sơ có thể có ảnh không đảm bảo chất lượng. Khách hàng đã up ${numOfImageErr}. Vui lòng check lại hồ sơ.  |
>| 13 | Hồ sơ có thể được chụp trong không gian tối, khách hàng đã up ${numOfImageErr} ảnh tối. Vui lòng check lại hồ sơ.  |
>| 14 | Hồ sơ có thể có ảnh mờ, khách hàng đã up ${numOfImageErr} ảnh mờ. Vui lòng check lại hồ sơ.  |
>| 15              | Hồ sơ có thể có ảnh lóa, khách hàng đã up ${numOfImageErr} ảnh lóa. Vui lòng check lại hồ sơ.  |
>| 16   | Hồ sơ có dấu hiệu gian lận, khách hàng đã up ${numOfImageErr} ảnh chụp qua màn hình. Vui lòng check lại hồ sơ.  |

#### d. Ví dụ đầu ra

```
{
    "state": 16,
    "message": "Hồ sơ có dấu hiệu gian lận, khách hàng đã up 2 ảnh chụp qua màn hình. Vui lòng check lại hồ sơ."
}
```

## **4. APIs tích hợp OCR**
### 4.1 API OCR căn cước công dân

##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | POST                                                        |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/identification-card |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                              | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                    |
|-----------------|----------------------------------------------------------------------------------------|----------|------------------|-------------|------------------------------|
| frontFilePath   | Path ảnh mặt trước cccd trên s3 (lấy từ kết quả trả về của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | OCR_IMAGES/front-image-1.jpg |
| rearFilePath    | Path ảnh mặt sau cccd trên s3 (lấy từ kết quả trả về của API get imageUrl ở mục 3.3)   | Bắt buộc | Text             | 1,255       | OCR_IMAGES/rear-image-1.jpg  |


**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/identification-card' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data '{
    "frontFilePath": "OCR_IMAGES/front-img-1.jpg",
    "rearFilePath": "OCR_IMAGES/rear-img-2.jpg"
}'

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "frontImgUrl": <Url ảnh mặt trước cccd (String)>,
    "rearImgUrl": <Url ảnh mặt sau cccd (String)>,
    "cardNumber": <Mã số định danh (String)>,
    "fullName": <Họ và tên (String)>,
    "dob": <Ngày, tháng, năm sinh (String)>,
    "gender": <Giới tính (String)>,
    "nationality": <Quốc tịch (String)>,
    "homeTown": <Quê quán (String)>,
    "duration":  <Ngày hết hạn (String)>,
    "placeOfResidence": <Nơi cấp (String)>,
    "identifyingCharacteristics": <Đặc điểm nhận dạng (String)>,
    "dateIssuance": <Ngày cấp (String)>
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "frontImgUrl": "{{s3Url}}",
    "rearImgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "cardNumber": "0123456789012",
    "fullName": "Nguyễn Văn A",
    "dob": "01/02/2003",
    "gender": "Nam",
    "nationality": "Việt Nam",
    "homeTown": "Làng Nhì, Trạm Tàu, Yên Bái",
    "duration": "01/02/2003",
    "placeOfResidence": "Thôn Láng Nhĩ Làng Nhì Tram Tầu, Yên Bái",
    "identifyingCharacteristics": "Sẹo chấm C:1cm trên sau đầu lông mày trái",
    "dateIssuance": "01/02/2003"
}
```


### 4.2 API OCR đăng kiểm xe

##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | POST                                                        |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/vehicle-inspection |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|-----------------------------------------------------------------------------------|----------|------------------|-------------|------------------------|
| filePath        | Path ảnh đăng kiểm trên s3 (lấy từ kết quả trả về của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | OCR_IMAGES/image-2.jpg |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/vehicle-inspection' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data '{
    "filePath": "OCR_IMAGES/img-2.jpg",
}'

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "imgUrl": <Url ảnh đăng kiểm (String)>,
    "registrationNumber": <Biển đăng ký (String)>,
    "inspectionNumber": <Số quản lý  (String)>,
    "typeValue": <Loại xe  (String)>,
    "markValue": <Nhãn hiệu (String)>,
    "modelCode": <Số loại (String)>,
    "engineNumber": <Số máy (String)>,
    "chassisNumber":  <Số khung (String)>,
    "manufacturedYearCountry": <Năm, nơi sản xuất (String)>,
    "wheelFormula": <Công thức bánh xe (String)>,
    "typeOfFuel": <Loại nhiên liệu (String)>,
    "engineDisplacement": <Dung tích động cơ (String)>
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "imgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "registrationNumber": "01A-012.34",
    "inspectionNumber": "01A-012.34",
    "typeValue": "ô tô con",
    "modelCode": "VIOS123456",
    "engineNumber": "1AB-CDEF",
    "chassisNumber": "ABCD123456",
    "manufacturedYearCountry": "2012,Việt Nam",
    "wheelFormula": "4x2",
    "typeOfFuel": "Xăng",
    "engineDisplacement": "1000 (cm3)"
}
```


### 4.3 API OCR giấy phép lái xe

##### a. Thông tin cơ bản

||                                                         |
|----|---------------------------------------------------------|
| Method | POST                                                    |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/driving-license |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`             |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|------------------------------------------------------------------------------------------|----------|------------------|-------------|------------------------|
| filePath        | Path ảnh giấy phép lái xe trên s3 (lấy từ kết quả trả về của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | OCR_IMAGES/image-3.jpg |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/driving-license' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data '{
    "filePath": "OCR_IMAGES/img-3.jpg",
}'

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "imgUrl": <Url ảnh đăng kiểm (String)>,
    "cardNumber": <Số thẻ (String)>,
    "fullName": <Họ và tên  (String)>,
    "dob": <Ngày, tháng , năm sinh  (String)>,
    "nationality": <Quốc tịch (String)>,
    "placeOfResidence": <Nơi cư trú (String)>,
    "placeIssuance": <Nơi cấp (String)>,
    "dateIssuance":  <Ngày cấp (String)>,
    "rank": <Hạng (String)>,
    "duration": <Ngày hết hạn (String)>,
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "imgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "cardNumber": "0123456789",
    "fullName": "Nguyễn Văn A",
    "dob": "01/02/2003",
    "nationality": "VIỆT NAM",
    "placeOfResidence": "Số 1, đường Nguyễn Văn A, Q.Đống Đa, Hà Nội",
    "placeIssuance": "Hà Nội",
    "dateIssuance": "01/01/2000",
    "rank": "C",
    "duration": "01/01/2000"
}
```

### 4.4 API OCR đăng ký xe

##### a. Thông tin cơ bản

||                                                              |
|----|--------------------------------------------------------------|
| Method | POST                                                         |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/vehicle-registration |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                  |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                          | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|------------------------------------------------------------------------------------|----------|------------------|-------------|------------------------|
| filePath        | Path ảnh đăng ký xe trên s3 (lấy từ kết quả trả về của API get imageUrl ở mục 3.3) | Bắt buộc | Text             | 1,255       | OCR_IMAGES/image-3.jpg |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/vehicle-registration' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data '{
    "filePath": "OCR_IMAGES/img-3.jpg",
}'

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "imgUrl": <Url ảnh đăng ký xe (String)>,
    "name": <Họ và tên (String)>,
    "address": <Địa chỉ  (String)>,
    "modelCode": <Số loại (String)>,
    "brand": <Hãng xe (String)>,
    "engine": <Động cơ (String)>,
    "chassis": <Số khung (String)>,
    "color":  <Màu xe (String)>,
    "plate": <Biển số (String)>,
    "place": <Nơi cấp (String)>,
    "date": <Ngày cấp (String)>
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "imgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "name": "Nguyễn Văn A",
    "address": "TDP Dộc, Tây Mỗ, Nam Từ Liêm, HIN",
    "modelCode": "VIOS",
    "brand": "TOYOTA",
    "engine": "ABC123456",
    "chassis": "12345678",
    "color": "Đen",
    "plate": "30E-111.11",
    "place": "Hà Nội",
    "date": "01/01/2011"
}
```

### 4.5 API OCR căn cước công dân (form-data)

##### a. Thông tin cơ bản

||                                                                    |
|----|--------------------------------------------------------------------|
| Method | POST                                                               |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/identification-card/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                        |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Form Data

| **Tên Tham số** | **Mô tả**                                                                       | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                    |
|-----------------|---------------------------------------------------------------------------------|----------|------------------|-------------|------------------------------|
| frontImg   | Ảnh mặt trước cccd                                                              | Bắt buộc | File             | n           |  |
| rearImg    | Ảnh mặt sau cccd  | Bắt buộc | File             | n           |   |


**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/identification-card/upload' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--form 'frontImg=@"$imgFilePathInLocalMachine"' \
--form 'rearImg=@"$imgFilePathInLocalMachine"'

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "frontImgUrl": <Url ảnh mặt trước cccd (String)>,
    "rearImgUrl": <Url ảnh mặt sau cccd (String)>,
    "cardNumber": <Mã số định danh (String)>,
    "fullName": <Họ và tên (String)>,
    "dob": <Ngày, tháng, năm sinh (String)>,
    "gender": <Giới tính (String)>,
    "nationality": <Quốc tịch (String)>,
    "homeTown": <Quê quán (String)>,
    "duration":  <Ngày hết hạn (String)>,
    "placeOfResidence": <Nơi cấp (String)>,
    "identifyingCharacteristics": <Đặc điểm nhận dạng (String)>,
    "dateIssuance": <Ngày cấp (String)>
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "frontImgUrl": "{{s3Url}}",
    "rearImgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "cardNumber": "0123456789012",
    "fullName": "Nguyễn Văn A",
    "dob": "01/02/2003",
    "gender": "Nam",
    "nationality": "Việt Nam",
    "homeTown": "Làng Nhì, Trạm Tàu, Yên Bái",
    "duration": "01/02/2003",
    "placeOfResidence": "Thôn Láng Nhĩ Làng Nhì Tram Tầu, Yên Bái",
    "identifyingCharacteristics": "Sẹo chấm C:1cm trên sau đầu lông mày trái",
    "dateIssuance": "01/02/2003"
}
```


### 4.6 API OCR đăng kiểm xe (form-data)

##### a. Thông tin cơ bản

||                                                                   |
|----|-------------------------------------------------------------------|
| Method | POST                                                              |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/vehicle-inspection/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                       |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Form Data

| **Tên Tham số** | **Mô tả**                                                                  | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------------------------------------------------------------------|----------|------------------|-------------|------------------------|
| img             | Ảnh đăng kiểm  | Bắt buộc | File             | n           | |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/vehicle-inspection/upload' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--form 'img=@"$imgFilePathInLocalMachine"'

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "imgUrl": <Url ảnh đăng kiểm (String)>,
    "registrationNumber": <Biển đăng ký (String)>,
    "inspectionNumber": <Số quản lý  (String)>,
    "typeValue": <Loại xe  (String)>,
    "markValue": <Nhãn hiệu (String)>,
    "modelCode": <Số loại (String)>,
    "engineNumber": <Số máy (String)>,
    "chassisNumber":  <Số khung (String)>,
    "manufacturedYearCountry": <Năm, nơi sản xuất (String)>,
    "wheelFormula": <Công thức bánh xe (String)>,
    "typeOfFuel": <Loại nhiên liệu (String)>,
    "engineDisplacement": <Dung tích động cơ (String)>
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "imgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "registrationNumber": "01A-012.34",
    "inspectionNumber": "01A-012.34",
    "typeValue": "ô tô con",
    "modelCode": "VIOS123456",
    "engineNumber": "1AB-CDEF",
    "chassisNumber": "ABCD123456",
    "manufacturedYearCountry": "2012,Việt Nam",
    "wheelFormula": "4x2",
    "typeOfFuel": "Xăng",
    "engineDisplacement": "1000 (cm3)"
}
```


### 4.7 API OCR giấy phép lái xe (form-data)

##### a. Thông tin cơ bản

||                                                                |
|----|----------------------------------------------------------------|
| Method | POST                                                           |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/driving-license/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                    |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Form Data

| **Tên Tham số** | **Mô tả**                                                                      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|--------------------------------------------------------------------------------|----------|------------------|-------------|------------------------|
| img             | Ảnh giấy phép lái xe  | Bắt buộc | File             | n           |  |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/driving-license/upload' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--form 'img=@"$imgFilePathInLocalMachine"''

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "imgUrl": <Url ảnh đăng kiểm (String)>,
    "cardNumber": <Số thẻ (String)>,
    "fullName": <Họ và tên  (String)>,
    "dob": <Ngày, tháng , năm sinh  (String)>,
    "nationality": <Quốc tịch (String)>,
    "placeOfResidence": <Nơi cư trú (String)>,
    "placeIssuance": <Nơi cấp (String)>,
    "dateIssuance":  <Ngày cấp (String)>,
    "rank": <Hạng (String)>,
    "duration": <Ngày hết hạn (String)>,
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "imgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "cardNumber": "0123456789",
    "fullName": "Nguyễn Văn A",
    "dob": "01/02/2003",
    "nationality": "VIỆT NAM",
    "placeOfResidence": "Số 1, đường Nguyễn Văn A, Q.Đống Đa, Hà Nội",
    "placeIssuance": "Hà Nội",
    "dateIssuance": "01/01/2000",
    "rank": "C",
    "duration": "01/01/2000"
}
```

### 4.8 API OCR đăng ký xe (form-data)

##### a. Thông tin cơ bản

||                                                                     |
|----|---------------------------------------------------------------------|
| Method | POST                                                                |
| API Url | https://api.aicycle.ai/insurance/v2/ocr/vehicle-registration/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                         |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Form Data

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| img             | Ảnh đăng ký xe | Bắt buộc | File             | n           |  |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/v2/ocr/vehicle-registration/upload' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--form 'img=@"$imgFilePathInLocalMachine"''

```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "status": <Trạng thái của Request (String)>,
    "imgUrl": <Url ảnh đăng ký xe (String)>,
    "name": <Họ và tên (String)>,
    "address": <Địa chỉ  (String)>,
    "modelCode": <Số loại (String)>,
    "brand": <Hãng xe (String)>,
    "engine": <Động cơ (String)>,
    "chassis": <Số khung (String)>,
    "color":  <Màu xe (String)>,
    "plate": <Biển số (String)>,
    "place": <Nơi cấp (String)>,
    "date": <Ngày cấp (String)>
}
```

#### d. Ví dụ đầu ra
```
{
    "status": "success",
    "imgUrl": "{{s3Url}}",
    "problem": "",
    "isValidDocument": true,
    "name": "Nguyễn Văn A",
    "address": "TDP Dộc, Tây Mỗ, Nam Từ Liêm, HIN",
    "modelCode": "VIOS",
    "brand": "TOYOTA",
    "engine": "ABC123456",
    "chassis": "12345678",
    "color": "Đen",
    "plate": "30E-111.11",
    "place": "Hà Nội",
    "date": "01/01/2011"
}
```

### 4.9 Các API cho định giá xe tải
##### 4.9.1 API lấy list hãng xe
###### a. Thông tin cơ bản

||                                                                     |
|----|---------------------------------------------------------------------|
| Method | GET                                                                |
| API Url | https://api.aicycle.ai/insurance/vehicles/${vehicleTypeKey}/company |
| API Headers | `{ "Authorization": "Bearer ${API_KEY}" }`                         |

> **Các giá trị của `vehicleTypeKey`**
>
>| vehicleTypeName    | vehicleTypeKey |
>|------------------|---|
>| Xe chở hàng hóa            | truck  |
>| Xe chuyên dụng | specialized  |
>| Xe khách | coach  |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Query String

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| vehicleTypeKey             | Key loại xe | Optional | Text             | 1,255           | truck |
| limit             | Giới hạn số bản ghi trả ra | Optional | Number             | 1,99999           | 30 |
| offset             | Bỏ qua số bản ghi | Optional | Number             | 1,99999           | 30 |
| companyName             | Tìm kiếm tên hãng xe | Optional | Text             | 1,255           | CHENGLONG |
| orderBy             | Sort hãng xe | Optional | Text             | 1,255           | ?orderBy=companyName DESC |


**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/vehicles/${vehicleTypeKey}/company' \
--header 'Authorization: Bearer ${API_KEY}' \
--header 'Content-Type: application/json'

```


##### d. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "count": <Số lượng data trả ra>,
    "limit": <Giới hạn data trả ra>,
    "offset": <Số lượng bản ghi bỏ qua>,
    "records": <Array(truckCompanyObj)>
}
```

*Chi tiết Object item `truckCompanyObj`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| companyKey             | Key hãng xe        |Bắt buộc| Text    | 1,255           | chenglong |
| companyName              | Tên hãng xe         |Bắt buộc| Text             | 1,255       | CHENGLONG |


##### 4.9.2 API lấy list hiệu xe, phiên bản
###### a. Thông tin cơ bản

||                                                                     |
|----|---------------------------------------------------------------------|
| Method | GET                                                                |
| API Url | https://api.aicycle.ai/insurance/vehicles/${vehicleTypeKey}/versions |
| API Headers | `{ "Authorization": "Bearer ${API_KEY}" }`                         |

> **Các giá trị của `vehicleTypeKey`**
>
>| vehicleTypeName    | vehicleTypeKey |
>|------------------|---|
>| Xe chở hàng hóa            | truck  |
>| Xe chuyên dụng | specialized  |
>| Xe khách | coach  |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Query String

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| vehicleTypeKey             | Key loại xe | Optional | Text             | 1,255           | truck |
| limit             | Giới hạn số bản ghi trả ra | Optional | Number             | 1,99999           | 30 |
| offset             | Bỏ qua số bản ghi | Optional | Number             | 1,99999           | 30 |
| companyName             | Flag dùng để filter hiệu xe theo hãng xe | Optional | Text             | 1,255           | CHENGLONG |
| versionName             | Flag dùng để tìm kiếm tên hiệu xe, phiên bản | Optional | Text             | 1,255          | 340HP 10X4 |
| orderBy             | Sort hiệu xe, phiên bản | Optional | Text             | 1,255           | ?orderBy=versionName DESC |


**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/vehicles/${vehicleTypeKey}/versions' \
--header 'Authorization: Bearer ${API_KEY}' \
--header 'Content-Type: application/json'

```


##### d. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "count": <Số lượng data trả ra>,
    "limit": <Giới hạn data trả ra>,
    "offset": <Số lượng bản ghi bỏ qua>,
    "records": <Array(truckVersionObj)>
}

```

*Chi tiết Object item `truckVersionObj`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| versionKey             | Key hiệu xe        |Bắt buộc| Text    | 1,255           | 180hp-4x2-thung-vuong |
| versionName              | Tên hiệu xe         |Bắt buộc| Text             | 1,255       | 180HP 4X2 THÙNG VUÔNG |


##### 4.9.3 API lấy các thông tin số loại, mã thùng,... từ hãng, hiệu xe
###### a. Thông tin cơ bản

||                                                                     |
|----|---------------------------------------------------------------------|
| Method | GET                                                                |
| API Url | https://api.aicycle.ai/insurance/vehicles/${vehicleTypeKey}/codebooks |
| API Headers | `{ "Authorization": "Bearer ${API_KEY}" }`                         |

> **Các giá trị của `vehicleTypeKey`**
>
>| vehicleTypeName    | vehicleTypeKey |
>|------------------|---|
>| Xe chở hàng hóa            | truck  |
>| Xe chuyên dụng | specialized  |
>| Xe khách | coach  |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Query String

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| vehicleTypeKey             | Key loại xe | Optional | Text             | 1,255           | truck |
| limit             | Giới hạn số bản ghi trả ra | Optional | Number             | 1,99999           | 30 |
| offset             | Bỏ qua số bản ghi | Optional | Number             | 1,99999           | 30 |
| companyKey             | Flag dùng để filter số loại,... theo hãng xe | Optional | Text             | 1,255           | chenglong |
| versionKey             | Flag dùng để filter số loại,... theo hiệu xe, phiên bản | Optional | Text             | 1,255          | 340hp-10x4 |
| orderBy             | Sort hiệu xe, phiên bản | Optional | Text             | 1,255           | ?orderBy=modelCode DESC |


**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/vehicles/${vehicleTypeKey}/codebooks' \
--header 'Authorization: Bearer ${API_KEY}' \
--header 'Content-Type: application/json'

```


##### d. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "count": <Số lượng data trả ra>,
    "limit": <Giới hạn data trả ra>,
    "offset": <Số lượng bản ghi bỏ qua>,
    "records": <Array(truckModelCodeObj)>
}

```

*Chi tiết Object item `truckModelCodeObj`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| barrelTypeKey             | Key loại thùng        |Bắt buộc| Text    | 1,255           | chassis |
| barrelTypeName              | Tên loại thùng         |Bắt buộc| Text             | 1,255       | Chassis |
| carTypeKey             | Key loại xe        |Bắt buộc| Text    | 1,255           | xe-tai |
| carTypeName              | Tên loại xe         |Bắt buộc| Text             | 1,255       | Xe tải |
| barrelTypeKey             | Key loại thùng        |Bắt buộc| Text    | 1,255           | chassis |
| modelCode             | Số loại         |Bắt buộc| Text             | 1,255       | LZ1340PELT |
| payload              | Tải trọng         |Bắt buộc| Number             | 1,99999       | 22450 |

##### 4.9.4 API lấy thông tin thùng xe
###### a. Thông tin cơ bản

||                                                                     |
|----|---------------------------------------------------------------------|
| Method | GET                                                                |
| API Url | https://api.aicycle.ai/insurance/vehicles/barrel |
| API Headers | `{ "Authorization": "Bearer ${API_KEY}" }`                         |


##### b. Chi tiết đầu vào
**Loại đầu vào**: Query String

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| isAll             | Flag lấy loại thùng, true là lấy tất cả các thùng, false là chỉ lấy ra loại thùng có giá  | Optional | Boolean             | n           | true |
| limit             | Giới hạn số bản ghi trả ra | Optional | Number             | 1,99999           | 30 |
| offset             | Bỏ qua số bản ghi | Optional | Number             | 1,99999           | 30 |



**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/vehicles/barrel' \
--header 'Authorization: Bearer ${API_KEY}' \
--header 'Content-Type: application/json'

```


##### d. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "count": <Số lượng data trả ra>,
    "limit": <Giới hạn data trả ra>,
    "offset": <Số lượng bản ghi bỏ qua>,
    "records": <Array(truckBarrelObj)>
}

```

*Chi tiết Object item `truckBarrelObj`*

| **Tên Tham số**     | **Mô tả**            |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------------------|----------------------|---|------------------|-------------|---------|
| key             | Key thùng         |Bắt buộc| Text    | 1,255           | thung-lung-dai-2m2 |
| name              | Tên thùng         |Bắt buộc| Text             | 1,255       | Thùng Lửng dài 2m2 |
| price             | Giá thùng        |Bắt buộc| Number    | 1,99999           | 40000000 |
| size              | Size thùng         |Bắt buộc| Text             | 1,255       | dài 2m2 |


##### 4.9.5 API định giá xe tải
###### a. Thông tin cơ bản

||                                                                     |
|----|---------------------------------------------------------------------|
| Method | POST                                                                |
| API Url | https://api.aicycle.ai/insurance/valuation/v3/truck-valuate |
| API Headers | `{ "Authorization": "Bearer ${API_KEY}" }`                         |


##### b. Chi tiết đầu vào
**Loại đầu vào**: Request Body

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| barrelTypeKey             | Key thùng (lấy từ API 4.9.4)  | Bắt buộc | Text             | 1,255           | thung-lung-dai-5m1 |
| companyName             | Key hãng xe (lấy từ API 4.9.1) | Bắt buộc | Text             | 1,255           | chenglong |
| manufactureYear             | Năm sản xuất (hỗ trợ từ năm 2000 đến hiện tại) | Bắt buộc | Text             | 1,255           | 2024 |
| modelCode             | Số loại (lấy từ API 4.9.3) | Bắt buộc | Text             | 1,255           | LZ1340PELT |
| vehicleTypeKey             | Loại xe (truch, coach, specialized) | Bắt buộc | Text             | 1,255           | truck |
| versionName             | Key hiệu xe, phiên bản (lấy từ API 4.9.2) | Bắt buộc | Text             | 1,255           | 340hp-10x4 |




**Ví dụ**

```
curl --location 'https://api.aicycle.ai/insurance/vehicles/barrel' \
--header 'Authorization: Bearer ${API_KEY}' \
--header 'Content-Type: application/json' \
--data '{
  "vehicleTypeKey":"truck",
  "companyName":"chenglong",
  "versionName":"340hp-10x4",
  "modelCode":"LZ1340PELT",
  "manufactureYear":"2024",
  "barrelTypeKey":"thung-lung-dai-5m1"
}'

```


##### d. Chi tiết đầu ra
**Loại đầu ra**: Response body
```
{
    "barrelTypeKey": <Mã thùng>,
    "companyName": <Mã hãng>,
    "manufactureYear": <năm sản xuất>,
    "maxPrice": <Max Price>,
    "minPrice": <Min Price>,
    "modelCode": <Số loại>,
    "price": <Giá trị xe>,
    "vehicleTypeKey": <Mã loại xe>,
    "versionName": <Mã hiệu xe, phiên bản>
}

```
`
## **5. APIs tích hợp partMe (phụ tùng)**
### 5.1 Đồng bộ mã, giá phụ tùng (gửi data sang khách hàng lưu trữ)
#### a. Thông tin cơ bản
```
Đây là flow đồng bộ giữa AICycle và khách hàng theo cách khách hàng sẽ mở các đầu API để AICycle gửi data sang, 
khách hàng sẽ lưu trữ data và tự định giá phụ tùng dựa trên dữ liệu của AICycle.
```

![Sync data picture](https://dyta7vmv7sqle.cloudfront.net/INSURANCE_DOCUMENT/sync-data-partme.png)

#### b. Chi tiết schema của AICycle gửi sang khách hàng
**Loại đầu vào**: JSON Body

| **Tên Tham số** | **Mô tả**      | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**              |
|-----------------|----------------|----------|------------------|-------------|------------------------|
| carCompany             | Hãng xe  | Bắt buộc | Text             | 1, 255           | TOYOTA |
| carModel             | Hiệu xe | Bắt buộc | Text             | 1,255           | VIOS |
| carVersion             | Phiên bản | Bắt buộc | Text             | 1,255           | 1.5E AT |
| manufactureYear             | Năm sản xuất | Bắt buộc | Number             | 1,99999           | 2025 |
| originName             | Xuất xứ | Bắt buộc | Text             | 1,255           | Nhập khẩu |
| standardPartName             | Tên phụ tùng chuẩn của AICycle | Bắt buộc | Text             | 1,255           | Cụm đèn pha trái |
| equivalentPartNames             | Tên phụ tùng hiện tại của khách hàng | Bắt buộc | Array[text]             | n           | [Cụm đèn pha L, Cụm đèn pha lái,...] |
| partPrice             | Giá phụ tùng | Bắt buộc | Number             | 1,99999           | 4257000 |
| garaName             | Tên gara | Bắt buộc | Text             | 1, 255           | CÔNG TY TNHH TOYOTA HƯNG THỊNH PHÁT THÁI BÌNH |
| approvalDate             | Ngày phê duyệt | Bắt buộc | Text             | 1, 255           | 2024-05-30 10:35:00 |
| province             | Tỉnh thành | Bắt buộc | Text             | 1, 255           | Tỉnh Thái Bình |
| region             | Khu vực | Bắt buộc | Text             | 1, 255           | Miền Bắc |

### **5.2: API định giá phụ tùng (sử dụng CodeBook hãng hiệu và phiên bản của AICycle) (deprecated)**
#### **5.2.1: API lấy thông tin xuất xứ**
##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | GET                                                         |
| API Url | https://api.aicycle.ai/insurance/part-me/origins |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-----------|-----------|
| id              | Id xuất xứ        | Optional     | Number             | 1,9999     | 1         |
| key             | Key xuất xứ       | Optional     | Text             | 1,255     | lrtn      |
| name            | Tên xuất xứ       | Optional     | Text             | 1,255     | LRTN      |
| limit           | Giới hạn số lượng | Optional     | Number             | 1,9999     | 30        |
| offset          | Index start       | Optional     | Number           | 1,9999    | 0         |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/part-me/origins' \
--header 'Authorization: Bearer <API Key>' \
```
##### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-------------|-----------|
| count    | Số lượng record   | Bắt buộc     | Number           | 1,9999       | 2         |
| limit      | Giới hạn số lượng | Bắt buộc     | Number             | 1,9999       | 30        |
| offset    | Index start       | Bắt buộc     | Number             | 1,9999       | 0         |
| records            | Danh sách xuất xứ | Bắt buộc     | Array[origin]    | n           | []        |

*Chi tiết Object item  `origin`*

| **Tên Tham số**  | **Mô tả**   |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|------------------|-------------|--------------|------------------|-------------|-----------|
| id               | Id xuất xứ  | Bắt buộc     | Number           | 1,9999       | 1         |
| key              | Key xuất xứ | Bắt buộc     | Text             | 1,255       | lrtn      |
| name             | Tên xuất xứ | Bắt buộc     | Text             | 1,255       | LRTN      |


##### d. Ví dụ đầu ra
```
{
    "count": 2,
    "limit": 30,
    "offset": 0,
    "records": [
        {
            "id": 2,
            "key": "lrtn",
            "name": "LRTN"
        },
        {
            "id": 1,
            "key": "nk",
            "name": "NK"
        }
    ]
}
```
#### **5.2.2: API lấy thông tin khu vực**
##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | GET                                                         |
| API Url | https://api.aicycle.ai/insurance/part-me/regions |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-----------|---------|
| id              | Id khu vực        | Optional     | Text             | 1,255     | uuid-region        |
| slug            | Key khu vực       | Optional     | Text             | 1,255     | mienbac |
| name            | Tên khu vực       | Optional     | Text             | 1,255     | Miền Bắc |
| provinceId      | Id tỉnh           | Optional     | Text             | 1,255     | uuid-tinh |
| limit           | Giới hạn số lượng | Optional     | Number             | 1,9999     | 30      |
| offset          | Index start       | Optional     | Number           | 1,9999    | 0       |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/part-me/regions' \
--header 'Authorization: Bearer <API Key>' \
```
##### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|-----------------|-------------|-----------|
| count    | Số lượng record   | Bắt buộc     | Number          | 1,9999       | 1         |
| limit      | Giới hạn số lượng | Bắt buộc     | Number          | 1,9999       | 30        |
| offset    | Index start       | Bắt buộc     | Number          | 1,9999       | 0         |
| records            | Danh sách khu vực | Bắt buộc     | Array[region]   | n           | []        |

*Chi tiết Object item  `region`*

| **Tên Tham số** | **Mô tả**   |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**   |
|-----------------|-------------|--------------|------------------|-------------|-------------|
| id              | Id khu vực  | Bắt buộc     | Text           | 1,255       | uuid-region |
| slug            | Key khu vực | Bắt buộc     | Text             | 1,255       | miennam        |
| name            | Tên khu vực | Bắt buộc     | Text             | 1,255       | Miền Nam        |


##### d. Ví dụ đầu ra
```
{
    "count": 1,
    "limit": 30,
    "offset": 0,
    "records": [
        {
            "id": "7ba25bde-a2f0-4772-a0ac-9c771894e9a2",
            "name": "Miền Nam",
            "slug": "miennam"
        }
    ]
}
```
#### **5.2.3: API lấy thông tin tỉnh**
##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | GET                                                         |
| API Url | https://api.aicycle.ai/insurance/part-me/provinces |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**     |
|-----------------|-------------------|--------------|------------------|-----------|---------------|
| id              | Id tỉnh           | Optional     | Text             | 1,255     | uuid-province |
| slug            | Key tỉnh          | Optional     | Text             | 1,255     | mienbac       |
| name            | Tên tỉnh          | Optional     | Text             | 1,255     | Miền Bắc      |
| regionId        | Id khu vực        | Optional     | Text             | 1,255     | uuid-region   |
| limit           | Giới hạn số lượng | Optional     | Number             | 1,9999     | 30            |
| offset          | Index start       | Optional     | Number           | 1,9999    | 0             |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/part-me/provinces' \
--header 'Authorization: Bearer <API Key>' \
```
##### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-------------|-----------|
| count    | Số lượng record   | Bắt buộc     | Number           | 1,9999       | 1         |
| limit      | Giới hạn số lượng | Bắt buộc     | Number           | 1,9999       | 30        |
| offset    | Index start       | Bắt buộc     | Number           | 1,9999       | 0         |
| records            | Danh sách khu vực | Bắt buộc     | Array[province]  | n           | []        |

*Chi tiết Object item  `province`*

| **Tên Tham số** | **Mô tả** |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**     |
|-----------------|-----------|--------------|------------------|-------------|---------------|
| id              | Id tỉnh   | Bắt buộc     | Text           | 1,255       | uuid-province |
| slug            | Key tỉnh  | Bắt buộc     | Text             | 1,255       | tinhtayninh   |
| name            | Tên tỉnh  | Bắt buộc     | Text             | 1,255       | Tỉnh Tây Ninh |


##### d. Ví dụ đầu ra
```
{
    "count": 1,
    "limit": 30,
    "offset": 0,
    "records": [
        {
            "id": "d67d1988-57da-41d1-8b7d-b4186a1bd3b8",
            "name": "Tỉnh Tây Ninh",
            "slug": "tinhtayninh"
        }
    ]
}
```
#### **5.2.4: API lấy thông tin nhóm phụ tùng**
##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | GET                                                         |
| API Url | https://api.aicycle.ai/insurance/part-me/groups |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**  |
|-----------------|-------------------|--------------|------------------|-----------|------------|
| id              | Id nhóm phụ tùng  | Optional     | Text             | 1,255     | uuid-group |
| slug            | Key nhóm phụ tùng | Optional     | Text             | 1,255     | mienbac    |
| name            | Tên nhóm phụ tùng | Optional     | Text             | 1,255     | Miền Bắc   |
| limit           | Giới hạn số lượng | Optional     | Number             | 1,9999     | 30         |
| offset          | Index start       | Optional     | Number           | 1,9999    | 0          |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/part-me/groups' \
--header 'Authorization: Bearer <API Key>' \
```
##### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-------------|-----------|
| count    | Số lượng record   | Bắt buộc     | Number           | 1,9999       | 1         |
| limit      | Giới hạn số lượng | Bắt buộc     | Number           | 1,9999       | 30        |
| offset    | Index start       | Bắt buộc     | Number           | 1,9999       | 0         |
| records            | Danh sách khu vực | Bắt buộc     | Array[group]     | n           | []        |

*Chi tiết Object item  `group`*

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**  |
|-----------------|-------------------|--------------|------------------|-------------|------------|
| id              | Id nhóm phụ tùng  | Bắt buộc     | Text           | 1,255       | uuid-group |
| slug            | Key nhóm phụ tùng | Bắt buộc     | Text             | 1,255       | hethongkhunggamtruyenluc    |
| name            | Tên nhóm phụ tùng | Bắt buộc     | Text             | 1,255       | Hệ thống khung gầm, truyền lực   |


##### d. Ví dụ đầu ra
```
{
    "count": 1,
    "limit": 30,
    "offset": 0,
    "records": [
        {
            "id": "65edef9d-abfc-4264-9369-0e252e6cafdd",
            "name": "Hệ thống khung gầm, truyền lực",
            "slug": "hethongkhunggamtruyenluc"
        }
    ]
}
```
#### **5.2.5: API lấy thông tin codebook phụ tùng**
##### a. Thông tin cơ bản

||                                                             |
|----|-------------------------------------------------------------|
| Method | GET                                                         |
| API Url | https://api.aicycle.ai/insurance/part-me/codebooks |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`                 |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**         | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**     |
|-----------------|-------------------|--------------|------------------|-----------|---------------|
| id              | Id tỉnh           | Optional     | Text             | 1,255     | uuid-codebook |
| slug            | Key tỉnh          | Optional     | Text             | 1,255     | mienbac       |
| name            | Tên tỉnh          | Optional     | Text             | 1,255     | Miền Bắc      |
| partCodebookGroupId        | Id nhóm phụ tùng  | Optional     | Text             | 1,255     | uuid-group    |
| limit           | Giới hạn số lượng | Optional     | Number             | 1,9999     | 30            |
| offset          | Index start       | Optional     | Number           | 1,9999    | 0             |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/part-me/codebooks' \
--header 'Authorization: Bearer <API Key>' \
```
##### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|-----------------|-------------------|--------------|------------------|-------------|-----------|
| count    | Số lượng record   | Bắt buộc     | Number           | 1,9999       | 1         |
| limit      | Giới hạn số lượng | Bắt buộc     | Number           | 1,9999       | 30        |
| offset    | Index start       | Bắt buộc     | Number           | 1,9999       | 0         |
| records            | Danh sách khu vực | Bắt buộc     | Array[codebook]  | n           | []        |

*Chi tiết Object item  `codebook`*

| **Tên Tham số**       | **Mô tả**         |**Bắt buộc**| **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**     |
|-----------------------|-------------------|--------------|------------------|-------------|---------------|
| id                    | Id codebook       | Bắt buộc     | Text           | 1,255       | uuid-codebook |
| partCodebookSlug      | Key codebook      | Bắt buộc     | Text             | 1,255       | ketgiannongdieuhoa   |
| partCodebookName      | Tên codebook      | Bắt buộc     | Text             | 1,255       | Két giàn nóng điều hòa |
| partCodebookGroupName | Tên nhóm codebook | Bắt buộc     | Text             | 1,255       | Hệ thống khung gầm, truyền lực |


##### d. Ví dụ đầu ra
```
{
    "count": 1,
    "limit": 30,
    "offset": 0,
    "records": [
        {
            "id": "222b2d19-5eab-4728-ae22-3e6a4b9cd395",
            "partCodebookName": "Két giàn nóng điều hòa",
            "partCodebookSlug": "ketgiannongdieuhoa",
            "partCodebookGroupName": "Hệ thống khung gầm, truyền lực"
        }
    ]
}
```
#### **5.2.6: API lấy giá phụ tùng**
##### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/valuation/car-part-valuate |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

##### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**              | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**           |
|-----------------|------------------------|--------------|------------------|-----------|---------------------|
| carCompanyId    | Id hãng xe             | Bắt buộc     | Text             | 1,255     | uuid-hang-xe        |
| carModelId      | Id hiệu xe             | Bắt buộc     | Text             | 1,255     | uuid-hieu-xe        |
| carVersionId    | Id phiên bản xe        | Bắt buộc     | Text             | 1,255     | uuid-phien-ban-xe   |
| partCodebookId    | Id codebook phụ tùng   | Bắt buộc     | Text             | 1,255     | uuid-codebook-phu-tung |
| year            | Năm sản xuất           | Bắt buộc     | Number           | 1,9999    | 2020                |
| regionId    | Id khu vực             | Optional     | Text             | 1,255     | uuid-khu-vuc        |
| provinceId    | Id tỉnh trong khu vực  | Optional     | Text             | 1,255     | uuid-tinh           |
| originId    | Id xuất xứ             | Optional     | Number           | 1,9999     | 1                   |
| isAuthentic          | Có là chính hãng không | Optional     | Boolean             | n     | true                    |

**Ví dụ**
```
curl --location 'https://api.aicycle.ai/insurance/valuation/car-part-valuate' \
--header 'Authorization: Bearer <API Key>' \
--header 'Content-Type: application/json' \
--data '{
    "carCompanyId": "ca3aee80-f81c-41fb-9b74-ce6f9fdb996e",
    "carModelId": "7a15f4c0-8b57-4e1a-ae06-587b75ffef39",
    "carVersionId": "7d801069-87b0-4d35-b38c-314c69257b8e",
    "year": 2022,
    "partCodebookId": "ac0cdab2-8f52-4f0c-9b83-3b62925e09ef",
    "regionId": "7ba25bde-a2f0-4772-a0ac-9c771894e9a2",
    "provinceId": "d67d1988-57da-41d1-8b7d-b4186a1bd3b8",
    "originId": 2,
    "isAuthentic": true
}'
```
##### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

| **Tên Tham số** | **Mô tả**                         |**Bắt buộc**| **Kiểu dữ liệu**                | **Min,Max** | **Ví dụ**                                    |
|-----------------|-----------------------------------|--------------|---------------------------------|-------------|----------------------------------------------|
| carCompanyName    | Tên hãng xe                       | Bắt buộc     | Text                            | 1,255       | TOYOTA                                       |
| carModelName      | Tên hiệu xe                       | Bắt buộc     | Text                            | 1,255       | VIOS                                         |
| carVersionName    | Tên phiên bản xe                  | Bắt buộc     | Text                            | 1,255       | 1.5G CVT                                     |
| year            | Năm sản xuất                      | Bắt buộc     | Number                          | 1,9999      | 2022                                         |
| partCodebookName       | Tên codebook phụ tùng             | Bắt buộc     | Text                            | 1,255       | Cản trước (bđx trước)                        |
| partCodebookGoup        | Tên nhóm codebook phụ tùng        | Bắt buộc     | Text                            | 1,255       | Thân vỏ xe                                   |
| regionName     | Tên khu vực                       | Bắt buộc     | Text                            | 1,255       | Miền Nam                                     |
| provinceName     | Tên tỉnh trong khu vực            | Bắt buộc     | Text                            | 1,255       | Tỉnh Tây Ninh                                |
| originName   | Tên xuất xứ                       | Bắt buộc     | Text                            | 1,255       | LRTN                                         |
| garageName     | Tên garage sửa chữa               | Bắt buộc     | Text                            | 1,255       | CÔNG TY TNHH TOYOTA HƯNG THỊNH PHÁT TÂY NINH |
| minPrice        | Giá min của phụ tùng              | Bắt buộc     | Number                          | 1,999999999 | 3160600                                    |
| maxPrice        | Giá max của phụ tùng              | Bắt buộc     | Number                          | 1,999999999 | 3180600                                    |
| price        | Giá trị của phụ tùng              | Bắt buộc     | Number                          | 1,999999999 | 3170600                                    |
| histories | Danh sách lịch sử giá của tổ chức | Bắt buộc     | Array[partCodebookPriceHistory] | n           | []                                           |

*Chi tiết Object item  `partCodebookPriceHistory`*

| **Tên Tham số** | **Mô tả**                        |**Bắt buộc**| **Kiểu dữ liệu**                | **Min,Max** | **Ví dụ**         |
|-----------------|----------------------------------|--------------|---------------------------------|-------------|-------------------|
| carCompanyName    | Tên hãng xe                      | Bắt buộc     | Text                            | 1,255       | TOYOTA      |
| carModelName      | Tên hiệu xe                      | Bắt buộc     | Text                            | 1,255       | VIOS      |
| carVersionName    | Tên phiên bản xe                 | Bắt buộc     | Text                            | 1,255       | 1.5G CVT |
| year            | Năm sản xuất                     | Bắt buộc     | Number                          | 1,9999      | 2020              |
| partCodebookName       | Tên codebook phụ tùng            | Bắt buộc     | Text                            | 1,255       | Cản trước (bđx trước)              |
| partCustomerName       | Tên codebook phụ tùng khách hàng | Bắt buộc     | Text                            | 1,255       | 01XOC.00049              |
| partCodebookGoup        | Tên nhóm codebook phụ tùng       | Bắt buộc     | Text                            | 1,255       | Thân vỏ xe         |
| regionName     | Tên khu vực                      | Bắt buộc     | Text                            | 1,255       | Miền Nam         |
| provinceName     | Tên tỉnh trong khu vực           | Bắt buộc     | Text                            | 1,255       | Tỉnh Tây Ninh         |
| originName   | Tên xuất xứ                      | Bắt buộc     | Text                            | 1,255       | LRTN         |
| garageName     | Tên garage sửa chữa              | Bắt buộc     | Text                            | 1,255       | CÔNG TY TNHH TOYOTA HƯNG THỊNH PHÁT TÂY NINH         |
| minPrice        | Giá min của phụ tùng             | Bắt buộc     | Number                          | 1,999999999 | 3160600         |
| maxPrice        | Giá max của phụ tùng             | Bắt buộc     | Number                          | 1,999999999 | 3180600         |
| price        | Giá trị của phụ tùng             | Bắt buộc     | Number                          | 1,999999999 | 3170600         |

##### d. Ví dụ đầu ra
```
{
    "carCompanyName": "TOYOTA",
    "carModelName": "VIOS",
    "carVersionName": "1.5G CVT",
    "year": 2022,
    "partCodebookName": "Cản trước (bđx trước)",
    "partCodebookGoup": "Thân vỏ xe",
    "regionName": "Miền Nam",
    "provinceName": "Tỉnh Tây Ninh",
    "originName": "LRTN",
    "garageName": null,
    "price": 3170600,
    "minPrice": 3160600,
    "maxPrice": 3180600,
    "histories": [
        {
            "carCompanyName": "TOYOTA",
            "carModelName": "VIOS",
            "carVersionName": "1.5G CVT",
            "year": 2022,
            "partCodebookName": "Cản trước (bđx trước)",
            "partCustomerName": "01XOC.00049",
            "partCodebookGoup": "Thân vỏ xe",
            "regionName": "Miền Nam",
            "provinceName": "Tỉnh Tây Ninh",
            "originName": "LRTN",
            "garageName": null,
            "price": 3170600,
            "minPrice": 3160600,
            "maxPrice": 3180600
        }
    ]
}
```


