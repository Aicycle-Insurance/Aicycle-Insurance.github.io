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
![alt text](https://drive.google.com/uc?id=17vpFwMIM6yqLUx5iA7HOOj-QeqcwVqVd)
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
| modelCode          | Mã hiệu xe của tổ chức                                                       | Tùy chọn     | Text             | 1,255   | code-hieu-xe      |
| versionCode        | Mã phiên bản xe của tổ chức                                                  | Bắt buộc     | Text             | 1,255   | code-phien-ban-xe |
| year               | Năm sản xuất                                                      | Bắt buộc     | Number           | 1,9999  | 2020              |
| minPrice           | Giá min của xe                                                    | Bắt buộc     | Number           | 1,999999999 | 100000000         |
| maxPrice           | Giá max của xe                                                    | Bắt buộc     | Number           | 1,999999999 | 300000000         |
| carValue           | Giá trị xe                                                        | Bắt buộc     | Number           | 1,999999999 | 300000000         |
| isWarning          | Thông tin về dòng xe này trên thị trường có đang thiếu hay không? | Bắt buộc     | Boolean          | n       | true              |
| listedPrice        | Giá xe niêm yết                                                   | Bắt buộc     | Number           | 1,999999999        | 320000000         |
| minListedPrice     | Giá trị min của giá niêm yết                                      | Bắt buộc     | Number          | 1,999999999        | 305000000         |
| maxListedPrice     | Giá trị max của giá niêm yết                                      | Bắt buộc     | Number          | 1,999999999        | 330000000         |
| hanoiOnRoadPrice   | Gía lăn bánh tại Hà Nội                                           | Bắt buộc     | Number          | 1,999999999        | 323000000         |
| hcmOnRoadPrice     | Gía lăn bánh tại TP HCM                                           | Bắt buộc     | Number          | 1,999999999        | 323000000         |
| generalOnRoadPrice | Gía lăn bánh tại các tỉnh khác                                    | Bắt buộc     | Number          | 1,999999999        | 323000000         |
| listAvailableYears | Danh sách các năm gợi ý hợp lệ cho CodeBook hiện tại                   | Bắt buộc     | Array[Number]          | 1,999999999        | [2020,2021]         |
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

### **3.5: API định giá xe v1 (sử dụng CodeBook hãng hiệu và phiên bản của AICycle) (deprecated)**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/valuation/car-valuate |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**       | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**         |
|-----------------|-----------------|--------------|------------------|-------------|-------------------|
| carCompanyId    | Id hãng xe      | Bắt buộc     | Text             | 1,255       | uuid-hang-xe      |
| carModelId      | Id hiệu xe      | Bắt buộc     | Text             | 1,255       | uuid-hieu-xe      |
| carVersionId    | Id phiên bản xe | Bắt buộc     | Text             | 1,255       | uuid-phien-ban-xe |
| year            | Năm sản xuất    | Bắt buộc     | Number           | 1,9999      | 2020              |

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

## **3. APIs tích hợp BuyMe**
### **3.1: Flow diagram tích hợp cho BuyMe realtime**
![alt text](https://drive.google.com/uc?id=1anBAk9Z3eyEZKY2huJNhJAvsRs_aS6Aq)
### **3.2: Tạo mới hồ sơ (claimFolder)**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/claimfolders |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimName|Mã hồ sơ của đối tác|Bắt buộc|Text|1,255|Folder-1|
|vehicleBrandName|Tên hãng xe của hồ sơ||Text|1,255|mazda|
|vehicleModel|Tên hiệu xe của hồ sơ||Text|1,255|mazda.3_sedan|
|vehicleSpec|Tên phiên bản xe của hồ sơ||Text|1,255|luxury_1_5l_at|
|vehicleLicensePlates|Tên hãng xe của hồ sơ||Text|1,255|30H84142|

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
| claimId         | Id của Hồ sơ                                                            | Bắt buộc | Number           | 1,999999    | 123 |
| imageName       | Tên ảnh                                                                  | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| filePath        | Path ảnh trên s3 (lấy từ kết quả trả về  của API get imageUrl ở mục 2.2) | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| position        | Slug Loại ảnh (Toàn cảnh, trung cảnh, cận cảnh)                          | Bắt buộc | Text             | 1,255       | toan-canh-afh4l5 |
| direction       | Slug Hướng ảnh                                                           | Bắt buộc | Text             | 1,255       | 45-trai-truoc-C1xM02 |
| oldImageId      | Id ảnh cũ muốn xóa                                                       | Optional | Number           | 1,9999      | 3 |

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

### **3.7: API lấy kết quả Hồ sơ (nhóm theo ảnh)**
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

### **3.8: API kiểm tra các ảnh trong Hồ sơ**
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


### **3.9: API callback lưu kết quả từ khách hàng**
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
**Loại đầu ra**: Response body
| **Tên Tham số** | **Mô tả**    | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ** |
|---------|--------------|-----------|---------|----------|-------|
| claimId | Tên folder claim | Bắt buộc  | Number  | 1,999999 | 123   |
| carPlate | Biển số xe   | Bắt buộc  | TEXT    | 1,255    | 30E 64737 |
| carCompany | Hãng xe      | Bắt buộc  | TEXT    | 1,255    | HYUNDAI |
| carModel | Mẫu xe       | Bắt buộc  | TEXT    | 1,255    | i10   |
| damages | Danh sách các hỏng hóc | Bắt buộc  | Array[damage] | n        | []    |
| ownerOrganizationId | Id tổ chức   | Bắt buộc  | Number  | 1,999999 | 1     |

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

## 3. APIs tích hợp Claim Me
##### a. Thông tin cơ bản

||                                                      |
|----|------------------------------------------------------|
| Method | POST                                                 |
| API Url | https://api.aicycle.ai/insurance/v2/claim-me/process |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }`          |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

| **Tên Tham số** | **Mô tả**                                                                | **Bắt buộc** | **Kiểu dữ liệu** | **Min,Max** | **Ví dụ**                    |
|-----------------|--------------------------------------------------------------------------|----------|------------------|-------------|------------------------------|
| claimId         | Id của folder                                                            | Bắt buộc | Number           | 1,999999    | 123 |
| imageName       | Tên ảnh                                                                  | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| filePath        | Path ảnh trên s3 (lấy từ kết quả trả về  của API get imageUrl ở mục 2.2) | Bắt buộc | Text             | 1,255       | INSURANCE_CLAIM/image-1.jpg |
| position        | Slug Loại ảnh (Toàn cảnh, trung cảnh, cận cảnh)                          | Bắt buộc | Text             | 1,255       | toan-canh-afh4l5 |
| direction       | Slug Hướng ảnh                                                           | Bắt buộc | Text             | 1,255       | 45-trai-truoc-C1xM02 |
| oldImageId      | Id ảnh cũ muốn xóa                                                       | Optional | Number           | 1,9999      | 3 |





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