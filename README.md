# أحلي مسا عليك يصاحبي
# 📄 API Documentation — DealsFoodApp
**Version:** 1.0.0  
**Base URL:** `https://sahlaapp.com/LEO/AdminApiLar/public/index.php/api/`  
**Image Base URL:** `https://sahlaapp.com/LEO/AdminApiLar/public/`

---

## 📌 Global Headers

كل الـ Requests يجب أن تتضمن الـ Headers التالية:

| Header | Value | Notes |
|--------|-------|-------|
| `Content-Type` | `application/json` | للـ JSON requests |
| `Accept` | `application/json` | دائماً |
| `lang` | `ar` / `en` | لغة الـ Response المطلوبة |

> [!NOTE]
> للـ Endpoints التي ترسل **Form Data** (تحميل صور)، لا يُرسَل `Content-Type` يدوياً — Dio يتولاه تلقائياً كـ `multipart/form-data`.

---

## 📌 Authentication Strategy

- معظم الـ Endpoints لا تستخدم Bearer Token في الـ Header.
- الـ User Identity يتم تمريره عبر `UserID` أو `access_token` في **Request Body**.
- الاستثناء: Reels endpoints تستخدم `access_token` في الـ Body.

---

## 📌 Standard Response Format

```json
{
  "status": "success",
  "message": "...",
  "data": { ... }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `status` | `string` | `"success"` أو `"error"` |
| `message` | `string` | رسالة توضيحية |
| `data` | `object \| array` | البيانات المرجعة |

---

---

# 🔐 1. Authentication

---

## 1.1 Register (Sign Up)

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/signUp` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `multipart/form-data` |
| **Feature** | شاشة التسجيل |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `first_name` | `string` | ✅ Required | الاسم الأول |
| `last_name` | `string` | ✅ Required | اسم العائلة |
| `email` | `string` | ✅ Required | البريد الإلكتروني |
| `PhoneKey` | `string` | ✅ Required | كود الدولة مثل `+966` |
| `phone_number` | `string` | ✅ Required | رقم الجوال بدون كود |
| `password` | `string` | ✅ Required | كلمة المرور |
| `latitude` | `string` | ⬜ Optional | خط العرض (default: `"0"`) |
| `longitude` | `string` | ⬜ Optional | خط الطول (default: `"0"`) |
| `fcm_token` | `string` | ⬜ Optional | Firebase Push Token |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "user": {
    "user_id": "123",
    "first_name": "أحمد",
    "last_name": "محمد",
    "email": "ahmed@example.com",
    "phone_number": "501234567",
    "country_code": "+966",
    "profile_pic": null,
    "access_token": "eyJ0eXAiOiJKV..."
  }
}
```

### Response Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user_id` | `string \| int` | ✅ | معرف المستخدم |
| `first_name` | `string` | ✅ | الاسم الأول |
| `last_name` | `string` | ✅ | اسم العائلة |
| `email` | `string` | ✅ | البريد الإلكتروني |
| `phone_number` | `string` | ⬜ | رقم الجوال |
| `country_code` | `string` | ⬜ | كود الدولة |
| `profile_pic` | `string \| null` | ⬜ | رابط الصورة الشخصية |
| `access_token` | `string` | ✅ | Token للعمليات المستقبلية |

### Error Responses

| Status Code | Description |
|-------------|-------------|
| `200` | نجح التسجيل (`status: "success"`) |
| `200` | فشل التسجيل (`status: "error"`, `message` يحتوي السبب) |
| `422` | بيانات غير صحيحة |
| `500` | خطأ في الخادم |

---

## 1.2 Login

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/login` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `multipart/form-data` |
| **Feature** | شاشة تسجيل الدخول |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `PhoneKey` | `string` | ✅ Required | كود الدولة مثل `+966` |
| `phone_number` | `string` | ✅ Required | رقم الجوال |
| `password` | `string` | ✅ Required | كلمة المرور |
| `fcm_token` | `string` | ⬜ Optional | Firebase Push Token |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "user": {
    "user_id": "123",
    "first_name": "أحمد",
    "last_name": "محمد",
    "email": "ahmed@example.com",
    "phone_number": "501234567",
    "country_code": "+966",
    "profile_pic": "Photo/users/abc.jpg",
    "access_token": "eyJ0eXAiOiJKV..."
  }
}
```

### Error Responses

| Status Code | Description |
|-------------|-------------|
| `200` | نجح الدخول (`status: "success"`) |
| `200` | بيانات خاطئة (`status: "error"`) |
| `401` | غير مصرح |
| `500` | خطأ في الخادم |

---

## 1.3 Google Login (& Apple Login)

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/googleLogin` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | تسجيل الدخول بـ Google / Apple |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id_token` | `string` | ✅ Required | الـ OAuth Token من Google/Apple |
| `email` | `string` | ✅ Required | البريد الإلكتروني |
| `first_name` | `string` | ✅ Required | الاسم الأول |
| `last_name` | `string` | ✅ Required | اسم العائلة |
| `profile_pic` | `string` | ⬜ Optional | رابط صورة الـ Google profile |
| `fcm_token` | `string` | ⬜ Optional | Firebase Push Token |
| `provider` | `string` | ⬜ Optional | `"apple"` للدخول بـ Apple (فارغ لـ Google) |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "user": {
    "user_id": "456",
    "first_name": "Sara",
    "last_name": "Ali",
    "email": "sara@gmail.com",
    "access_token": "eyJ0eXAiOiJKV..."
  }
}
```

> [!NOTE]
> الـ Backend قد يرجع `user` أو `data` كـ key للمستخدم — كلاهما مقبول.

### Error Responses

| Status Code | Description |
|-------------|-------------|
| `200` | نجاح (`status: "success"`) |
| `200` | فشل (`status: "error"`) |
| `401` | الـ Token غير صالح |
| `500` | خطأ في الخادم |

---

## 1.4 Send OTP

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/mobileOTP` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `multipart/form-data` |
| **Feature** | التحقق من رقم الجوال، نسيت كلمة المرور |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `phone_number` | `string` | ✅ Required | رقم الجوال |
| `PhoneKey` | `string` | ✅ Required | كود الدولة |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": "1234"
}
```

> [!IMPORTANT]
> الـ Server يرجع الـ OTP code مباشرةً في حقل `data` — للـ Frontend التحقق منه محلياً مع المستخدم.

### Error Responses

| Status Code | Description |
|-------------|-------------|
| `200` | تم إرسال الـ OTP (`status: "success"`, `data` يحتوي الكود) |
| `200` | فشل (`status: "error"`) |
| `500` | خطأ في الخادم |

---

## 1.5 Check Phone

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/checkPhone` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `multipart/form-data` |
| **Feature** | التحقق من وجود الرقم قبل التسجيل |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `phone_number` | `string` | ✅ Required | رقم الجوال |
| `PhoneKey` | `string` | ✅ Required | كود الدولة |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

| Status Code | Description |
|-------------|-------------|
| `200` | الرقم موجود/مقبول (`status: "success"`) |
| `200` | الرقم غير موجود أو مُستخدم (`status: "error"`) |

---

---

# 🏠 2. Home

---

## 2.1 Get Sliders (Home Banners)

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getSliders` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` (body فارغ) |
| **Feature** | الشرائح الإعلانية في الهوم |
| **Pagination** | ❌ لا يحتاج |
| **Caching** | ✅ يُخزَّن مؤقتاً |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "SlidersID": "1",
      "SlidersImage": "Photo/sliders/banner1.jpg",
      "TextAr": "عروض رمضان",
      "TextEN": "Ramadan Offers",
      "HolderType": "SHOP",
      "HolderValue": "42",
      "Type": "NORMAL",
      "EndDate": "2026-12-31"
    }
  ]
}
```

### Response Fields — Slider Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `SlidersID` | `string` | ✅ | معرف الشريحة |
| `SlidersImage` | `string` | ✅ | مسار الصورة (نسبي أو مطلق) |
| `TextAr` | `string` | ✅ | النص العربي |
| `TextEN` | `string` | ✅ | النص الإنجليزي |
| `HolderType` | `string` | ✅ | نوع الإجراء: `NO` \| `SHOP` \| `OFFER` \| `LINK` |
| `HolderValue` | `string` | ⬜ | القيمة المرتبطة (shop_id / offer_id / URL) |
| `Type` | `string` | ✅ | نوع الشريحة: `NORMAL` \| `AD` |
| `EndDate` | `string` | ⬜ | تاريخ الانتهاء أو `"-"` للدائمة |

---

## 2.2 Get Main Categories

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getMainCategories` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` (body فارغ) |
| **Feature** | التصنيفات الرئيسية في الهوم |
| **Pagination** | ❌ لا يحتاج |
| **Caching** | ✅ يُخزَّن مؤقتاً |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "category_id": "1",
      "parent_id": null,
      "name_ar": "مطاعم",
      "name_en": "Restaurants",
      "category_image": "Photo/categories/rest.jpg"
    }
  ]
}
```

### Response Fields — Category Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `category_id` | `string` | ✅ | معرف التصنيف |
| `parent_id` | `string \| null` | ⬜ | معرف التصنيف الأب |
| `name_ar` | `string` | ✅ | الاسم بالعربي |
| `name_en` | `string` | ✅ | الاسم بالإنجليزي |
| `category_image` | `string` | ✅ | مسار صورة التصنيف |

---

## 2.3 Get Offers

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `User/getOffers` |
| **Auth** | ❌ لا يحتاج |
| **Feature** | قسم العروض في الهوم |
| **Pagination** | ⚠️ مقترح (راجع ملاحظة Pagination) |
| **Caching** | ✅ يُخزَّن مؤقتاً |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "OffersID": "10",
      "TitleAr": "وجبة عائلية",
      "TitleEn": "Family Meal",
      "DescAr": "وجبة كاملة للعائلة",
      "DescEn": "Complete family meal",
      "Status": "ACTIVE",
      "Color": "#fb7f33",
      "icon": "Photo/offers/family.jpg",
      "vendor_id": "42",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel Restaurant",
      "vendor_logo": "Photo/vendors/logo.jpg"
    }
  ]
}
```

### Response Fields — Offer Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `OffersID` | `string` | ✅ | معرف العرض |
| `TitleAr` | `string` | ✅ | عنوان العرض بالعربي |
| `TitleEn` | `string` | ✅ | عنوان العرض بالإنجليزي |
| `DescAr` | `string` | ✅ | وصف العرض بالعربي |
| `DescEn` | `string` | ✅ | وصف العرض بالإنجليزي |
| `Status` | `string` | ✅ | حالة العرض: `ACTIVE` \| `INACTIVE` |
| `Color` | `string` | ✅ | لون كارد العرض (Hex) |
| `icon` | `string` | ✅ | مسار صورة العرض |
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `vendor_name_ar` | `string` | ✅ | اسم المتجر بالعربي |
| `vendor_name_en` | `string` | ✅ | اسم المتجر بالإنجليزي |
| `vendor_logo` | `string` | ✅ | مسار شعار المتجر |

---

## 2.4 Get Best Shops

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `User/getBestShops` |
| **Auth** | ❌ لا يحتاج |
| **Feature** | أفضل المتاجر في الهوم |
| **Pagination** | ⚠️ مقترح |
| **Caching** | ✅ يُخزَّن مؤقتاً |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lat` | `string` | ⬜ Optional | خط العرض للمستخدم |
| `long` | `string` | ⬜ Optional | خط الطول للمستخدم |
| `lng` | `string` | ⬜ Optional | نسخة بديلة من `long` (يُرسَل كلاهما) |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "vendor_id": "42",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "vendor_logo": "Photo/vendors/logo.jpg",
      "vendor_cover": "Photo/vendors/cover.jpg",
      "rating": "4.5",
      "delivery_time_min": 30,
      "delivery_fee": "15.00",
      "is_open": 1,
      "is_busy": 0,
      "review_count": 120,
      "phone_number": "0501234567",
      "email": "info@nakheel.com",
      "latitude": "24.7136",
      "longitude": "46.6753",
      "address_description": "الرياض، حي النخيل",
      "vendor_status": "ACTIVE",
      "distance": 2.5,
      "FAV": "NO"
    }
  ]
}
```

### Response Fields — Shop Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `vendor_name_ar` | `string` | ✅ | اسم المتجر بالعربي |
| `vendor_name_en` | `string` | ✅ | اسم المتجر بالإنجليزي |
| `vendor_logo` | `string` | ✅ | مسار الشعار |
| `vendor_cover` | `string` | ⬜ | مسار صورة الغلاف |
| `rating` | `string` | ✅ | متوسط التقييم |
| `delivery_time_min` | `integer \| null` | ⬜ | وقت التوصيل بالدقائق |
| `delivery_fee` | `string` | ✅ | رسوم التوصيل |
| `is_open` | `integer` | ✅ | `1` = مفتوح، `0` = مغلق |
| `is_busy` | `integer` | ✅ | `1` = مشغول |
| `review_count` | `integer` | ✅ | عدد التقييمات |
| `phone_number` | `string` | ⬜ | رقم الهاتف |
| `email` | `string` | ⬜ | البريد الإلكتروني |
| `latitude` | `string` | ⬜ | خط العرض للمتجر |
| `longitude` | `string` | ⬜ | خط الطول للمتجر |
| `address_description` | `string` | ⬜ | وصف العنوان |
| `vendor_status` | `string` | ✅ | حالة المتجر |
| `distance` | `float \| null` | ⬜ | المسافة بالكيلومتر |
| `FAV` | `string` | ⬜ | `"YES"` إذا مُضاف للمفضلة |

---

## 2.5 Get Best Products

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `User/getBestProducts` |
| **Auth** | ❌ لا يحتاج |
| **Feature** | أفضل المنتجات في الهوم |
| **Pagination** | ⚠️ مقترح |
| **Caching** | ✅ يُخزَّن مؤقتاً |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "product_id": "101",
      "name_ar": "برجر دجاج",
      "name_en": "Chicken Burger",
      "description_ar": "برجر دجاج مقرمش",
      "description_en": "Crispy chicken burger",
      "base_price": "35.00",
      "sale_price": "25.00",
      "is_on_sale": 1,
      "image_url": "Photo/products/burger.jpg",
      "vendor_id": "42",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel"
    }
  ]
}
```

### Response Fields — Product Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | `string` | ✅ | معرف المنتج |
| `name_ar` | `string` | ✅ | اسم المنتج بالعربي |
| `name_en` | `string` | ✅ | اسم المنتج بالإنجليزي |
| `description_ar` | `string` | ⬜ | وصف المنتج بالعربي |
| `description_en` | `string` | ⬜ | وصف المنتج بالإنجليزي |
| `base_price` | `string` | ✅ | السعر الأصلي |
| `sale_price` | `string` | ⬜ | سعر العرض |
| `is_on_sale` | `integer` | ✅ | `1` = في عرض، `0` = لا |
| `image_url` | `string` | ✅ | مسار صورة المنتج |
| `vendor_id` | `string` | ✅ | معرف المتجر المالك |
| `vendor_name_ar` | `string` | ⬜ | اسم المتجر بالعربي |
| `vendor_name_en` | `string` | ⬜ | اسم المتجر بالإنجليزي |

---

---

# 🏪 3. Shop Details

---

## 3.1 Get Shop Info

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getShopDetails` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | شاشة تفاصيل المتجر |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_id` | `string` | ✅ Required | معرف المتجر |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": {
    "vendor_id": "42",
    "vendor_name_ar": "مطعم النخيل",
    "vendor_name_en": "Al Nakheel",
    "vendor_logo": "Photo/vendors/logo.jpg",
    "vendor_cover": "Photo/vendors/cover.jpg",
    "rating": "4.5",
    "delivery_time_min": 30,
    "delivery_fee": "15.00",
    "is_open": 1,
    "is_busy": 0,
    "review_count": 120,
    "FAV": "YES"
  }
}
```

> [!NOTE]
> نفس هيكل الـ Shop Object الموضح في **2.4**

---

## 3.2 Get Shop Products

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getShopProducts` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | قائمة منتجات المتجر مجمعة بالأقسام |
| **Pagination** | ❌ لا يحتاج (يرجع كل المنتجات) |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_id` | `string` | ✅ Required | معرف المتجر |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "section_id": "5",
      "name_ar": "الوجبات الرئيسية",
      "name_en": "Main Meals",
      "sort_order": 1,
      "Prod": [
        {
          "product_id": "101",
          "name_ar": "برجر دجاج",
          "name_en": "Chicken Burger",
          "base_price": "35.00",
          "sale_price": "25.00",
          "is_on_sale": 1,
          "image_url": "Photo/products/burger.jpg"
        }
      ]
    }
  ]
}
```

### Response Fields — Section Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `section_id` | `string` | ✅ | معرف القسم |
| `name_ar` | `string` | ✅ | اسم القسم بالعربي |
| `name_en` | `string` | ✅ | اسم القسم بالإنجليزي |
| `sort_order` | `integer` | ✅ | ترتيب العرض |
| `Prod` | `array` | ✅ | قائمة المنتجات داخل القسم |

---

## 3.3 Get Shop Reviews

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getShopReviews` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | تقييمات المتجر في شاشة التفاصيل |
| **Pagination** | ⚠️ مقترح |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ShopID` | `string` | ✅ Required | معرف المتجر |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "Rate": 5,
      "Text": "خدمة ممتازة",
      "CreatedAtRatings": "2026-01-15",
      "order_id": 789,
      "first_name": "محمد",
      "last_name": "علي",
      "profile_pic": "Photo/users/pic.jpg"
    }
  ]
}
```

### Response Fields — Review Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Rate` | `integer` | ✅ | التقييم (1–5) |
| `Text` | `string` | ⬜ | نص التعليق |
| `CreatedAtRatings` | `string` | ✅ | تاريخ التقييم |
| `order_id` | `integer` | ✅ | معرف الطلب المرتبط |
| `first_name` | `string` | ✅ | اسم المستخدم |
| `last_name` | `string` | ⬜ | اسم العائلة |
| `profile_pic` | `string` | ⬜ | صورة المستخدم |

---

## 3.4 Add to Favorite

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/addToFavorite` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | إضافة متجر للمفضلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `ShopID` | `string` | ✅ Required | معرف المتجر |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "message": "Shop added to favorites"
}
```

---

## 3.5 Remove from Favorite

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/removeFromFavorite` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | إزالة متجر من المفضلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `ShopID` | `string` | ✅ Required | معرف المتجر |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "message": "Shop removed from favorites"
}
```

---

## 3.6 Get Favorite Shops

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getFavorites` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | شاشة المفضلة في البروفايل |
| **Pagination** | ✅ يدعم Pagination |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `page` | `integer` | ⬜ Optional | رقم الصفحة (يبدأ من `0`) |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    { ... }
  ]
}
```

> [!NOTE]
> البيانات تتبع نفس هيكل Shop Object في **2.4**

---

---

# 📦 4. Product Details

---

## 4.1 Get Product Images

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `User/getProductImages` |
| **Auth** | ❌ لا يحتاج |
| **Feature** | معرض صور المنتج |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `product_id` | `string` | ✅ Required | معرف المنتج |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "images": [
    "Photo/products/burger1.jpg",
    "Photo/products/burger2.jpg"
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `images` | `array<string>` | ✅ | قائمة مسارات الصور |

---

## 4.2 Get Product Extras

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getProductExtras` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | إضافات المنتج عند الطلب |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | `string` | ✅ Required | معرف المنتج |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "ProductExtraCatID": "3",
      "ProductExtraCatName": "الإضافات",
      "ProductExtraCatNameEn": "Extras",
      "LessChoice": "0",
      "MostChoice": "2",
      "Tarteb": "1",
      "values": [
        {
          "ProductExtraValID": "10",
          "ProductExtraValName": "جبنة إضافية",
          "ProductExtraValNameEn": "Extra Cheese",
          "ProductExtraValPrice": "5.00"
        }
      ]
    }
  ]
}
```

### Response Fields — Extra Category Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ProductExtraCatID` | `string` | ✅ | معرف فئة الإضافة |
| `ProductExtraCatName` | `string` | ✅ | اسم فئة الإضافة بالعربي |
| `ProductExtraCatNameEn` | `string` | ✅ | اسم فئة الإضافة بالإنجليزي |
| `LessChoice` | `string` | ✅ | الحد الأدنى للاختيارات (0 = اختياري) |
| `MostChoice` | `string` | ✅ | الحد الأقصى (0 = غير محدود) |
| `Tarteb` | `string` | ✅ | ترتيب العرض |
| `values` | `array` | ✅ | قائمة عناصر الإضافة |

### Response Fields — Extra Item Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ProductExtraValID` | `string` | ✅ | معرف العنصر |
| `ProductExtraValName` | `string` | ✅ | الاسم بالعربي |
| `ProductExtraValNameEn` | `string` | ✅ | الاسم بالإنجليزي |
| `ProductExtraValPrice` | `string` | ✅ | سعر الإضافة |

---

## 4.3 Get Similar Products

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getSimilarProducts` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | منتجات مشابهة في شاشة تفاصيل المنتج |
| **Pagination** | ❌ لا يحتاج |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | `string` | ✅ Required | معرف المنتج |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    { "product_id": "102", "name_ar": "برجر لحم", ... }
  ]
}
```

> [!NOTE]
> نفس هيكل Product Object في **2.5**

---

---

# 🛒 5. Cart

---

## 5.1 Add to Cart

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/addToCart` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | إضافة منتج للسلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `product_id` | `string` | ✅ Required | معرف المنتج |
| `Qty` | `integer` | ✅ Required | الكمية |
| `vendor_id` | `string` | ✅ Required | معرف المتجر |
| `Extra` | `array<string>` | ⬜ Optional | قائمة معرفات الإضافات المختارة |

### Request Example

```json
{
  "UserID": "123",
  "product_id": "101",
  "Qty": 2,
  "vendor_id": "42",
  "Extra": ["10", "11"]
}
```

### Expected Response `200 OK`

```json
{
  "status": "success",
  "message": "Item added to cart"
}
```

### Error Responses

| Status Code | Description |
|-------------|-------------|
| `200` | نجح (`status: "success"`) |
| `200` | فشل (`status: "error"`) |
| `401` | غير مصرح |

---

## 5.2 Get My Cart

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getMyCart` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | شاشة السلة |
| **Pagination** | ❌ لا يحتاج |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "cart_item_id": "501",
      "product_id": "101",
      "name_ar": "برجر دجاج",
      "name_en": "Chicken Burger",
      "quantity": 2,
      "base_price": "35.00",
      "sale_price": "25.00",
      "is_on_sale": 1,
      "image_url": "Photo/products/burger.jpg",
      "vendor_id": "42",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "extra_text_ar": "جبنة إضافية",
      "extra_text_en": "Extra Cheese"
    }
  ]
}
```

### Response Fields — Cart Item Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cart_item_id` | `string` | ✅ | معرف عنصر السلة |
| `product_id` | `string` | ✅ | معرف المنتج |
| `name_ar` | `string` | ✅ | اسم المنتج بالعربي |
| `name_en` | `string` | ✅ | اسم المنتج بالإنجليزي |
| `quantity` | `integer` | ✅ | الكمية |
| `base_price` | `string` | ✅ | السعر الأصلي |
| `sale_price` | `string` | ⬜ | سعر العرض |
| `is_on_sale` | `integer` | ✅ | `1` = في عرض |
| `image_url` | `string` | ✅ | صورة المنتج |
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `vendor_name_ar` | `string` | ⬜ | اسم المتجر |
| `extra_text_ar` | `string` | ⬜ | نص الإضافات بالعربي |
| `extra_text_en` | `string` | ⬜ | نص الإضافات بالإنجليزي |

---

## 5.3 Get Cart Count

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getCartCount` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | بادج عدد عناصر السلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": "3"
}
```

> [!NOTE]
> الـ count قد يكون في `data` أو `count`، يجب دعم كلا الحالتين.

---

## 5.4 Delete Cart Item

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/deleteCartItem` |
| **Auth** | ✅ يحتاج `cart_item_id` |
| **Format** | `application/json` |
| **Feature** | حذف عنصر من السلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cart_item_id` | `string` | ✅ Required | معرف عنصر السلة |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "message": "Item removed from cart"
}
```

---

## 5.5 Update Cart Item Quantity

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/updateQuantity` |
| **Auth** | ✅ يحتاج `cart_item_id` |
| **Format** | `application/json` |
| **Feature** | تعديل كمية عنصر في السلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cart_item_id` | `string` | ✅ Required | معرف عنصر السلة |
| `quantity` | `integer` | ✅ Required | الكمية الجديدة |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 5.6 Calculate Delivery

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/calculateDelivery` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | حساب رسوم التوصيل في السلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_id` | `string` | ✅ Required | معرف المتجر |
| `lat` | `string` | ✅ Required | خط عرض العنوان المختار |
| `lng` | `string` | ✅ Required | خط طول العنوان المختار |

### Expected Response `200 OK`

```json
{
  "delivery_fee": "15.00",
  "branch_id": "7",
  "branch_name": "فرع الرياض",
  "distance": 3.5
}
```

### Response Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `delivery_fee` | `string` | ✅ | رسوم التوصيل |
| `branch_id` | `string` | ✅ | معرف أقرب فرع |
| `branch_name` | `string` | ✅ | اسم الفرع |
| `distance` | `float` | ✅ | المسافة بالكيلومتر |

> [!NOTE]
> قد يكون الحقل `delivery_fee` أو `deliveryFee` أو `DeliveryFee` أو `DelvPrice` أو `fee` — ادعم جميع الاحتمالات.

---

## 5.7 Place Order

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/placeOrder` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تأكيد الطلب من السلة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `MarketShopID` | `string` | ✅ Required | معرف المتجر |
| `SubTotal` | `float` | ✅ Required | المجموع قبل التوصيل |
| `DeliveryFee` | `float` | ✅ Required | رسوم التوصيل |
| `Total` | `float` | ✅ Required | المجموع الكلي |
| `AddressesID` | `string` | ✅ Required | معرف العنوان المختار |
| `Method` | `string` | ✅ Required | طريقة الدفع: `CASH` \| `ONLINE` |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "order_id": "789"
}
```

> [!NOTE]
> معرف الطلب قد يكون في `order_id` أو `OrderID` أو `id`.

---

---

# 📋 6. Orders

---

## 6.1 Get My Orders

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getMyOrders` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | شاشة طلباتي |
| **Pagination** | ⚠️ مقترح |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "order_id": "789",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "vendor_logo": "Photo/vendors/logo.jpg",
      "vendor_id": "42",
      "order_status": "DELIVERED",
      "total_amount": "65.00",
      "sub_total": "50.00",
      "delivery_fee": "15.00",
      "item_count": 3,
      "currency": "SAR",
      "payment_method": "CASH",
      "payment_status": "PAID",
      "created_at": "2026-08-01 14:30:00",
      "DriverName": "خالد",
      "DriverPhone": "0501111111",
      "DriverTracking": "https://...",
      "Rating": null,
      "Rated": "NO"
    }
  ]
}
```

### Response Fields — Order Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | `string` | ✅ | معرف الطلب |
| `vendor_name_ar` | `string` | ✅ | اسم المتجر بالعربي |
| `vendor_name_en` | `string` | ✅ | اسم المتجر بالإنجليزي |
| `vendor_logo` | `string` | ✅ | شعار المتجر |
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `order_status` | `string` | ✅ | حالة الطلب (`PENDING`, `CONFIRMED`, `DELIVERED`, etc.) |
| `total_amount` | `string` | ✅ | المبلغ الكلي |
| `sub_total` | `string` | ✅ | مجموع المنتجات |
| `delivery_fee` | `string` | ✅ | رسوم التوصيل |
| `item_count` | `integer` | ✅ | عدد العناصر |
| `currency` | `string` | ✅ | العملة |
| `payment_method` | `string` | ✅ | طريقة الدفع |
| `payment_status` | `string` | ✅ | حالة الدفع |
| `created_at` | `string` | ✅ | تاريخ الطلب |
| `DriverName` | `string` | ⬜ | اسم السائق |
| `DriverPhone` | `string` | ⬜ | هاتف السائق |
| `DriverTracking` | `string` | ⬜ | رابط تتبع السائق |
| `Rating` | `integer \| null` | ⬜ | التقييم المعطى |
| `Rated` | `string` | ✅ | `"YES"` إذا تم التقييم |

---

## 6.2 Get Order Details

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getOneOrder` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تفاصيل طلب محدد |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | `string` | ✅ Required | معرف الطلب |
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "order_item_id": 1,
      "order_id": 789,
      "product_id": 101,
      "quantity": 2,
      "unit_price": 25.00,
      "total_price": 50.00,
      "created_at": "2026-08-01",
      "Extra": "0",
      "name_ar": "برجر دجاج",
      "name_en": "Chicken Burger",
      "image_url": "Photo/products/burger.jpg",
      "extra_text_ar": "",
      "extra_text_en": ""
    }
  ]
}
```

### Response Fields — Order Detail Item Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_item_id` | `integer` | ✅ | معرف السطر في الطلب |
| `order_id` | `integer` | ✅ | معرف الطلب |
| `product_id` | `integer` | ✅ | معرف المنتج |
| `quantity` | `integer` | ✅ | الكمية |
| `unit_price` | `float` | ✅ | سعر الوحدة |
| `total_price` | `float` | ✅ | المجموع لهذا الصنف |
| `name_ar` | `string` | ✅ | اسم المنتج بالعربي |
| `name_en` | `string` | ✅ | اسم المنتج بالإنجليزي |
| `image_url` | `string` | ✅ | صورة المنتج |
| `extra_text_ar` | `string` | ⬜ | نص الإضافات بالعربي |
| `extra_text_en` | `string` | ⬜ | نص الإضافات بالإنجليزي |

---

## 6.3 Rate Shop

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/rateShop` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تقييم المتجر بعد التوصيل |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `ShopID` | `string` | ✅ Required | معرف المتجر |
| `Rate` | `string` | ✅ Required | التقييم (1-5) |
| `Comment` | `string` | ✅ Required | التعليق (يمكن إرساله فارغاً) |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 6.4 Rate Order

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/rateOrder` |
| **Auth** | ❌ (الطلب يُعرِّف المستخدم) |
| **Format** | `application/json` |
| **Feature** | تقييم شامل للطلب (المتجر + التوصيل) |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | `string` | ✅ Required | معرف الطلب |
| `rating_vendor` | `string` | ✅ Required | تقييم المتجر (1-5) |
| `rating_delivery` | `string` | ✅ Required | تقييم التوصيل (1-5) |
| `rating_text` | `string` | ✅ Required | تعليق نصي |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

---

# 👤 7. Profile

---

## 7.1 Get Profile

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getProfile` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | شاشة البروفايل |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "user": {
    "user_id": "123",
    "first_name": "أحمد",
    "last_name": "محمد",
    "email": "ahmed@example.com",
    "country_code": "+966",
    "phone_number": "501234567",
    "profile_pic": "Photo/users/pic.jpg",
    "fcm_token": "fcm_abc...",
    "access_token": "eyJ0eXAiOiJKV...",
    "auth_method": "PHONE",
    "latitude": "24.7136",
    "longitude": "46.6753",
    "wallet_balance": "0.00",
    "is_verified": "1",
    "app_language": "ar",
    "is_active": "1",
    "created_at": "2026-01-01",
    "updated_at": "2026-08-01"
  }
}
```

### Response Fields — Profile Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user_id` | `string` | ✅ | معرف المستخدم |
| `first_name` | `string` | ✅ | الاسم الأول |
| `last_name` | `string` | ✅ | اسم العائلة |
| `email` | `string` | ✅ | البريد الإلكتروني |
| `country_code` | `string` | ✅ | كود الدولة |
| `phone_number` | `string` | ✅ | رقم الجوال |
| `profile_pic` | `string \| null` | ⬜ | رابط الصورة الشخصية |
| `fcm_token` | `string \| null` | ⬜ | Firebase Token |
| `access_token` | `string` | ✅ | الـ Access Token |
| `auth_method` | `string` | ✅ | طريقة الدخول: `PHONE`, `GOOGLE`, `APPLE` |
| `latitude` | `string` | ⬜ | آخر موقع مسجل |
| `longitude` | `string` | ⬜ | آخر موقع مسجل |
| `wallet_balance` | `string` | ✅ | رصيد المحفظة |
| `is_verified` | `string` | ✅ | `"1"` إذا تم التحقق |
| `app_language` | `string` | ✅ | `"ar"` أو `"en"` |
| `is_active` | `string` | ✅ | `"1"` إذا الحساب نشط |
| `created_at` | `string` | ✅ | تاريخ الإنشاء |
| `updated_at` | `string` | ✅ | تاريخ آخر تعديل |

---

## 7.2 Update Profile (Text Only)

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/updateProfile` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تعديل بيانات البروفايل |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `first_name` | `string` | ✅ Required | الاسم الأول |
| `last_name` | `string` | ✅ Required | اسم العائلة |
| `email` | `string` | ✅ Required | البريد الإلكتروني |
| `country_code` | `string` | ✅ Required | كود الدولة |
| `phone_number` | `string` | ✅ Required | رقم الجوال |

### Expected Response `200 OK`

```json
{
  "success": true,
  "message": "Profile updated successfully"
}
```

---

## 7.3 Update Profile with Image

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/updateProfile` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `multipart/form-data` |
| **Feature** | تغيير صورة البروفايل مع البيانات |

### Request Fields (Form Data)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `first_name` | `string` | ✅ Required | الاسم الأول |
| `last_name` | `string` | ✅ Required | اسم العائلة |
| `email` | `string` | ✅ Required | البريد الإلكتروني |
| `country_code` | `string` | ✅ Required | كود الدولة |
| `phone_number` | `string` | ✅ Required | رقم الجوال |
| `profile_pic` | `file` | ✅ Required | ملف الصورة |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "user": {
    "profile_pic": "Photo/users/new_pic.jpg"
  }
}
```

---

## 7.4 Change Password

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/changePassword` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تغيير كلمة المرور من الإعدادات |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `new_password` | `string` | ✅ Required | كلمة المرور الجديدة |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 7.5 Delete Account

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/deleteAccount` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | حذف الحساب نهائياً |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "success": true
}
```

---

## 7.6 Contact Us

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/contactUs` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | نموذج التواصل معنا |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | ✅ Required | اسم المرسل |
| `email` | `string` | ✅ Required | البريد الإلكتروني |
| `phone` | `string` | ✅ Required | رقم الهاتف |
| `message` | `string` | ✅ Required | الرسالة |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

---

# 📍 8. Addresses

---

## 8.1 Get Addresses

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getAddresses` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | قائمة عناوين المستخدم |
| **Pagination** | ❌ لا يحتاج |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "address_id": "201",
      "address_label": "المنزل",
      "city": "الرياض",
      "cityAR": "الرياض",
      "cityEN": "Riyadh",
      "area": "النخيل",
      "street_name": "شارع الملك فهد",
      "building_number": "12",
      "floor_number": "3",
      "apartment_number": "5",
      "landmark": "بجوار مسجد النور",
      "latitude": "24.7136",
      "longitude": "46.6753",
      "DelvPrice": "15.00",
      "is_default": 1
    }
  ]
}
```

### Response Fields — Address Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `address_id` | `string` | ✅ | معرف العنوان |
| `address_label` | `string` | ✅ | تسمية العنوان (`المنزل`, `العمل`, إلخ) |
| `city` | `string` | ⬜ | المدينة |
| `cityAR` | `string` | ⬜ | المدينة بالعربي |
| `cityEN` | `string` | ⬜ | المدينة بالإنجليزي |
| `area` | `string` | ⬜ | الحي / المنطقة |
| `street_name` | `string` | ⬜ | اسم الشارع |
| `building_number` | `string` | ⬜ | رقم المبنى |
| `floor_number` | `string` | ⬜ | رقم الطابق |
| `apartment_number` | `string` | ⬜ | رقم الشقة |
| `landmark` | `string` | ⬜ | معلم قريب |
| `latitude` | `string` | ✅ | خط العرض |
| `longitude` | `string` | ✅ | خط الطول |
| `DelvPrice` | `string` | ⬜ | سعر التوصيل للعنوان |
| `is_default` | `integer` | ✅ | `1` = العنوان الافتراضي |

---

## 8.2 Add Address

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/addAddress` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `multipart/form-data` |
| **Feature** | إضافة عنوان جديد |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `address_label` | `string` | ✅ Required | تسمية العنوان |
| `city` | `string` | ✅ Required | المدينة |
| `latitude` | `string` | ✅ Required | خط العرض |
| `longitude` | `string` | ✅ Required | خط الطول |
| `street_name` | `string` | ⬜ Optional | اسم الشارع |
| `building_number` | `string` | ⬜ Optional | رقم المبنى |
| `floor_number` | `string` | ⬜ Optional | رقم الطابق |
| `apartment_number` | `string` | ⬜ Optional | رقم الشقة |
| `landmark` | `string` | ⬜ Optional | معلم قريب |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "message": "Address added successfully"
}
```

---

## 8.3 Delete Address

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/deleteAddress` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | حذف عنوان |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `AddressID` | `string` | ✅ Required | معرف العنوان |
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 8.4 Make Address Default

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/makeAddressDefault` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تعيين عنوان كعنوان افتراضي |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `AddressID` | `string` | ✅ Required | معرف العنوان |
| `UserID` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 8.5 Get Cities

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `User/getCities` |
| **Auth** | ❌ لا يحتاج |
| **Feature** | قائمة المدن المتاحة عند إضافة عنوان |
| **Pagination** | ❌ لا يحتاج |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "city_id": "1",
      "name_ar": "الرياض",
      "name_en": "Riyadh"
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `city_id` | `string` | ✅ | معرف المدينة |
| `name_ar` | `string` | ✅ | اسم المدينة بالعربي |
| `name_en` | `string` | ✅ | اسم المدينة بالإنجليزي |

---

---

# 🔔 9. Notifications

---

## 9.1 Get Notifications

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getNotifications` |
| **Auth** | ✅ يحتاج `userid` |
| **Format** | `application/json` |
| **Feature** | شاشة الإشعارات |
| **Pagination** | ⚠️ مقترح |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `userid` | `string` | ✅ Required | معرف المستخدم (لاحظ lowercase) |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "unread_count": "3",
  "notifications": [
    {
      "UserNotificationID": "501",
      "TextAR": "تم تأكيد طلبك",
      "TextEn": "Your order has been confirmed",
      "CreatedAtUserNotification": "2026-08-01 10:00:00",
      "SEEN": "UNSEEN",
      "Action": "ORDERS",
      "HolderID": "789"
    }
  ]
}
```

### Response Fields — Notification Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserNotificationID` | `string` | ✅ | معرف الإشعار |
| `TextAR` | `string` | ✅ | نص الإشعار بالعربي |
| `TextEn` | `string` | ✅ | نص الإشعار بالإنجليزي |
| `CreatedAtUserNotification` | `string` | ✅ | تاريخ الإشعار |
| `SEEN` | `string` | ✅ | `"SEEN"` أو `"UNSEEN"` |
| `Action` | `string` | ✅ | نوع الإجراء: `ORDERS` \| `SHOP` |
| `HolderID` | `string` | ✅ | معرف الكيان المرتبط (order_id / shop_id) |
| `unread_count` | `string` | ✅ | عدد الإشعارات غير المقروءة (في Root) |

---

## 9.2 Mark Notification Seen

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/markNotificationSeen` |
| **Auth** | ✅ يحتاج `UserID` |
| **Format** | `application/json` |
| **Feature** | تعليم إشعار كمقروء |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `UserID` | `string` | ✅ Required | معرف المستخدم |
| `notification_id` | `string` | ✅ Required | معرف الإشعار |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

---

# 🔍 10. Search

---

## 10.1 Search

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `search` |
| **Auth** | ❌ لا يحتاج |
| **Feature** | شاشة البحث — يبحث في المتاجر والمنتجات |
| **Pagination** | ⚠️ مقترح |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | `string` | ✅ Required | نص البحث |
| `lat` | `string` | ⬜ Optional | خط عرض المستخدم |
| `lng` | `string` | ⬜ Optional | خط طول المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "vendors": [
    {
      "vendor_id": "42",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "vendor_logo": "Photo/vendors/logo.jpg",
      "vendor_cover": "Photo/vendors/cover.jpg",
      "phone_number": "0501234567",
      "email": "info@nakheel.com",
      "address_description": "الرياض، حي النخيل"
    }
  ],
  "products": [
    {
      "product_id": "101",
      "name_ar": "برجر دجاج",
      "name_en": "Chicken Burger",
      "description_ar": "وصف بالعربي",
      "description_en": "Description in English",
      "base_price": "35.00",
      "sale_price": "25.00",
      "is_on_sale": 1,
      "image_url": "Photo/products/burger.jpg",
      "vendor_id": "42",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "vendor_logo": "Photo/vendors/logo.jpg"
    }
  ]
}
```

### Response Fields — Search Response

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendors` | `array<SearchVendor>` | ✅ | المتاجر المطابقة |
| `products` | `array<SearchProduct>` | ✅ | المنتجات المطابقة |

---

---

# 🎬 11. Social (Reels / Videos)

---

## 11.1 Get Reels

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getReels` |
| **Auth** | ✅ يحتاج `access_token` |
| **Format** | `application/json` |
| **Feature** | شاشة الـ Reels / الفيديوهات |
| **Pagination** | ⚠️ مقترح بشدة |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `access_token` | `string` | ✅ Required | الـ Access Token للمستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "VideoRealsID": "301",
      "vendor_id": "42",
      "Video": "Photo/reels/video1.mp4",
      "Status": "ACTIVE",
      "DescAr": "وصف الفيديو",
      "DescEn": "Video description",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "vendor_logo": "Photo/vendors/logo.jpg",
      "TotalLikes": "245",
      "IsLiked": "1",
      "TotalComments": "18"
    }
  ]
}
```

### Response Fields — Reel Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `VideoRealsID` | `string` | ✅ | معرف الـ Reel |
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `Video` | `string` | ✅ | مسار ملف الفيديو |
| `Status` | `string` | ✅ | `"ACTIVE"` \| `"INACTIVE"` |
| `DescAr` | `string` | ✅ | وصف بالعربي |
| `DescEn` | `string` | ✅ | وصف بالإنجليزي |
| `vendor_name_ar` | `string` | ✅ | اسم المتجر بالعربي |
| `vendor_name_en` | `string` | ✅ | اسم المتجر بالإنجليزي |
| `vendor_logo` | `string` | ✅ | شعار المتجر |
| `TotalLikes` | `string` | ✅ | إجمالي الإعجابات |
| `IsLiked` | `string` | ✅ | `"1"` إذا أعجب المستخدم |
| `TotalComments` | `string` | ✅ | إجمالي التعليقات |

---

## 11.2 Get Vendor Videos

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getVendorVideos` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | فيديوهات متجر محدد في شاشة تفاصيل المتجر |
| **Pagination** | ⚠️ مقترح |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_id` | `string` | ✅ Required | معرف المتجر |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [ { ... } ]
}
```

> [!NOTE]
> نفس هيكل Reel Object في **11.1**

---

## 11.3 Toggle Reel Like

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/toggleReelLike` |
| **Auth** | ✅ يحتاج `access_token` |
| **Format** | `application/json` |
| **Feature** | إعجاب / إلغاء إعجاب بـ Reel |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `video_id` | `integer` | ✅ Required | معرف الـ Reel |
| `access_token` | `string` | ✅ Required | الـ Access Token |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "is_liked": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `is_liked` | `boolean` | الحالة الجديدة بعد الـ Toggle |

---

## 11.4 Get Reel Comments

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getReelComments` |
| **Auth** | ✅ يحتاج `access_token` |
| **Format** | `application/json` |
| **Feature** | قسم التعليقات في الـ Reel |
| **Pagination** | ⚠️ مقترح |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `video_id` | `string` | ✅ Required | معرف الـ Reel |
| `access_token` | `string` | ✅ Required | الـ Access Token |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [
    {
      "comment_id": "1",
      "comment": "ممتاز!",
      "parent_id": "0",
      "user_name": "أحمد",
      "user_pic": "Photo/users/pic.jpg",
      "created_at": "2026-08-01"
    }
  ]
}
```

---

## 11.5 Add Reel Comment

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/addReelComment` |
| **Auth** | ✅ يحتاج `access_token` |
| **Format** | `application/json` |
| **Feature** | إضافة تعليق على Reel |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `video_id` | `string` | ✅ Required | معرف الـ Reel |
| `comment` | `string` | ✅ Required | نص التعليق |
| `parent_id` | `string` | ⬜ Optional | معرف التعليق الأب (للرد) — default: `"0"` |
| `access_token` | `string` | ✅ Required | الـ Access Token |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 11.6 Delete Reel Comment

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/deleteReelComment` |
| **Auth** | ✅ يحتاج `access_token` |
| **Format** | `application/json` |
| **Feature** | حذف تعليق من Reel |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `comment_id` | `string` | ✅ Required | معرف التعليق |
| `access_token` | `string` | ✅ Required | الـ Access Token |

### Expected Response `200 OK`

```json
{
  "status": "success"
}
```

---

## 11.7 Get Reel Favorites (User's Liked Reels)

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `users/{userId}/reels-favorites` |
| **Auth** | ✅ يحتاج `userId` في الـ URL |
| **Format** | `application/json` |
| **Feature** | الـ Reels التي أعجبت المستخدم في البروفايل |
| **Pagination** | ⚠️ مقترح |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `userId` | `string` | ✅ Required | معرف المستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": [ { ... } ]
}
```

> [!NOTE]
> نفس هيكل Reel Object في **11.1**

---

---

# 🎁 12. Offers Details

---

## 12.1 Get Offer Details

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getOfferDetails` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | شاشة تفاصيل العرض |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `offer_id` | `string` | ✅ Required | معرف العرض |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "offer": {
    "OffersID": "10",
    "TitleAr": "وجبة عائلية",
    "TitleEn": "Family Meal",
    "DescAr": "وصف بالعربي",
    "DescEn": "Description",
    "Status": "ACTIVE",
    "Color": "#fb7f33",
    "icon": "Photo/offers/family.jpg",
    "vendor_id": "42",
    "vendor_name_ar": "مطعم النخيل",
    "vendor_name_en": "Al Nakheel",
    "vendor_logo": "Photo/vendors/logo.jpg"
  },
  "products": [
    {
      "OfferPrice": "99.00",
      "quantity": "2",
      "product_id": "101",
      "name_ar": "برجر دجاج",
      "name_en": "Chicken Burger",
      "description_ar": "وصف",
      "description_en": "Description",
      "image_url": "Photo/products/burger.jpg",
      "vendor_name_ar": "مطعم النخيل",
      "vendor_name_en": "Al Nakheel",
      "vendor_logo": "Photo/vendors/logo.jpg"
    }
  ]
}
```

### Response Fields — Offer Products Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `OfferPrice` | `string` | ✅ | سعر العرض |
| `quantity` | `string` | ✅ | الكمية ضمن العرض |
| `product_id` | `string` | ✅ | معرف المنتج |
| `name_ar` | `string` | ✅ | اسم المنتج بالعربي |
| `name_en` | `string` | ✅ | اسم المنتج بالإنجليزي |
| `description_ar` | `string` | ⬜ | وصف المنتج بالعربي |
| `description_en` | `string` | ⬜ | وصف المنتج بالإنجليزي |
| `image_url` | `string` | ✅ | صورة المنتج |

---

## 12.2 Place Offer Order

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/placeOfferOrder` |
| **Auth** | ✅ يحتاج `access_token` |
| **Format** | `application/json` |
| **Feature** | تأكيد طلب العرض |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `offer_id` | `string` | ✅ Required | معرف العرض |
| `address_id` | `string` | ✅ Required | معرف العنوان |
| `delivery_fee` | `float` | ✅ Required | رسوم التوصيل |
| `access_token` | `string` | ✅ Required | الـ Access Token |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "order_id": "990"
}
```

> [!NOTE]
> معرف الطلب قد يكون في `order_id` أو `OrderID` أو `id`.

---

---

# 📁 13. Category Details

---

## 13.1 Get Category Details

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `User/getCategoryDetails` |
| **Auth** | ❌ لا يحتاج |
| **Format** | `application/json` |
| **Feature** | شاشة تفاصيل التصنيف — المتاجر والمنتجات |
| **Pagination** | ⚠️ مقترح |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `string` | ✅ Required | معرف التصنيف |
| `lat` | `string` | ✅ Required | خط العرض للمستخدم |
| `lng` | `string` | ✅ Required | خط الطول للمستخدم |

### Expected Response `200 OK`

```json
{
  "status": "success",
  "data": {
    "vendors": [ { ... } ],
    "products": [ { ... } ]
  }
}
```

---

---

# 📊 Pagination Recommendations

جدول الـ Endpoints التي يُنصح بإضافة Pagination لها:

| Endpoint | الوضع الحالي | الاقتراح |
|----------|-------------|----------|
| `User/getOffers` | لا يوجد | Cursor-based أو Offset |
| `User/getBestShops` | لا يوجد | Offset-based |
| `User/getBestProducts` | لا يوجد | Offset-based |
| `User/getMyOrders` | لا يوجد | Offset-based |
| `User/getFavorites` | `page` يبدأ من `0` | ✅ موجود — وثّقه |
| `User/getShopReviews` | لا يوجد | Offset-based |
| `User/getReels` | لا يوجد | Cursor-based |
| `User/getVendorVideos` | لا يوجد | Offset-based |
| `User/getReelComments` | لا يوجد | Cursor-based |
| `User/getReelsFavorites` | لا يوجد | Offset-based |
| `User/getNotifications` | لا يوجد | Cursor-based |
| `search` | لا يوجد | Offset-based |
| `User/getCategoryDetails` | لا يوجد | Offset-based |

### Suggested Pagination Contract (Offset-Based)

**Request:**
```json
{
  "page": 1,
  "per_page": 20
}
```

**Response:**
```json
{
  "status": "success",
  "data": [ ... ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total": 150,
    "last_page": 8
  }
}
```

### Suggested Pagination Contract (Cursor-Based — للـ Reels)

**Request:**
```json
{
  "cursor": "eyJpZCI6MTAwfQ=="
}
```

**Response:**
```json
{
  "status": "success",
  "data": [ ... ],
  "next_cursor": "eyJpZCI6ODB9",
  "has_more": true
}
```

---

---

# ⚠️ HTTP Error Codes Reference

| Code | Meaning | متى يحدث |
|------|---------|----------|
| `200` | OK | الطلب نجح — تحقق من `status` في الـ Body |
| `400` | Bad Request | بيانات الطلب غير صحيحة أو ناقصة |
| `401` | Unauthorized | الـ Token منتهي أو غير صحيح |
| `403` | Forbidden | لا يملك المستخدم صلاحية الوصول |
| `404` | Not Found | الـ Endpoint أو الـ Resource غير موجود |
| `409` | Conflict | البيانات موجودة مسبقاً (مثل رقم هاتف مكرر) |
| `422` | Unprocessable Entity | فشل التحقق من البيانات (Validation Error) |
| `500` | Internal Server Error | خطأ في الخادم |

> [!CAUTION]
> معظم الأخطاء التجارية في هذا الـ API تُرجع بـ `HTTP 200` مع `status: "error"` في الـ Body — لا تعتمد على HTTP Status Code وحده للتحقق من النجاح.

---

---

# 🌐 Image URL Resolution

رابط الصور يُبنى وفق القاعدة التالية:

```
Base Image URL: https://sahlaapp.com/LEO/AdminApiLar/public/
```

| حالة | التحويل |
|------|---------|
| رابط مطلق (يبدأ بـ `http`) | يُستخدم كما هو |
| مسار نسبي يبدأ بـ `Photo/` | `Base URL + raw` |
| مسار نسبي لا يبدأ بـ `Photo/` | `Base URL + Photo/ + raw` |

**مثال:**
```
"Photo/vendors/logo.jpg"
→ https://sahlaapp.com/LEO/AdminApiLar/public/Photo/vendors/logo.jpg
```
