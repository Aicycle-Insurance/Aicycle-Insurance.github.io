<h1 align="center">
    <img src="https://bucket-aicycle.s3.ap-southeast-1.amazonaws.com/logo.png" style="width: 400px;height: 100px;" alt="jsdeliver"> 
</h1>

## **Current version: 2.0.0**
| Version | Description | Menu Change Details |
| ---- | ----- |-------|
| 1.0.0 | Initial API Document |
| 1.0.1 | More details of errors which covered for the Engine APIs | Menu Error Output Details  |
| 1.0.2 | Add an API to return images which have pre-drawn masks | Menu API details |
| 1.0.3 | Add `oldImageId` field to engine API to delete old images |
| 1.0.4 | Change the API to get the image upload link (ignore requiring filePaths input and filePath will be return in response) | Menu Field Details |
| 2.0.0 | A BuyMe API change (claimFolderId is required) | Menu Field Details |

## **1. Document Specification**
### **1.1 Basic Information**
|||
|----|----|
| Method | |
| API Url | |
| API Headers | |

### **1.2 Response HTTP Codes**
- 200: OK
- 400: Bab request. Please consider the input data.
- 401: Authentication erros. Please review your authen-token
- 403: API not allow error. Don't have permission to access API
- 5XX: Error from AICycle, Please contact AICycle support team with `errorId` for more details.

### **1.3 Base Url:** 
https://api-aws-insurance.aicycle.ai

**API Key:** Please contact AICycle support team for the organization APIKEY.

## **2. APIs Integration**
### **2.1: Create claim folder**
#### a. Basic infomation
|||
|----|----|
| Method | POST |
| API Url | https://api.aicycle.ai/insurance/claimfolders |
| API Headers | `{ "Authorization": "Bearer $$APIKEY$$" }` |

#### b. Input Details
**Input Type**: Body

|**ParamName**|**Description**|**IsRequired**|**DataType**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|claimName|The Claim Folder name or partner's folder code|Yes|Text|1,255|My Test Folder 1|

**Note**

> **ID of Vehicle Brand in AIC system** (vehicleBrandId)
>
>|**Vehicle Brand Name**|**vehicleBrandId**|
>|---|---|
>|KIA Morning|1|
>|Toyota Innova|3|
>|Toyota Cross|4|
>|Toyota Vios|5|
>|Mazda Cx5|6|

**Sample curl**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/claimfolders' \
--header 'authorization: Bearer $$apiKey$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "claimName":"34AAAAA",
    "vehicleBrandId":"5"
}'
```

#### c. Response
**Response type**: Response body

|**Property Name**|**Description**|**IsRequred**|**Data Type**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|claimId|Id của claim folder|Yes|Text|1,255|123|

#### d. Sample response
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

### **2.2: API to initial `filePath` from AICycle Storage**
#### a. Basic infomation
|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/images/url |
| API Headers | `{ "Authorization": "Bearer $$apiKey$$" }` |
#### b. Input Details
**Input Type**: Body

|**Property Name**|**Description**|**IsRequired**|**DataType**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|filePaths|Expect `filePath` in AIC Storage|Optional|Text|1,255|INSURANCE_CLAIM/image-1.jpg|

**Sample request**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/images/url' \
--header 'Authorization: Bearer $$API_KEY$$' \
--header 'Content-Type: application/json' \
--data-raw '{
    "filePaths": [
"INSURANCE_CLAIM/image-1.jpg"
    ]
}'
```
#### c. Response
**Response type**: Response body

|**Property Name**|**Description**|**IsRequred**|**Data Type**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|urls.fetchUrl|Url to download image|Required|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/image-1.jpg|
|urls.uploadUrl|Url to upload image|Required|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_CLAIM/image-1.jpg|
|urls.filePath|`filePath` to be used for Engine API calls|Required|Text|1,255|INSURANCE/image-1.jpg|

#### d. Sample response
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
### **2.3: APIs to Call AICycle Engines**
#### 2.3.1. API return damage masks, car parts by image
##### a. Basic Infomation

|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/claimimages/triton-assessment |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |


#### b. Request Details
**Request type**: Body

|**ParamName**|**Description**|**IsRequired**|**DataType**|**Min,Max**|**Sample Data**|
|---|-------------------------------------------------------------|----------|---------|-------|--------------------------------|
|claimId| Id of Claim Folder | Required | Number  | 1,999999 | 123 |
|imageName| The Name of Image | Required | Text | 1,255 | INSURANCE_CLAIM/image-1.jpg |
|filePath| The `filePath` got from API to initial fildePath (Menu [2.2]()) | Required | Text | 1,255 | INSURANCE_CLAIM/image-1.jpg |
|imageRangeId| ID Image Type (wide range, mid-range, close-range) | Required | Number | 1,9999 | 1 |
|partDirectionId| ID of image direction | Required | Number | 1,9999 | 3 |
|oldImageId| The old image ID to be overwrited | Optional | Number  | 1,9999 | 3 |
|isClaim| True is ClaimMe, false is BuyMe | Required | Boolean | n  | true |
|location | The location to take picture | Optional | TEXT | 1,255 | Duy Tan, Cau Giay, Hanoi |
|requestedTime | The time to take picture | Optional | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
|uploadLocation | The location that is using Application | Optional | TEXT | 1,255 | Hai Ba Trung, Hanoi |

*Note*
> **Tha values of `imageRangeId`**
>
>|imageRangeName|imageRangeId|
>|--------------|------------|
>|Wide-range|1|
>|Mid-range|2|
>|Close-range|3|

> **The values of `partDirectionId`**
>
>|partDirectionName|partDirectionId|
>|---|---|
>|Front|2|
>|45° Right - Front|3|
>|45° Left - Front|4|
>|Back|5|
>|45° Right - Back|6|
>|45° Left - Back|7|
>|Right - Front|8|
>|Left - Front|9|
>|Right - Back|10|
>|Left - Back|11|

**Sample Request**
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

#### c. Response
**Response type**: Response body

|**Property Name**|**Description**|**IsRequred**|**Data Type**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|car_parts|The Car part and damage details|Required|Array[carPart]|n|[]|
|damages|Damage details|Required|Array[damage]|n|[]|
|errorCodeFromEngine|Image Error code|Required|Number|1,999999|0|
|message|Image Error code details|Required|Text|1,255|Image should not be a screen capture|
| img_size            | The image resolution               |Required| [Number]         | 1,9999999   | [1920,1080]           |
| extra_infor         | Some other image informations  |Required| Object           | n           | {}                    |

* Object item  `carPart` details*

|**Property Name**|**Description**|**IsRequred**|**Data Type**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|class_uuid|The car part code|Required|Text|1,255|den-gam-truoc-trai-d7WevY|
|location|The car part position|Required|Text|1,255|Left - Front|
|car_part_name|The car part name|Required|Text|1,255|Front left under-light|
|mask_url|The car part mask image url|Required|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Object item `damage` details*

|**Property Name**|**Description**|**IsRequred**|**Data Type**|**Min,Max**|**Sample Data**|
|---|---|---|---|---|---|
|damage_type_name|The damage name|Required|Text|1,255|Scratches|
|mask_url|Damage mask url|Required|Text|1,255|...|

*Object item `extra_info` details*

| **Parameter Name** | **Description** |**Required**| **DataType** | **Min,Max** | **Example** |
|--------|--------|---|------------ -----|------------|---------------|
| plate_number | License plate |Required| Text | 1,255 | 30A 9999 |
| car_company | Car manufacturer company |Required| Text | 1,255 | TOYOTA |
| car_model | Car Vehicle Brand |Required| Text | 1,255 | Vios |
| car_color | Car RGB color |Required| [Number] | 1.999999 | [220,219,215] |
| imagePosition | Image type id |Required| Number | 1.3 | 1 |
| corner_id | Shooting angle id |Required| Text | 1,255 | left-front-r6BEZd |


***Error Code detail Table and HttpStatus of `errorCodeFromEngine`***

*Note*
> For errors which have level is error, they will just return error code and error message. If the level is warning, errors will return error code, message, and car_part, car_damages as usual.

|**Error Code**|**HTTP Status**|**Response body**|**Error Level**|
|----|----|----|----|
|0|200|Valid image||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Cannot identify the car in the photo. Please retake"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Photo is blurry. Please retake"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Unable to download image"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Photo is dark"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Screenshot. Please retake"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Returned engine error"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Cannot identify the car in the photo. Please retake"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Snapshot blurred"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Image is not in the right angle. Please retake"}`|Error|

***Image type id details***

| **imagePosition** | **Image type** |
|-------------------|--------------|
| 1 | Wide-range |
| 2 | Mid-range |
| 3 | Close-range |


***corner_id details***

| **corner_id** | **Góc chụp**     |
|---------------|------------------|
| truoc-sT9qgX             | Front            |
| 45-phai-truoc-UoYzs6             | 45° Right - Front |
| 45-trai-truoc-C1xM02            | 45° Left - Front |
| sau-htBwjB          | Back              |
| 45-phai-sau-fRzY3r           | 45° Right - Back   |
| 45-trai-sau-1q3G3J            | 45° Left - Back   |
| phai-truoc-eYWg1d            | Right - Front     |
| trai-truoc-r6BEZd            | Left - Front     |
| phai-sau-v1hAm6            | Right - Back       |
| trai-sau-t8QgFO            | Left - Back       |
| phai-4wif2Z            | Right             |
| trai-MyuVUE            | Left             |

#### d. Sample response

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
                "imagePosition": 1,
                "corner_id": trai-truoc-r6BEZd
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
#### 2.3.2. Mask pre-drawing API
##### a. Basic Information
|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/claimimages/triton-assessment-box |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |                         |


#### b. Request details
**Request type**: Body

| **Parameter Name** | **Description** |**Required**| **DataType** | **Min,Max** | **Example** |
|---------------------------------------------- --------------------------------------------------- ---------------------|---|------------------|----- --------|-----------------------------------------|
| claimId | The Claim Folder ID |Required| Number | 1.999999 | 123 |
| imageName | Image name |Required| Text | 1,255 | INSURANCE_CLAIM/image-1.jpg |
| filePath | The `filePath` got from API to initial fildePath (Section [2.2]()) |Required| Text | 1,255 | INSURANCE_CLAIM/image-1.jpg |
| imageRangeId | ID Image Type (wide range, mid-range, close-range) |Required| Number | 1.9999 | 1 |
| partDirectionId | ID Image orientation |Required| Number | 1.9999 | 3 |
| oldImageId | Image ID which want to delete |Optional| Number | 1.9999 | 3 |
| isClaim | True is ClaimMe, false is BuyMe |Required| Boolean | n | true |
| location | Where the photo was taken | Optional | TEXT | 1,255 | Duy Tan, Cau Giay, Hanoi |
| requestedTime | Time to take pictures | Optional | TEXT | 1,255 | 2023-03-24 03:28:29.414232 +00:00 |
| uploadLocation | Where to upload photos to make claims | Optional | TEXT | 1,255 | Hai Ba Trung, Hanoi |
| isValidate | true to enable image validation (screenshot, wrong angle,...), false to turn off (default is false) | Optional | Boolean | n | true |

*Note*
> **`imageRangeId` values**
>
>|imageRangeName|imageRangeId|
>|--------------|------------|
>|Wide-range|1|
>|Mid-range|2|
>|Close-range|3|

> **`partDirectionId` values**
>
>|partDirectionName|partDirectionId|
>|---|---|
>|Front|2|
>|45° Right - Front|3|
>|45° Left - Front|4|
>|Back|5|
>|45° Right - Back|6|
>|45° Left - Back|7|
>|Right - Front|8|
>|Left - Front|9|
>|Right - Back|10|
>|Left - Back|11|

**Example**
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

#### c. Response
**Response type**: Response body

| **Parameter Name** | **Description** |**Required**| **DataType** | **Min,Max** | **Example** |
|---------------------|--------------------------- ---|---|------------------|------------|--------- --------------|
| car_parts | Car Part and Damage details |Required| Array[carPart] | n | [] |
| damage | Damage details |Required| Array[damage] | n | [] |
| errorCodeFromEngine | Engine error code |Required| Number | 1.999999 | 0 |
| message | Engine error details |Required| Text | 1,255 | Screenshot |
| mask_url | Mask url |Required| Text | 1,255 | {{maskUrl}} |
| img_size | Image Size |Required| [Number] | 1.9999999 | [1920,1080] |
| extra_infor | More image information |Required| Object | n | {} |

*Object item  `carPart` details*

|**Parameter Name**|**Description**|**Required**|**DataType**|**Min,Max**|**Example**|
|---|---|---|---|---|---|
|class_uuid|Car part Code|Required|Text|1,255|den-gam-truoc-trai-d7WevY|
|location|Car part position|Required|Text|1,255|Left - Previous|
|car_part_name|Car part name|Required|Text|1,255|Left front headlight|
|mask_url|Url mask from Engine|Required|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Object item `damage` details*

|**Parameter Name**|**Description**|**Required**|**DataType**|**Min,Max**|**Example**|
|---|---|---|---|---|---|
|damage_type_name|Damage type name|Required|Text|1,255|Scratch (scratch)|
|mask_url|Damage mask image URL|Required|Text|1,255|...|

*Object item `extra_info` details*

| **Parameter Name** | **Description** |**Required**| **DataType** | **Min,Max** | **Example** |
|--------|--------|---|------------ -----|------------|---------------|
| plate_number | License plate |Required| Text | 1,255 | 30A 9999 |
| car_company | Car manufacturer company |Required| Text | 1,255 | TOYOTA |
| car_model | Car Vehicle Brand |Required| Text | 1,255 | Vios |
| car_color | Car RGB color |Required| [Number] | 1.999999 | [220,219,215] |
| imagePosition | Image type id |Required| Number | 1.3 | 1 |
| corner_id | Shooting angle id |Required| Text | 1,255 | left-front-r6BEZd |

***Error Code detail Table and HttpStatus of `errorCodeFromEngine`***

*Note*
> For errors which have level is error, they will just return error code and error message. If the level is warning, errors will return error code, message, and car_part, car_damages as usual.

|**Error Code**|**HTTP Status**|**Response body**|**Error Level**|
|----|----|----|----|
|0|200|Valid image||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Cannot identify the car in the photo. Please retake"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Photo is blurry. Please retake"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Unable to download image"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Photo is dark"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Screenshot. Please retake"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Returned engine error"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Cannot identify the car in the photo. Please retake"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Snapshot blurred"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "Image is not in the right angle. Please retake"}`|Error|

***Image type id details***

| **imagePosition** | **Image type** |
|-------------------|--------------|
| 1 | Wide-range |
| 2 | Mid-range |
| 3 | Close-range |


***corner_id details***

| **corner_id** | **Góc chụp**     |
|---------------|------------------|
| truoc-sT9qgX             | Front            |
| 45-phai-truoc-UoYzs6             | 45° Right - Front |
| 45-trai-truoc-C1xM02            | 45° Left - Front |
| sau-htBwjB          | Back              |
| 45-phai-sau-fRzY3r           | 45° Right - Back   |
| 45-trai-sau-1q3G3J            | 45° Left - Back   |
| phai-truoc-eYWg1d            | Right - Front     |
| trai-truoc-r6BEZd            | Left - Front     |
| phai-sau-v1hAm6            | Right - Back       |
| trai-sau-t8QgFO            | Left - Back       |
| phai-4wif2Z            | Right             |
| trai-MyuVUE            | Left             |


#### d. Sample response

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
                "imagePosition": 1,
                "corner_id": trai-truoc-r6BEZd
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
            "img_url": "{{s3Url}}",
            "mask_url": "{{s3Url}}"
        }
    ]
}

```
#### **2.3.3: API call Engine AICycle (Single level for direct image upload, not recommended)**
##### a. Basic information

|||
|----|----|
| Method | POST |
| API Url | https://api-aws-insurance.aicycle.ai/insurance/images/upload |
| API Headers | `{ "Authorization": "Bearer $$API_KEY$$" }` |


#### b. Request details
**Request type**: Body

|**Parameter Name**|**Description**|**Required**|**DataType**|**Min,Max**|**Example**|
|---|---|---|---|---|---|
|formFile|Image File from client|Required|File|1|image.jpg|
|claimFolderId|Claim Folder ID will include image|Required|Text|1,255|folder1|

**Example**
```
curl --location --request POST 'https://api-aws-insurance.aicycle.ai/insurance/images/upload' \
--header 'Authorization: Bearer 2cef5:44c332de62534cdea6f3c27b7afdba0aa254f90b562c47aa9087db66764be4be' \
--header 'Content-Type: application/json' \
--form 'formFile=@"/home/user/Downloads/image/IMG_9917.JPG"' \
--form 'claimFolderId="folder1"'
```

#### c. Response
**Response type**: Response body

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|is_photo_valid|Ảnh có phải ô tô hay không|Bắt buộc|Boolean|1,999999|true|
|car_info|Thông tin xe|Bắt buộc|Object|n|{}|
|detected_item|Chi tiết bộ phận|Bắt buộc|Array[carPart]|n|[]|
|damaged_item|Chi tiết hỏng hóc|Bắt buộc|Array[damage]|n|[]|
|additional_data|Các thông tin bổ sung|Bắt buộc|Object|n|{}|
|errorCodeFromEngine|Mã lỗi ảnh|Bắt buộc|Number|1,999999|0|
|message|Chi tiết lỗi ảnh|Bắt buộc|Text|1,255|Ảnh chụp qua màn hình|

|**Parameter Name**|**Description**|**Required**|**DataType**|**Min,Max**|**Example**|
|---|---|---|---|---|---|
|is_photo_valid|Is the photo a car|Required|Boolean|1,999999|true|
|car_info|Vehicle Info|Required|Object|n|{}|
|detected_item|Car parts details|Required|Array[carPart]|n|[]|
|damaged_item|Damage Details|Required|Array[damage]|n|[]|
|additional_data|Additional Information|Required|Object|n|{}|
|errorCodeFromEngine|Engine Error Code|Required|Number|1,999999|0|
|message|Engine error message details|Required|Text|1,255|Screenshot|

*Object item  `carPart` details*

|**Tên Tham số**|**Mô tả**|**Bắt buộc**|**Kiểu dữ liệu**|**Min,Max**|**Ví dụ**|
|---|---|---|---|---|---|
|id|Mã bộ phận|Bắt buộc|Text|1,255|den-gam-truoc-trai-d7WevY|
|name|Tên bộ phận|Bắt buộc|Text|1,255|Đèn gầm trước trái|
|confidence|Độ dự đoán tin cậy|Bắt buộc|Text|0,1|0.80|
|mask|Url mask bộ phận|Bắt buộc|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

|**Parameter Name**|**Description**|**Required**|**DataType**|**Min,Max**|**Example**|
|---|---|---|---|---|---|
|id|Car part ID|Required|Text|1,255|den-gaming-truc-trai-d7WevY|
|name|Car part name|Required|Text|1,255|Left front bumper light|
|confidence|Confidentiality|Required|Text|0.1|0.80|
|mask|Car part mask Image URL|Required|Text|1,255|https://dyta7vmv7sqle.cloudfront.net/INSURANCE_RESULT/EGfDHztL2Vffgc72cM1DG.png|

*Object item `damage` details*

|**Parameter Name**|**Description**|**Required**|**DataType**|**Min,Max**|**Example**|
|---|---|---|---|---|---|
|id|Damage Code|Required|Text|1,255|yfMzer07THdYoCI1SM2LN|
|title|Broken title|Required|Text|1,255|Scratched (scratched) on the door|
|position_id|Location detail Code|Required|Text|1,255|Framework
|damaged_type|Damage Type Code|Required|Text|1,255|MUCDO01|
|estimate_score|Damage Level|Required|Text|0.1|0.3|
|confidence|Confidentiality|Required|Text|0.1|0.8|
|mask|Damage mask|Required|Text|1,255|...|

***Error Code detail Table and HttpStatus of `errorCodeFromEngine`***

*Note*
> For errors which have level is error, they will just return error code and error message. If the level is warning, errors will return error code, message, and car_part, car_damages as usual.

|**Error Code**|**HTTP Status**|**Response body**|**Error Level**|
|----|----|----|----|
|0|200|Valid image||
|37143|400|`{"errorCodeFromEngine": 37143, "message": "Cannot identify the car in the photo. Please retake"}`|Error|
|40412|400|`{"errorCodeFromEngine": 40412, "message": "Photo is blurry. Please retake"}`|Error|
|32324|400|`{"errorCodeFromEngine": 32324, "message": "Unable to download image"}`|Error|
|23212|200|`{"errorCodeFromEngine": 23212, "message": "Photo is dark"}`|Warning|
|84680|400|`{"errorCodeFromEngine": 84680, "message": "Screenshot. Please retake"}`|Error|
|47565|400|`{"errorCodeFromEngine": 47565, "message": "Returned engine error"}`|Error|
|67219|400|`{"errorCodeFromEngine": 67219, "message": "Cannot identify the car in the photo. Please retake"}`|Error|
|77704|200|`{"errorCodeFromEngine": 77704, "message": "Snapshot blurred"}`|Warning|
|50676|400|`{"errorCodeFromEngine": 50676, "message": "The picture is not from the right angle. Please retake"}`|Error|


#### d. Sample response

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
            "name": "Door glass pillar",
            "confidence": 0.80,
            "mask_url": "{{s3Url}}"
        }
    ],
    "damaged_item": [
        {
            "id": "yfMzer07THdYoCI1SM2LN",
            "title": "Scratches on the door glass pillar",
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