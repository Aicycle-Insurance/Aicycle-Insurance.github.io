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
https://api-aws-insurance.aicycle.ai

**API Key:** Liên hệ đội ngũ support tích hợp của AICycle để được cấp apiKey cho tổ chức.

## **2. API tích hợp**
### **2.1: Tạo claim folder**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/claimfolders |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimName|Tên folder claim|Bắt buộc|Text|1,255|Folder 1|
|vehicleBrandId|Id hãng xe|Bắt buộc|Number|1,9999|1|

**Chú thích**

> **ID hãng xe** (vehicleBrandId)
>
>|**Tên hãng xe**|**vehicleBrandId**|
>|---|---|
>|KIA Morning|1|
>|Toyota Innova|3|
>|Toyota Cross|4|
>|Toyota Vios|5|
>|Mazda Cx5|6|

**Ví dụ**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/claimfolders' \
--header 'authorization: Bearer $$apiKey$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimName":"34AAAAA",
    "vehicleBrandId":"5"
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
### **2.2: API lấy link upload ảnh lên s3 AICycle**
#### a. Thông tin cơ bản
|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/images/url |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|filePaths|Đường dẫn file trên S3|Optional|Text|1,255|INSURANCE_CLAIM/image-1.jpg|

**Ví dụ**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/images/url' \
--header 'Authorization: Bearer $$API_KEY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "filePaths": [
"INSURANCE_CLAIM/image-1.jpg"
    ]
}'
```


#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|urls|Url upload và lấy image trên s3|Bắt buộc|Object|1,255|INSURANCE_CLAIM/image-1.jpg|
|urls.fetchUrl|Url lấy ảnh|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/image-1.jpg|
|urls.uploadUrl|Url upload ảnh|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/image-1.jpg|
|urls.filePath|filePath dùng để call engine|Bắt buộc|Text|1,255|INSURANCE/image-1.jpg|

#### d. Ví dụ đầu ra
```
{
    "urls": {
        "fetchUrl": "https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/quang-6.jpg",
        "uploadUrl": "https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/quang-6.jpg",
        “filePath": “INSURANCE/image-1.jpg”
    },
    "msg": "Get url image success",
    "status": "success"
}
```

	
**Chú ý**
> Sau khi call api lấy link `uploadUrl` để up ảnh. Trên postman dùng link đó với method là PUT, body dạng binary và select file ảnh trên máy để upload. Khi nhận được status là 200 (upload thành công), dùng đường link `fetchUrl` đó paste lên browser để xem kết quả


### **2.3: API Call Engine AICycle**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai//claimimages/triton-assessment |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimId|Id của folder|Bắt buộc|Number|1,999999|123|
|imageName|Tên ảnh|Bắt buộc|Text|1,255|INSURANCE_CLAIM/image-1.jpg|
|filePath|Path ảnh trên s3 (lấy từ kết quả trả về  của API get imageUrl ở mục 2.2)|Bắt buộc|Text|1,255|INSURANCE_CLAIM/image-1.jpg|
|imageRangeId|ID Loại ảnh (Toàn cảnh, trung cảnh, cận cảnh)|Bắt buộc|Number|1,9999|1|
|partDirectionId|ID Hướng ảnh|Bắt buộc|Number|1,9999|3|
|oldImageId|Id ảnh muốn xóa|Optional|Number|1,9999|3|
|isClaim|True là giám định, false là cấp đơn|Bắt buộc|Boolean|n|true|

*Lưu ý*
> **Các giá trị của `imageRangeId`**
>
>|imageRangeName|imageRangeId|
>|--------------|------------|
>|Toàn cảnh|1|
>|Trung cảnh|2|
>|Cận cảnh|3|

> **Các giá trị của `partDirectionId`**
>
>|partDirectionName|partDirectionId|
>|---|---|
>|Trước|2|
>|45° Phải - Trước|3|
>|45° Trái - Trước|4|
>|Sau|5|
>|45° Phải - Sau|6|
>|45° Trái - Sau|7|
>|Phải - Trước|8|
>|Trái - Trước|9|
>|Phải - Sau|10|
>|Trái - Sau|11|

**Ví dụ**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/claimimages/triton-assessment' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimId": 1,
    "imageName": "INSURANCE/1666145614576/1666145614447.jpg",
    "filePath": "INSURANCE/1666145614576/1666145614447.jpg",
    "imageRangeId": 1,
    "partDirectionId": 2,
    "isClaim": false,
     “oldImageId": 123
}'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|car_parts|Chi tiết bộ phận và hỏng hóc|Bắt buộc|Array[carPart]|n|[]|
|damages|Chi tiết hỏng hóc|Bắt buộc|Array[damage]|n|[]|
|errorCodeFromEngine|Mã lỗi ảnh|Bắt buộc|Number|1,999999|0|
|message|Chi tiết lỗi ảnh|Bắt buộc|Text|1,255|Ảnh chụp qua màn hình|

*Chi tiết Object item  `carPart`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|class_uuid|Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
|location|Vị trí bộ phận|Bắt buộc|Text|1,255|Trái - Trước|
|car_part_name|Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
|mask_url|Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Chi tiết Object item `damage`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|damage_type_name|Tên hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước)|
|mask_url|mask hỏng hóc|Bắt buộc|Text|1,255|...|


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
    "isCarExistInProfile": false,
    "isPhotoValid": true,
    "traceId": "80540cf80585486f63432dcdf680fd20",
    "errorCodeFromEngine": 0,
    "message": "",
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
                ...
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
                        ...
                    ],
                    "car_part_name": "Khung kính cánh cửa trước trái",
                    "rawLocation": "Trái - Trước",
                    "car_part_color": "#21E0C1",
                    "mask_url": "{{s3Url}}"
                },
                ...
            ],
            "img_url": "{{s3Url}}"
        }
    ]
}

```

### **2.4: API lấy kết quả Folder (hồ sơ)**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | GET |
| API Url | https://api-aws-insurance.aicycle.ai/claimfolders/claim-results/{claimId} |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |

#### b. Chi tiết đầu vào
**Loại đầu vào**: Params

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|claimId|Id của folder|Bắt buộc|Number|1,999999|123|

**Ví dụ**
```
curl --location --request GET 'https://api-aws-insurance.aicycle.ai/claimfolders/claim-results/2450' \
--header 'authorization: Bearer $$API_KEY$$' \
--data-raw ''
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

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


### **2.5: API Call Engine AICycle Issue Insurance**
#### a. Thông tin cơ bản

|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/insurance/images/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |


#### b. Chi tiết đầu vào
**Loại đầu vào**: Body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|formFile|File ảnh|Bắt buộc|File|1|image.jpg|
|claimFolderId|Id của folder|Bắt buộc|Text|1,255|folder1|



**Ví dụ**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/insurance/images/upload' \
--header 'Authorization: Bearer 2cef5:44c332de62534cdea6f3c27b7afdba0aa254f90b562c47aa9087db66764be4be' \
--header 'Content-Type: application/json' \
--form 'formFile=@"/home/user/Downloads/image/IMG_9917.JPG"' \
--form 'claimFolderId="folder1"'
```

#### c. Chi tiết đầu ra
**Loại đầu ra**: Response body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|is_photo_valid|Ảnh có phải ô tô hay không|Bắt buộc|Boolean|1,999999|true|
|car_info|Thông tin xe|Bắt buộc|Object|n|{}|
|detected_item|Chi tiết bộ phận|Bắt buộc|Array[carPart]|n|[]|
|damaged_item|Chi tiết hỏng hóc|Bắt buộc|Array[damage]|n|[]|
|additional_data|Các thông tin bổ sung|Bắt buộc|Object|n|{}|
|errorCodeFromEngine|Mã lỗi ảnh|Bắt buộc|Number|1,999999|0|
|message|Chi tiết lỗi ảnh|Bắt buộc|Text|1,255|Ảnh chụp qua màn hình|

*Chi tiết Object item  `carPart`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|id|Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
|name|Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
|confidence|Độ dự đoán tin cậy|Bắt buộc|Text|0,1|0.80|
|mask|Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Chi tiết Object item `damage`*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|id|Mã hỏng hóc|Bắt buộc|Text|1,255|yfMzer07THdYoCI1SM2LN|
|title|Tiêu đề hỏng hóc|Bắt buộc|Text|1,255|Trầy (xước) ở cánh cửa|
|position_id|Mã vị trí chi tiết|Bắt buộc|Text|1,255|khung-kinh-canh-cua-truoc-trai-KMtJpH|
|damaged_type|Mã loại hỏng hóc|Bắt buộc|Text|1,255|MUCDO01|
|estimate_score|Mức độ tổn thất|Bắt buộc|Text|0,1|0.3|
|confidence|Độ dự đoán tin cậy|Bắt buộc|Text|0,1|0.8|
|mask|mask hỏng hóc|Bắt buộc|Text|1,255|...|


***Chi tiết Bảng Mã lỗi cùng httpStatus trả về của `errorCodeFromEngine`***

*Chú thích*
> Với những lỗi có level là error sẽ chỉ trả ra mã lỗi và message lỗi, những lỗi level warning sẽ trả ra mã lỗi, message, và detected_item, damaged_item như bình thường

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
    "is_photo_valid": true,
    "car_info": {
        "plate_number": "",
        "chassis_number": null,
        "car_company": "",
        "car_model": "",
        "car_color": [222, 220,216]
    },
    "detected_item": [
        {
            "id": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
            "name": "Trụ kính cánh cửa",
            "confidence": 0.80,
            "mask_url": "{{s3Url}}"
        }
    ],
    "damaged_item": [
        {
            "id": "yfMzer07THdYoCI1SM2LN",
            "title": "Trầy, xước ở Trụ kính cánh cửa",
            "position_id": "khung-kinh-canh-cua-truoc-trai-KMtJpH",
            "damaged_type": "",
            "estimate_score": 0.3,
            "confidence": 0.91,
            "mask_url": "{{s3Url}}"
        }
    ],
    "additional_data": {
        "img_url": "{{s3Url}}",
        ...
    }
}
```