***
**Main API URL**  --> https://api.shoplater.app/

**User-Agent:** shoplater/301 CFNetwork/1474 Darwin/23.0.0

**Authorization:** JWT Key

**Data Storage Infrastructure:** Google App Engine & Google Storage Buckets & Google Firebase
***
## Save URL to Server

`In order to call the API you must first Resquest the product page URL and parse it, retrieving the URL of the product, its title, its image, its owner, and the icon of the website (in the Client Side)`

>POST https://api.shoplater.app/api/v1/saves
>Requires Authorization --> JWT key

### Request Structure:
---
>{"url": {URL}, "item":{"title":{PRODUCT_NAME},"image":{PRODUCT_IMAGE}, "owner":{BRAND_OR_USER}, "url": {PRODUCT_URL}, "favicon": {STORE_ICON_URL}}}

### Request Payload Example:
---
`{"url":"https://www.ikea.com/pt/pt/p/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura-s89421969/", "item":{"title":"BESTÅ Armário parede c/2portas, preto-castanho Björköviken/castanho chapa de carvalho c/velatura, 60x22x128 cm - IKEA","image":"https://www.ikea.com/pt/pt/images/products/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura__1024316_pe833495_s5.jpg", "owner":"ikea.com", "url":"https://www.ikea.com/pt/pt/p/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura-s89421969/", "favicon":"https://icons.duckduckgo.com/ip3/ikea.com.ico"}}`

### Response:
---
><u>Success Status:</u> HTTP 201 Created
><u>Returns:</u> PRODUCT_ID of the created Product


## Save Product to Collection

>POST https://api.shoplater.app/api/v1/collections/{COLLECTION_ID}/items
>Requires Authorization --> JWT key

### Request Structure:
---
>{"itemId":{PRODUCT_ID}}

### Request Payload Example:
---
`{"itemId": 2799}`

### Response:
---
><u>Success Status:</u> HTTP 201 Created


## **Get Products by ID:**

>GET https://api.shoplater.app/api/v1/items/{ITEM_NUMBER}/

### Response `{ITEM_NUMBER: 2799}`:
---
`{"id":2799,"userId":null,"storeId":51,"available":null,"creation":"2023-11-04T17:47:39.035Z","lastUpdated":"2023-11-04T17:47:39.035Z","currency":null,"url":"https://www.ikea.com/pt/pt/p/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura-s89421969/","title":"BESTÅ Armário parede c/2portas, preto-castanho Björköviken/castanho chapa de carvalho c/velatura, 60x22x128 cm - IKEA","description":null,"favicon":"https://icons.duckduckgo.com/ip3/ikea.com.ico","image":"https://www.ikea.com/pt/pt/images/products/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura__1024316_pe833495_s5.jpg","imageDimensions":null,"imageType":null,"imageInternal":null,"imageInternalAlt":null,"imageReference":null,"label":null,"lastUpdatedPrice":null,"owner":"ikea.com","ownerName":null,"productName":null,"safeSearch":null,"scrapperGenerated":false,"relatedUrls":null,"type":null,"performance":null,"imageAlt":[],"store":{"id":51,"url":"https://www.ikea.com","name":"ikea.com","logo":"https://icons.duckduckgo.com/ip3/ikea.com.ico","invalidSelectorCount":null,"selectorStatus":null,"lastUpdated":"2023-11-04T17:47:39.112Z","creation":"2023-03-22T00:01:48.940Z"}}`

#### Warning:
---
Changing the item number will leak other users products even if in a private collection (without the User ID, so probably fine?)

#### Note #1:
---
Some Images are stored on ShopLater's Google Storage Bucket (imageInternal) and have the original image URL whereas others are not and instead only the own product's website image URL is stored.  Probably the ones that get stored are used for training the recommendation system.

#### Note #2:
---
For store icons, it seems like https://icons.duckduckgo.com/ is always used, but we could also probably use the stores own favicon (via HTML tag) instead as it would be more reliable across less known websites, at least as a fallback mechanism (it may already be implemented this way though).


## **Get Stored Google Product Images**

> Bucket: https://storage.googleapis.com/shoplater/

### Get images of all products in public Collections:
---
>GET https://storage.googleapis.com/shoplater/item_images/{PRODUCT_ID}_{IMAGE_ID}.jpg

#### Get list of all product images:
---
>GET https://storage.googleapis.com/shoplater/

Includes:
1. Size Information
2. User ID associated
3. Image ID and Location
4. Time of Generation
5. Meta Generation --> (whether it was scrapped)
6. Last Modified
7. ETAG --> (Product Type Tag)

#### Google Access ID for Scraper:
-    scrapper@shoplater.iam.gserviceaccount.com

## Get Price History by Product ID

>GET https://api.shoplater.app/api/v1/items/{ITEM_NUMBER}/prices
>_Params --> `?period={PERIOD}&groupBy={GROUP_FREQ}`_

### Response `{PERIOD: week, GROUP_FREQ: day}`:
---
`[{"date":"2023-11-04T21:52:23.270Z","minimumPrice":"399"}]`

#### Note #1:
---
It doesn't seem to be currently working for some products, including the product from IKEA or the one from Amazon that I uploaded. (returns an empty list `[]`)


## Get Related Products by Product ID

>GET https://api.shoplater.app/api/v1/items/{ITEM_NUMBER}/related

### Request Example:
---
GET https://api.shoplater.app/api/v1/items/2799/related

>Response Code: HTTP/2 204 No Content

#### Note #1:
---
It's unclear whether all requests end up on a No Content Header Response because of the unimplemented recommendation algorithm or if there is already a simple algorithm being used and it doesn't have enough data. Further testing needed.

## Get Recommended User Feed:

>GET https://api.shoplater.app/api/v1/users/{USER_ID}/feed
>_Params --> `?page={PAGE}&skip={SK}&take={TAKE}&uid={USER_ID}&generate={GEN}&limitAltImages={LIM}`_

-   SKIP --> Amount of elements to skip
-   TAKE --> Number of elements to return
-   Returns: `[SKIP, SKIP+TAKE]` elements if SKIP and TAKE present

### Response {PAGE:0 ,SK:0 ,TAKE:10 ,USER_ID: ..., GEN:TRUE, LIM:2}:
---
`{"count":93,"products":[{"id":1920,"title":"tina earrings [andreea ali for CINCO]","url":null,"favicon":"https://www.google.com/s2/favicons?domain=cinco-store.com&size=256.ico","createdOn":"2023-03-31T15:02:09.601Z","ownerName":null,"currency":"EUR","owner":"cinco-store.com","user":{"id":"4GTCtSUWW4XUje2d9LbtediyFh33","username":"juliewi","photoURL":"https://firebasestorage.googleapis.com:443/v0/b/shoplater.appspot.com/o/user-avatars%2F4GTCtSUWW4XUje2d9LbtediyFh33.png?alt=media&token=601cf8be-1b63-4a44-b3ec-63827ace78b8"},"prices":[{"value":95},{"value":95}],"isSavedByUser":null,"image":"https://cinco-store.com/media/catalog/product/g/i/gif_test.gif"},{"id":1951,...}}`

### Note #1:
---
It seems like there is some sort of recommendation algorithm going on, it doesn't appear in the app though and it will have to be basically rewritten from scratch.


## User Account API

### Get User Details
---
>GET https://api.shoplater.app/api/v1/api/v1/users/{USER_ID}/

#### Response `{USER_ID: 5N8HqK8ZJMVvii8viBTKaqPdy7M2}`:
---
`{"id":673,"documentId":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","displayName":"joaofrancisco.rarb@gmail.com","photoURL":null,"phoneNumber":null,"email":"joaofrancisco.rarb@gmail.com","firstName":"João","lastName":"Barata","username":"joaofbarata","uid":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","providerData":[{"uid":"joaofrancisco.rarb@gmail.com","email":"joaofrancisco.rarb@gmail.com","providerId":"password"}],"disabled":false,"banned":false,"creationDate":"2023-11-01T15:30:21.000Z","private":false,"following":false,"interests":[{"id":3,"name":"Technology","image":"https://firebasestorage.googleapis.com/v0/b/shoplist-319316.appspot.com/o/interests%2FTechnology.png?alt=media&token=16c159dd-d5ea-478f-98d4-682bce0c987a","creation":"2023-02-09T00:27:32.025"},{"id":10,"name":"Men's Apparel","image":"https://firebasestorage.googleapis.com/v0/b/shoplist-319316.appspot.com/o/interests%2FMensApparel.png?alt=media&token=220c7a5d-fe88-4a85-b1f9-544ad6d25ef0","creation":"2023-02-09T17:11:01.142651"},{"id":15,"name":"Watches","image":"https://firebasestorage.googleapis.com/v0/b/shoplist-319316.appspot.com/o/interests%2FWatches.png?alt=media&token=5982e872-e6f1-4e5c-949a-a5d09849434f","creation":"2023-02-09T17:14:05.323414"}]}`

### Get Followers and Following Users
---
>GET https://api.shoplater.app/api/v1/users/USER_ID/follow
>_Params --> `?type={FOLLOW}&skip={SK}&take={TAKE}`_

-   SKIP --> Amount of elements to skip
-   TAKE --> Number of elements to return
-   Returns: `[SKIP, SKIP+TAKE]` elements if SKIP and TAKE present

#### Get Following
---
_Params: `{FOLLOW: FOLLOWING}`_

#### Get Followers
---
_Params: `{FOLLOW: FOLLOWER}`_

#### Response `{FOLLOW: FOLLOWING, SKIP: 0, TAKE: 10}`:
---
`{"count":"0","data":null}`
-   `data --> following/followers`
-   `count --> length of data`

#### Note #1:
---
I am currently unable to follow any collection through the app as the option does not appear anywhere besides the setup screen.

## Collections API

#### Create a Collection:
---
>POST https://api.shoplater.com/api/v1/collections
>Requires Authentication --> JWT key

##### Request Structure:
---
>{"title": {TITLE}, "description": {DESC}, "private": {PRIV}}

##### Response:
---
><u>Success Status:</u> HTTP 201 Created

##### Request Payload Example:
---
`{"title":"Lets Go","description":"","private":true}`

##### Response Example:
---
`{"userId":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","description":"","private":true,"title":"Lets Go","id":426,"creation":"2023-11-05T18:33:53.496Z","lastUpdated":"2023-11-05T18:33:53.496Z"}`
   \\--> _New Collection Object_

#### Get My Collections:
---
>GET https://api.shoplater.app/api/v1/collections
>_Params --> `?skip={SK}&take={TAKE}&items={ITEM_NUM}&userId={USER_ID}`_

-   SKIP --> Amount of elements to skip
-   TAKE --> Number of collections to return
-   ITEMS --> Number of items to return per collection
-   Returns: `[SKIP, SKIP+TAKE]` elements if SKIP and TAKE present with `ITEMS` number of Items per collection

##### Request:
---
>Requires Authorization --> JWT key

##### Response Example:
---
`{"count":"2","data":[{"id":426,"title":"Lets Go","userId":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","lastUpdated":"2023-11-05T18:33:53.496509","creation":"2023-11-05T18:33:53.496509","description":"","private":true,"items":[null],"itemExistsInCollection":false},{"id":420,"title":"Private","userId":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","lastUpdated":"2023-11-04T17:47:42.095258","creation":"2023-11-01T15:36:13.830396","description":"","private":true,"items":[{"id":2799, ...}]}]}}`

#### Get Collection By ID:
---
>GET https://api.shoplater.app/api/v1/collections/{COLLECTION_ID}

##### Response `{COLLECTION_ID: 420}`:
---
`{"data":[{"itemExistsInUserCollection":false,"id":518,"collectionId":420,"userId":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","lastUpdated":"2023-11-01T15:36:14.072Z","creation":"2023-11-01T15:36:14.072Z","itemId":2793,"item":[{"id":2793,"userId":null,"storeId":3,"url":"https://www.amazon.com/...","available":true,"currency":null,"title":"Wjnvfioo Business Blazer Men...","description":null,"creation":"2023-11-01T15:36:02.753421","lastUpdated":"2023-11-01T15:36:58.377016","favicon":"https://icons.duckduckgo.com/ip3/amazon.com.ico","image":"\"https://m.media-amazon.com/images/I/51YQpQJrswL._AC_UY1000_.jpg\"data-fling-asin=\"B0C4TNFZ7H\"data-fling-refmarker=\"detail_main_image_block\"data-zoom-hires=\"https://m.media-amazon.com/images/I/51YQpQJrswL._AC_UL1500_.jpg\"onload=\"loadMainImage(this)\"class=\"lookbook-img-content\"data-a-hires=\"https://m.media-amazon.com/images/I/51YQpQJrswL._AC_UY1000_.jpg\"","imageDimensions":null,"imageType":null,"imageInternal":null,"imageReference":null,"label":null,"lastUpdatedPrice":null,"owner":"amazon.com","ownerName":"Amazon","productName":null,"relatedUrls":null,"safeSearch":null,"scrapperGenerated":true,"type":"create","performance":{"pageLoad":23108,"getMetada":9,"fetchTitle":25,"fetchImageAlt":3014,"fetchDescription":1001,"fetchPriceFailed":1004,"cloudinaryFailedImg":1954},"imageInternalAlt":[{"image":"https://storage.googleapis.com/shoplater/item_images/2793_2V0TT.jpg?GoogleAccessId=scrapper%40shoplater.iam.gserviceaccount.com&Expires=16447017600&Signature=ZdXCFTxPeKQ94xSUg4nj7j2diRxj1O%2Bb%2FNzrRoB5ROzXFIGYb2AlZ%2Bh12z3PGvewiOiSCPjzefz11%2BFz4jDOJhXYmCgeyWaEKFmVdSQhoCOmO%2BTw2g5zEvzCYI%2FvOVoV%2BoarE1zBSf%2B%2Fy0Iu%2FN8nR7XnVcASHC24fAzjl%2B4BF6DdZScOBDaE1K6wZOhQVNaoyynhPL6JRuWQEJirxBiUHQDQhBJH5G47p8cqpjWFsZxEnbkMeA3MtYOD7%2BgIVOgYceQJuWcWeFb8FrdLI3ydqxPc2RfwtrPe9csgOtV%2FiBnoRa7aAptjYuc3jlMc0Qye%2BkkS5BmymPm1eGkweG0bdQ%3D%3D","gcsUri":"gs://shoplater/item_images/2793_2V0TT.jpg"},{"image":"https://storage.googleapis.com/shoplater/item_images/2793_Fikgo.jpg?...","gcsUri":"gs://shoplater/item_images/2793_Fikgo.jpg"}]}]}],"count":"3"}`

#### Get Collection Items By ID:
---
>GET https://api.shoplater.app/api/v1/collections/{COLLECTION_ID}/items
>_Params --> `?skip={SK}&take={TAKE}`_

-   SKIP --> Amount of elements to skip
-   TAKE --> Number of elements to return
-   Returns: `[SKIP, SKIP+TAKE]` elements if SKIP and TAKE present

##### Response `{SK: 0, TAKE: 6}`:
---
`{"data":[{"itemExistsInUserCollection":true,"id":522,"collectionId":420,"userId":"5N8HqK8ZJMVvii8viBTKaqPdy7M2","lastUpdated":"2023-11-04T17:47:42.095Z","creation":"2023-11-04T17:47:42.095Z","itemId":2799,"item":[{"id":2799,"userId":null,"storeId":51,"url":"https://www.ikea.com/pt/pt/p/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura-s89421969/","available":true,"currency":null,"title":"BESTÅ Armário parede c/2portas, preto-castanho Björköviken/castanho chapa de carvalho c/velatura, 60x22x128 cm - IKEA","description":null,"creation":"2023-11-04T17:47:39.035348","lastUpdated":"2023-11-04T17:47:51.89611","favicon":"https://icons.duckduckgo.com/ip3/ikea.com.ico","image":"https://www.ikea.com/pt/pt/images/products/besta-armario-parede-c-2portas-preto-castanho-bjoerkoeviken-castanho-chapa-de-carvalho-c-velatura__1024316_pe833495_s5.jpg","imageDimensions":null,"imageType":null,"imageInternal":null,"imageReference":null,"label":null,"lastUpdatedPrice":null,"owner":"ikea.com","ownerName":null,"productName":null,"relatedUrls":null,"safeSearch":null,"scrapperGenerated":true,"type":null,"performance":{"pageLoad":7705,"cloudinaryFailedImg":903},"imageInternalAlt":null}]},...]}],"count":"3"}`

#### Warning #1:
---
**IMPORTANT:** Private Collections are susceptible of being accessed and matched to a specific User.

By specifying arbitrary collection number and looping through them it is possible, with no authentication, to not only access the collection contents and all its items, but also, via the leaked User ID in the Collection's API response, correlate it to a specific user using the User Account API with the given ID, therefore attaching to it a username, an email, a photo, interests, and so on.

There are two problems here. First, the collections API shouldn't leak private collections to anyone else other than the owner (let alone someone not authenticated). Second, the User Account API probably shouldn't allow for any User ID to be queried and show that much information for accounts that don't even have the option to keep that information public or private (fixing only the first one will make it so that users with public collections will remain affected, specially if trending or if their products appear suggested in one's feed).

#### Warning #2:
---
Image URLs are not being correctly extracted, at least in a clean way, and its causing a malform string of HTML tag attributes which although seems to work reasonably well, causes a blank image on product #2793 from Amazon. Fortunately, the images stored in the Google Storage Bucket are correct. 

#### Get Collection Followers:
---
>GET https://api.shoplater.app/api/v1/collections/{COLLECTION_ID}/followers
>_Params --> `?skip={SK}&take={TAKE}`_

-   SKIP --> Amount of elements to skip
-   TAKE --> Number of elements to return
-   Returns: `[SKIP, SKIP+TAKE]` elements if SKIP and TAKE present

##### Response {SK: 0, TAKE: 6}:
---
`{"count":"0","data":null}`
-   `data --> followers`
-   `count --> length of data`

#### Get Trending Collections:
---
>GET https://api.shoplater.app/api/v1/trending/collections

##### Response Example:
---
`[{"user":{"id":"FlsU1LbX9dS5e7Ju2ZS7JyQAdVE3","username":"laurenn","photoURL":"https://firebasestorage.googleapis.com:443/v0/b/shoplater.appspot.com/o/user-avatars%2FFlsU1LbX9dS5e7Ju2ZS7JyQAdVE3.jpg?alt=media&token=724709bb-78ac-4a36-961d-47ea4e052244"},"id":357,"title":"Future Garden 🍃","products":[{"id":2237,"title":"Candeeiros de mesa modernos | Candeeiros mesa","images":["https://cdn.sklum.com/pt/wk/308930/...jpg","https://cdn.sklum.com/pt/wk/308941/...jpg","https://cdn.sklum.com/pt/wk/308919/...jpg"]},{"id":2211,"title":"Poltrona suspensa de jardim com base e almofada Anoop","images":["https://cdn.sklum.com/pt/wk/2266635/...jpg","https://cdn.sklum.com/pt/wk/1769025/...jpg","https://cdn.sklum.com/pt/wk/2266635/...jpg","https://cdn.sklum.com/pt/wk/1769025/...jpg"]},{"id":2216,"title":"Mesa de Jardim Redonda em Madeira de Acácia (Ø120 cm) Mura","images":["https://cdn.sklum.com/pt/wk/1857629/...jpg","https://cdn.sklum.com/pt/wk/2217085/...jpg"]},{"id":2218,"title":"Cadeira Suspensa de Jardim com Almofada Rufus","images":["https://cdn.sklum.com/pt/wk/2369269/...jpg","https://cdn.sklum.com/pt/wk/2369269/...jpg","https://cdn.sklum.com/pt/wk/2154387/...jpg","https://cdn.sklum.com/pt/wk/2154387/...jpg"]},{"id":2219,"title":"Luminária de piso LED sem fio externa Lovech ","images":["https://cdn.sklum.com/pt/wk/1819579/...jpg","https://cdn.sklum.com/pt/wk/2112040/...jpg","https://cdn.sklum.com/pt/wk/1819579/...jpg","https://cdn.sklum.com/pt/wk/2112040/...jpg"]}]},{"user": ...}]`

## **Analytics:**

### Sentry System
---
ShopLater uses https://sentry.twt-env.com/ as Interface.

POST https://sentry.twt-env.com/api/11/envelope/

#### Request Payload Example:
---
`{"sdk":{"name":"sentry.cocoa.react-native","version":"8.7.1","packages":{"name":"cocoapods:getsentry\/sentry.cocoa.react-native","version":"8.7.1"}},"sent_at":"2023-11-04T17:47:47.967Z"}`
`{"type":"session","length":309}`
`{"errors":0,"status":"exited","started":"2023-11-04T17:40:18.799Z","did":"4171D2C5-EE4C-4815-A99D-6A41407E502A","duration":35.63700008392334,"sid":"66528538-D6F8-48E9-9B1F-23BF0649E72F","attrs":{"release":"com.twistag.shoplater@2.0.30+301","environment":"PROD"},"timestamp":"2023-11-04T17:40:54.436Z","seq":2}`

### Branch.io --> Linking System & Analytics:
---
GET https://cdn.branch.io/sdk/uriskiplist_v3.json --> Deep Linking System

	Deep links take users directly to relevant content in your app or website, rather
	than a generic homepage. Once created, you can use a single deep link across all your
	channels: email, ads, QR codes, smart banners, in-app notifications, social media, and
	more.

POST https://api2.branch.io/v1/open/ (with System Information)

#### Request Payload Example:
---
`{"apple_testflight":false,"os_version":"17.0.1","ios_bundle_id":"com.twistag.shoplater","debug":false,"screen_dpi":2,"country":"PT","is_hardware_id_real":true,"opted_in_status":"denied","locale":"en_PT","anon_id":"67D4C43B-98E1-4B9F-A38A-45D639FD18C6","ad_tracking_enabled":false,"sdk":"ios2.2.0","uri_scheme":"shoplater","environment":"FULL_APP","metadata":{"skan_time_window":"5184000.000000"},"connection_type":"wifi","plugin_name":"ReactNative","previous_update_time":1697648424000,"retryNumber":0,"facebook_app_link_checked":false,"lastest_update_time":1697648424000,"local_ip":"100.123.48.210","brand":"Apple","update":0,"cd":{"pn":"com.twistag.shoplater","mv":"-1"},"plugin_version":"5.9.0","cpu_type":"16777228","screen_height":1624,"model":"iPhone11,8","randomized_device_token":"1248284944221110132","branch_key":"key_live_ie2rAqIhk8bMGHHOV3a3Ollgvumxm9Kn","user_agent":"Mozilla/5.0 (iPhone; CPU iPhone OS 17_0_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15E148","first_install_time":1698852534684,"hardware_id":"86A40448-E31D-41BB-8102-85AADC2298C6","os":"iOS","hardware_id_type":"vendor_id","latest_install_time":1698852534684,"randomized_bundle_token":"1248284944284090706","screen_width":750,"build":"21A340","language":"en","ios_vendor_id":"86A40448-E31D-41BB-8102-85AADC2298C6","app_version":"2.0.30","device_carrier":"--"}`

#### Response:
---
>**Uses:** https://shoplater.app.link
   To report user event information using: _?$randomized_bundle_token={BUNDLE_TOKEN}_

_Example Event Data:_
`"data":"{\"+clicked_branch_link\":false,\"+is_first_session\":false}"`

### OneSignal for Notifications
---
Endpoint: https://api.onesignal.com/
APP ID: "405a0d5e-d837-436c-bc0f-424fba1f5036"


### General Notes:
---
- Status Code 200/201 --> Success/Created
- Uses Cache: `True` --> HTTP 304 Not Modified;