# أحلي مسا عليك يصاحبي
# 📄 API Documentation — DealsFoodApp
**API Version:** `v1`  
**Base URL:** `https://sahlaapp.com/api/v1/`  
**Image Base URL:** `https://sahlaapp.com/storage/`  
**Last Updated:** 2026-08-10  

---

## 📌 Design Principles (مبادئ التصميم)

| المبدأ | التطبيق |
|--------|---------|
| **HTTP Verbs** | `GET` للاسترجاع، `POST` للإنشاء، `PUT/PATCH` للتعديل، `DELETE` للحذف |
| **Authentication** | `Bearer Token` في الـ `Authorization` Header |
| **Soft Delete** | لا يوجد حذف فعلي — يُغيَّر الـ `status` إلى `deleted` |
| **Pagination** | دائماً عبر **Query Parameters** |
| **Filtering** | دائماً عبر **Query Parameters** |
| **Naming** | `snake_case` لكل الـ fields |
| **Versioning** | `/api/v1/` في بداية كل URL |

---

## 📌 Global Headers

| Header | Value | Notes |
|--------|-------|-------|
| `Authorization` | `Bearer {access_token}` | للـ Protected routes |
| `Accept` | `application/json` | دائماً |
| `Accept-Language` | `ar` / `en` | لغة الـ Response |
| `Content-Type` | `application/json` | لـ JSON body |
| `Content-Type` | `multipart/form-data` | لرفع الملفات فقط |

> [!IMPORTANT]
> **لا يُرسَل `UserID` أبداً في الـ Request Body.** الـ Backend يستخرج هوية المستخدم من الـ Bearer Token في كل الـ Protected routes.

---

## 📌 Standard Response Envelope

**النجاح:**
```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": { ... }
}
```

**فشل:**
```json
{
  "success": false,
  "message": "Human-readable error message",
  "errors": {
    "field_name": ["Validation error detail"]
  }
}
```

**قائمة مع Pagination:**
```json
{
  "success": true,
  "data": [ ... ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total": 150,
    "last_page": 8,
    "has_more": true
  }
}
```

---

## 📌 Pagination Contract (Global)

**يُضاف لأي Endpoint يُرجع قائمة:**

| Query Param | Type | Default | Description |
|-------------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد العناصر في الصفحة (max: 100) |

---

## 📌 Authentication Strategy

- الـ `access_token` يُرسَل فقط في `Authorization: Bearer {token}` Header.
- الـ Backend يعرف هوية المستخدم تلقائياً من الـ Token.
- لا داعي لإرسال `UserID` أو `user_id` في أي Request Body أو Query Param.

---

## ⚠️ HTTP Error Codes Reference

| Code | Meaning | متى يحدث |
|------|---------|----------|
| `200` | OK | الطلب نجح (GET, PATCH, DELETE) |
| `201` | Created | إنشاء ناجح (POST) |
| `204` | No Content | حذف ناجح (DELETE) بدون Response Body |
| `400` | Bad Request | بيانات غير صحيحة أو ناقصة |
| `401` | Unauthorized | الـ Token منتهي أو غير صحيح |
| `403` | Forbidden | لا يملك صلاحية الوصول |
| `404` | Not Found | الـ Resource غير موجود |
| `409` | Conflict | البيانات موجودة مسبقاً (مثل رقم هاتف مكرر) |
| `422` | Unprocessable Entity | فشل Validation |
| `429` | Too Many Requests | تجاوز Rate Limit |
| `500` | Internal Server Error | خطأ في الخادم |

> [!CAUTION]
> **ممنوع** إرجاع أخطاء تجارية بـ `HTTP 200`. كل خطأ له Status Code صحيح.

---

---

# 🔐 1. Authentication

---

## 1.1 Register

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/register` |
| **Auth** | ❌ Public |
| **Format** | `application/json` |

### Request Body

```json
{
  "first_name": "أحمد",
  "last_name": "محمد",
  "email": "ahmed@example.com",
  "phone_key": "+966",
  "phone_number": "501234567",
  "password": "SecurePass123!",
  "password_confirmation": "SecurePass123!",
  "latitude": "24.7136",
  "longitude": "46.6753",
  "fcm_token": "fcm_abc123"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `first_name` | `string` | ✅ | الاسم الأول |
| `last_name` | `string` | ✅ | اسم العائلة |
| `email` | `string` | ✅ | البريد الإلكتروني |
| `phone_key` | `string` | ✅ | كود الدولة مثل `+966` |
| `phone_number` | `string` | ✅ | رقم الجوال بدون كود |
| `password` | `string` | ✅ | كلمة المرور (min: 8 chars) |
| `password_confirmation` | `string` | ✅ | تأكيد كلمة المرور |
| `latitude` | `string` | ⬜ | خط العرض |
| `longitude` | `string` | ⬜ | خط الطول |
| `fcm_token` | `string` | ⬜ | Firebase Push Notification Token |

### Response `201 Created`

```json
{
  "success": true,
  "message": "Account created successfully",
  "data": {
    "user": {
      "id": "123",
      "first_name": "أحمد",
      "last_name": "محمد",
      "email": "ahmed@example.com",
      "phone_key": "+966",
      "phone_number": "501234567",
      "profile_pic": null,
      "auth_method": "phone",
      "is_verified": false,
      "created_at": "2026-08-10T14:00:00Z"
    },
    "access_token": "eyJ0eXAiOiJKV...",
    "token_type": "Bearer",
    "expires_in": 86400
  }
}
```

### Error Responses

| Status | Scenario |
|--------|----------|
| `201` | تم التسجيل بنجاح |
| `409` | رقم الهاتف أو البريد مسجل مسبقاً |
| `422` | بيانات غير صحيحة (Validation failed) |

---

## 1.2 Login

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/login` |
| **Auth** | ❌ Public |
| **Format** | `application/json` |

### Request Body

```json
{
  "phone_key": "+966",
  "phone_number": "501234567",
  "password": "SecurePass123!",
  "fcm_token": "fcm_abc123"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `phone_key` | `string` | ✅ | كود الدولة |
| `phone_number` | `string` | ✅ | رقم الجوال |
| `password` | `string` | ✅ | كلمة المرور |
| `fcm_token` | `string` | ⬜ | Firebase Token |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "user": { ... },
    "access_token": "eyJ0eXAiOiJKV...",
    "token_type": "Bearer",
    "expires_in": 86400
  }
}
```

### Error Responses

| Status | Scenario |
|--------|----------|
| `200` | تسجيل دخول ناجح |
| `401` | بيانات خاطئة |
| `403` | الحساب محظور أو غير نشط |
| `429` | محاولات دخول كثيرة (Rate Limit) |

---

## 1.3 Social Login (Google / Apple)

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/social-login` |
| **Auth** | ❌ Public |
| **Format** | `application/json` |

### Request Body

```json
{
  "provider": "google",
  "id_token": "google_oauth_token_here",
  "email": "user@gmail.com",
  "first_name": "Sara",
  "last_name": "Ali",
  "profile_pic": "https://lh3.googleusercontent.com/...",
  "fcm_token": "fcm_abc123"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `provider` | `string` | ✅ | `"google"` أو `"apple"` |
| `id_token` | `string` | ✅ | OAuth Token من الـ Provider |
| `email` | `string` | ✅ | البريد الإلكتروني |
| `first_name` | `string` | ✅ | الاسم الأول |
| `last_name` | `string` | ✅ | اسم العائلة |
| `profile_pic` | `string` | ⬜ | رابط صورة الـ Profile |
| `fcm_token` | `string` | ⬜ | Firebase Token |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "user": { ... },
    "access_token": "eyJ0eXAiOiJKV...",
    "token_type": "Bearer",
    "expires_in": 86400,
    "is_new_user": false
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `is_new_user` | `boolean` | `true` إذا تم إنشاء حساب جديد |

---

## 1.4 Send OTP

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/otp/send` |
| **Auth** | ❌ Public |
| **Format** | `application/json` |

### Request Body

```json
{
  "phone_key": "+966",
  "phone_number": "501234567"
}
```

### Response `200 OK`

```json
{
  "success": true,
  "message": "OTP sent successfully",
  "data": {
    "expires_in": 300
  }
}
```

> [!CAUTION]
> **لا يُرجَع الـ OTP Code أبداً في الـ Response.** الـ Backend يرسله للمستخدم عبر SMS/WhatsApp ويتحقق منه server-side عبر endpoint منفصل.

---

## 1.5 Verify OTP

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/otp/verify` |
| **Auth** | ❌ Public |
| **Format** | `application/json` |

### Request Body

```json
{
  "phone_key": "+966",
  "phone_number": "501234567",
  "otp_code": "1234"
}
```

### Response `200 OK`

```json
{
  "success": true,
  "message": "Phone number verified successfully"
}
```

---

## 1.6 Check Phone Availability

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/auth/check-phone` |
| **Auth** | ❌ Public |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `phone_key` | `string` | ✅ | كود الدولة |
| `phone_number` | `string` | ✅ | رقم الجوال |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "available": true
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `available` | `boolean` | `true` = الرقم غير مُسجَّل (متاح للتسجيل) |

---

## 1.7 Logout

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/logout` |
| **Auth** | ✅ Bearer Token |

### Response `200 OK`

```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## 1.8 Refresh Token

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/auth/refresh` |
| **Auth** | ✅ Bearer Token (منتهي الصلاحية) |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "access_token": "new_token_here",
    "expires_in": 86400
  }
}
```

---

---

# 🏠 2. Home

---

## 2.1 Get Sliders

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/home/sliders` |
| **Auth** | ❌ Public |
| **Pagination** | ❌ لا يحتاج |
| **Cache** | ✅ `Cache-Control: max-age=3600` |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "image_url": "https://sahlaapp.com/storage/sliders/banner1.jpg",
      "text_ar": "عروض رمضان",
      "text_en": "Ramadan Offers",
      "action_type": "shop",
      "action_value": "42",
      "slide_type": "normal",
      "expires_at": "2026-12-31T23:59:59Z"
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `string` | ✅ | معرف الشريحة |
| `image_url` | `string` | ✅ | رابط الصورة (كامل — لا يحتاج بناء يدوي) |
| `text_ar` | `string` | ✅ | النص العربي |
| `text_en` | `string` | ✅ | النص الإنجليزي |
| `action_type` | `string` | ✅ | `none` \| `shop` \| `offer` \| `link` |
| `action_value` | `string` | ⬜ | القيمة المرتبطة |
| `slide_type` | `string` | ✅ | `normal` \| `ad` |
| `expires_at` | `string` | ⬜ | تاريخ الانتهاء (ISO 8601) أو `null` للدائمة |

---

## 2.2 Get Main Categories

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/home/categories` |
| **Auth** | ❌ Public |
| **Pagination** | ❌ لا يحتاج |
| **Cache** | ✅ `Cache-Control: max-age=3600` |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "parent_id": null,
      "name_ar": "مطاعم",
      "name_en": "Restaurants",
      "image_url": "https://sahlaapp.com/storage/categories/rest.jpg"
    }
  ]
}
```

---

## 2.3 Get Offers

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/home/offers` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |
| **Cache** | ✅ `Cache-Control: max-age=300` |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد العروض في الصفحة |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "10",
      "title_ar": "وجبة عائلية",
      "title_en": "Family Meal",
      "description_ar": "وجبة كاملة للعائلة",
      "description_en": "Complete family meal",
      "status": "active",
      "color": "#fb7f33",
      "image_url": "https://sahlaapp.com/storage/offers/family.jpg",
      "vendor": {
        "id": "42",
        "name_ar": "مطعم النخيل",
        "name_en": "Al Nakheel",
        "logo_url": "https://sahlaapp.com/storage/vendors/logo.jpg"
      }
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total": 45,
    "last_page": 3,
    "has_more": true
  }
}
```

---

## 2.4 Get Best Shops

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/home/best-shops` |
| **Auth** | ❌ Public (⬜ Optional Bearer للـ `is_favorite`) |
| **Pagination** | ✅ Query Params |
| **Cache** | ✅ `Cache-Control: max-age=300` |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد المتاجر |
| `lat` | `float` | ⬜ | خط العرض (لحساب المسافة) |
| `lng` | `float` | ⬜ | خط الطول |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "42",
      "name_ar": "مطعم النخيل",
      "name_en": "Al Nakheel",
      "logo_url": "https://sahlaapp.com/storage/vendors/logo.jpg",
      "cover_url": "https://sahlaapp.com/storage/vendors/cover.jpg",
      "rating": 4.5,
      "rating_count": 120,
      "delivery_time_min": 30,
      "delivery_fee": 15.00,
      "is_open": true,
      "is_busy": false,
      "status": "active",
      "distance_km": 2.5,
      "address": "الرياض، حي النخيل",
      "is_favorite": false
    }
  ],
  "meta": { ... }
}
```

---

## 2.5 Get Best Products

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/home/best-products` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |
| **Cache** | ✅ `Cache-Control: max-age=300` |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد المنتجات |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "101",
      "name_ar": "برجر دجاج",
      "name_en": "Chicken Burger",
      "description_ar": "برجر دجاج مقرمش",
      "description_en": "Crispy chicken burger",
      "base_price": 35.00,
      "sale_price": 25.00,
      "is_on_sale": true,
      "discount_percent": 29,
      "image_url": "https://sahlaapp.com/storage/products/burger.jpg",
      "vendor": {
        "id": "42",
        "name_ar": "مطعم النخيل",
        "name_en": "Al Nakheel"
      }
    }
  ],
  "meta": { ... }
}
```

---

---

# 🏪 3. Shops

---

## 3.1 Get Shop Details

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/shops/{shop_id}` |
| **Auth** | ⬜ Optional Bearer (لمعرفة `is_favorite`) |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shop_id` | `string` | ✅ | معرف المتجر |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "id": "42",
    "name_ar": "مطعم النخيل",
    "name_en": "Al Nakheel",
    "logo_url": "https://sahlaapp.com/storage/vendors/logo.jpg",
    "cover_url": "https://sahlaapp.com/storage/vendors/cover.jpg",
    "rating": 4.5,
    "rating_count": 120,
    "delivery_time_min": 30,
    "delivery_fee": 15.00,
    "is_open": true,
    "is_busy": false,
    "phone_number": "0501234567",
    "email": "info@nakheel.com",
    "latitude": 24.7136,
    "longitude": 46.6753,
    "address": "الرياض، حي النخيل",
    "status": "active",
    "is_favorite": true
  }
}
```

---

## 3.2 Get Shop Menu (Products by Sections)

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/shops/{shop_id}/menu` |
| **Auth** | ❌ Public |
| **Pagination** | ❌ يُرجع كل القائمة دفعة واحدة |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shop_id` | `string` | ✅ | معرف المتجر |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "5",
      "name_ar": "الوجبات الرئيسية",
      "name_en": "Main Meals",
      "sort_order": 1,
      "products": [
        {
          "id": "101",
          "name_ar": "برجر دجاج",
          "name_en": "Chicken Burger",
          "base_price": 35.00,
          "sale_price": 25.00,
          "is_on_sale": true,
          "discount_percent": 29,
          "image_url": "https://sahlaapp.com/storage/products/burger.jpg",
          "is_available": true
        }
      ]
    }
  ]
}
```

---

## 3.3 Get Shop Reviews

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/shops/{shop_id}/reviews` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shop_id` | `string` | ✅ | معرف المتجر |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `10` | عدد التقييمات |
| `rating` | `integer` | ⬜ | فلترة بتقييم محدد (1-5) |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "201",
      "rating": 5,
      "comment": "خدمة ممتازة",
      "created_at": "2026-01-15T10:30:00Z",
      "order_id": 789,
      "user": {
        "first_name": "محمد",
        "last_name": "علي",
        "profile_pic": "https://sahlaapp.com/storage/users/pic.jpg"
      }
    }
  ],
  "meta": { ... }
}
```

---

## 3.4 Get Shop Videos

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/shops/{shop_id}/videos` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shop_id` | `string` | ✅ | معرف المتجر |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `15` | عدد الفيديوهات |

### Response `200 OK`

```json
{
  "success": true,
  "data": [ { ... reel objects ... } ],
  "meta": { ... }
}
```

---

## 3.5 Toggle Favorite Shop

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/shops/{shop_id}/favorite` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shop_id` | `string` | ✅ | معرف المتجر |

> [!TIP]
> استخدام `POST` كـ Toggle يبسط الـ Client-side — الـ Backend يتحقق من الحالة الحالية ويعكسها.

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "is_favorite": true
  }
}
```

---

## 3.6 Get Favorite Shops

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/me/favorite-shops` |
| **Auth** | ✅ Bearer Token |
| **Pagination** | ✅ Query Params |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد المتاجر |

### Response `200 OK`

```json
{
  "success": true,
  "data": [ { ... shop objects ... } ],
  "meta": { ... }
}
```

---

---

# 📦 4. Products

---

## 4.1 Get Product Details

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/products/{product_id}` |
| **Auth** | ❌ Public |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `product_id` | `string` | ✅ | معرف المنتج |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "id": "101",
    "name_ar": "برجر دجاج",
    "name_en": "Chicken Burger",
    "description_ar": "برجر دجاج مقرمش",
    "description_en": "Crispy chicken burger",
    "base_price": 35.00,
    "sale_price": 25.00,
    "is_on_sale": true,
    "discount_percent": 29,
    "images": [
      "https://sahlaapp.com/storage/products/burger1.jpg",
      "https://sahlaapp.com/storage/products/burger2.jpg"
    ],
    "vendor": {
      "id": "42",
      "name_ar": "مطعم النخيل",
      "name_en": "Al Nakheel",
      "logo_url": "..."
    },
    "is_available": true
  }
}
```

> [!TIP]
> دمج صور المنتج مع تفاصيله في endpoint واحد يقلل عدد الـ API calls من 2 إلى 1.

---

## 4.2 Get Product Extras

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/products/{product_id}/extras` |
| **Auth** | ❌ Public |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `product_id` | `string` | ✅ | معرف المنتج |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "3",
      "name_ar": "الإضافات",
      "name_en": "Extras",
      "min_choices": 0,
      "max_choices": 2,
      "sort_order": 1,
      "is_required": false,
      "items": [
        {
          "id": "10",
          "name_ar": "جبنة إضافية",
          "name_en": "Extra Cheese",
          "price": 5.00,
          "is_available": true
        }
      ]
    }
  ]
}
```

---

## 4.3 Get Similar Products

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/products/{product_id}/similar` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `product_id` | `string` | ✅ | معرف المنتج |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | `integer` | `10` | عدد المنتجات المشابهة (max: 20) |

### Response `200 OK`

```json
{
  "success": true,
  "data": [ { ... product objects ... } ]
}
```

---

---

# 🛒 5. Cart

---

## 5.1 Add Item to Cart

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/cart/items` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Request Body

```json
{
  "product_id": "101",
  "vendor_id": "42",
  "quantity": 2,
  "extras": ["10", "11"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `product_id` | `string` | ✅ | معرف المنتج |
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `quantity` | `integer` | ✅ | الكمية (min: 1) |
| `extras` | `array<string>` | ⬜ | معرفات الإضافات المختارة |

### Response `201 Created`

```json
{
  "success": true,
  "message": "Item added to cart",
  "data": {
    "cart_item_id": "501",
    "product_id": "101",
    "quantity": 2,
    "unit_price": 25.00,
    "line_total": 50.00
  }
}
```

---

## 5.2 Get My Cart

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/cart` |
| **Auth** | ✅ Bearer Token |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "501",
        "product_id": "101",
        "name_ar": "برجر دجاج",
        "name_en": "Chicken Burger",
        "quantity": 2,
        "unit_price": 25.00,
        "original_price": 35.00,
        "line_total": 50.00,
        "image_url": "https://sahlaapp.com/storage/products/burger.jpg",
        "vendor_id": "42",
        "vendor_name_ar": "مطعم النخيل",
        "vendor_name_en": "Al Nakheel",
        "extras_text_ar": "جبنة إضافية",
        "extras_text_en": "Extra Cheese"
      }
    ],
    "items_count": 2,
    "subtotal": 50.00,
    "vendor_id": "42"
  }
}
```

---

## 5.3 Get Cart Item Count

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/cart/count` |
| **Auth** | ✅ Bearer Token |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "count": 3
  }
}
```

---

## 5.4 Update Cart Item Quantity

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/cart/items/{cart_item_id}` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `cart_item_id` | `string` | ✅ | معرف عنصر السلة |

### Request Body

```json
{
  "quantity": 3
}
```

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "id": "501",
    "quantity": 3,
    "line_total": 75.00
  }
}
```

---

## 5.5 Remove Cart Item

| Property | Value |
|----------|-------|
| **Method** | `DELETE` |
| **URL** | `/cart/items/{cart_item_id}` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `cart_item_id` | `string` | ✅ | معرف عنصر السلة |

### Response `204 No Content`

> لا يوجد Response Body.

---

## 5.6 Calculate Delivery Fee

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/cart/delivery-fee` |
| **Auth** | ✅ Bearer Token |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `lat` | `float` | ✅ | خط عرض العنوان المختار |
| `lng` | `float` | ✅ | خط طول العنوان المختار |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "delivery_fee": 15.00,
    "branch_id": "7",
    "branch_name": "فرع الرياض",
    "distance_km": 3.5,
    "estimated_minutes": 30
  }
}
```

---

## 5.7 Place Order

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/orders` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Request Body

```json
{
  "vendor_id": "42",
  "address_id": "201",
  "payment_method": "cash",
  "subtotal": 50.00,
  "delivery_fee": 15.00,
  "total": 65.00
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_id` | `string` | ✅ | معرف المتجر |
| `address_id` | `string` | ✅ | معرف العنوان المختار |
| `payment_method` | `string` | ✅ | `cash` \| `online` |
| `subtotal` | `float` | ✅ | مجموع المنتجات |
| `delivery_fee` | `float` | ✅ | رسوم التوصيل |
| `total` | `float` | ✅ | المجموع الكلي |

### Response `201 Created`

```json
{
  "success": true,
  "message": "Order placed successfully",
  "data": {
    "order_id": "789",
    "status": "pending",
    "created_at": "2026-08-10T14:00:00Z"
  }
}
```

---

---

# 📋 6. Orders

---

## 6.1 Get My Orders

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/orders` |
| **Auth** | ✅ Bearer Token |
| **Pagination** | ✅ Query Params |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد الطلبات |
| `status` | `string` | ⬜ | فلترة بالحالة: `pending`, `confirmed`, `delivered`, `cancelled` |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "789",
      "vendor": {
        "id": "42",
        "name_ar": "مطعم النخيل",
        "name_en": "Al Nakheel",
        "logo_url": "https://sahlaapp.com/storage/vendors/logo.jpg"
      },
      "status": "delivered",
      "total": 65.00,
      "subtotal": 50.00,
      "delivery_fee": 15.00,
      "items_count": 3,
      "currency": "SAR",
      "payment_method": "cash",
      "payment_status": "paid",
      "created_at": "2026-08-01T14:30:00Z",
      "driver": {
        "name": "خالد",
        "phone": "0501111111",
        "tracking_url": "https://..."
      },
      "is_rated": false,
      "rating": null
    }
  ],
  "meta": { ... }
}
```

---

## 6.2 Get Order Details

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/orders/{order_id}` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `order_id` | `string` | ✅ | معرف الطلب |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "id": "789",
    "status": "delivered",
    "items": [
      {
        "id": 1,
        "product_id": 101,
        "name_ar": "برجر دجاج",
        "name_en": "Chicken Burger",
        "quantity": 2,
        "unit_price": 25.00,
        "total_price": 50.00,
        "image_url": "https://sahlaapp.com/storage/products/burger.jpg",
        "extras_text_ar": "",
        "extras_text_en": ""
      }
    ],
    "created_at": "2026-08-01T14:30:00Z"
  }
}
```

---

## 6.3 Rate Shop

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/shops/{shop_id}/ratings` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shop_id` | `string` | ✅ | معرف المتجر |

### Request Body

```json
{
  "order_id": "789",
  "rating": 5,
  "comment": "خدمة ممتازة"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `order_id` | `string` | ✅ | معرف الطلب المرتبط |
| `rating` | `integer` | ✅ | التقييم (1-5) |
| `comment` | `string` | ⬜ | التعليق |

### Response `201 Created`

```json
{
  "success": true,
  "message": "Rating submitted successfully"
}
```

---

## 6.4 Rate Order

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/orders/{order_id}/rating` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `order_id` | `string` | ✅ | معرف الطلب |

### Request Body

```json
{
  "vendor_rating": 5,
  "delivery_rating": 4,
  "comment": "الطعام ممتاز، التوصيل كان شوية متأخر"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `vendor_rating` | `integer` | ✅ | تقييم المتجر (1-5) |
| `delivery_rating` | `integer` | ✅ | تقييم التوصيل (1-5) |
| `comment` | `string` | ⬜ | تعليق |

### Response `201 Created`

```json
{
  "success": true,
  "message": "Order rated successfully"
}
```

---

## 6.5 Place Offer Order

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/orders/offers` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Request Body

```json
{
  "offer_id": "10",
  "address_id": "201",
  "delivery_fee": 15.00
}
```

### Response `201 Created`

```json
{
  "success": true,
  "data": {
    "order_id": "990",
    "status": "pending",
    "created_at": "2026-08-10T14:00:00Z"
  }
}
```

---

---

# 👤 7. Profile (Me)

---

## 7.1 Get My Profile

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/me` |
| **Auth** | ✅ Bearer Token |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "id": "123",
    "first_name": "أحمد",
    "last_name": "محمد",
    "email": "ahmed@example.com",
    "phone_key": "+966",
    "phone_number": "501234567",
    "profile_pic": "https://sahlaapp.com/storage/users/pic.jpg",
    "auth_method": "phone",
    "latitude": 24.7136,
    "longitude": 46.6753,
    "wallet_balance": 0.00,
    "is_verified": true,
    "app_language": "ar",
    "is_active": true,
    "created_at": "2026-01-01T00:00:00Z",
    "updated_at": "2026-08-01T00:00:00Z"
  }
}
```

---

## 7.2 Update My Profile (Text Only)

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/me` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Request Body

```json
{
  "first_name": "أحمد",
  "last_name": "محمد",
  "email": "ahmed@example.com",
  "phone_key": "+966",
  "phone_number": "501234567"
}
```

> [!TIP]
> `PATCH` يعني أن الـ Client يُرسل فقط الـ Fields التي تغيرت — ليس إلزامياً إرسال كل البيانات.

### Response `200 OK`

```json
{
  "success": true,
  "message": "Profile updated successfully",
  "data": { ... updated user object ... }
}
```

---

## 7.3 Update Profile Picture

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/me/profile-picture` |
| **Auth** | ✅ Bearer Token |
| **Format** | `multipart/form-data` |

> [!NOTE]
> نستخدم `POST` لرفع الملفات لأن `PATCH` مع `multipart` غير مدعوم في بعض الـ Servers.

### Request Fields (Form Data)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `profile_pic` | `file` | ✅ | ملف الصورة (jpg, png — max: 5MB) |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "profile_pic": "https://sahlaapp.com/storage/users/new_pic.jpg"
  }
}
```

---

## 7.4 Change Password

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/me/password` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Request Body

```json
{
  "current_password": "OldPass123!",
  "new_password": "NewPass456!",
  "new_password_confirmation": "NewPass456!"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `current_password` | `string` | ✅ | كلمة المرور الحالية |
| `new_password` | `string` | ✅ | كلمة المرور الجديدة (min: 8 chars) |
| `new_password_confirmation` | `string` | ✅ | تأكيد كلمة المرور الجديدة |

### Response `200 OK`

```json
{
  "success": true,
  "message": "Password changed successfully"
}
```

---

## 7.5 Delete Account (Soft Delete)

| Property | Value |
|----------|-------|
| **Method** | `DELETE` |
| **URL** | `/me` |
| **Auth** | ✅ Bearer Token |

> [!IMPORTANT]
> **Soft Delete فقط** — لا يُحذف الحساب من قاعدة البيانات.  
> الـ Backend يُغيِّر `status` إلى `deleted` ويُبطل كل الـ Tokens.

### Response `200 OK`

```json
{
  "success": true,
  "message": "Account has been deactivated"
}
```

---

## 7.6 Get My Favorite Reels

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/me/favorite-reels` |
| **Auth** | ✅ Bearer Token |
| **Pagination** | ✅ Query Params |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `15` | عدد الـ Reels |

### Response `200 OK`

```json
{
  "success": true,
  "data": [ { ... reel objects ... } ],
  "meta": { ... }
}
```

---

## 7.7 Contact Us

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/support/contact` |
| **Auth** | ❌ Public |
| **Format** | `application/json` |

### Request Body

```json
{
  "name": "أحمد محمد",
  "email": "ahmed@example.com",
  "phone": "0501234567",
  "message": "استفساري هو..."
}
```

### Response `201 Created`

```json
{
  "success": true,
  "message": "Your message has been sent successfully"
}
```

---

---

# 📍 8. Addresses

---

## 8.1 Get My Addresses

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/me/addresses` |
| **Auth** | ✅ Bearer Token |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "201",
      "label": "المنزل",
      "type": "home",
      "city": "الرياض",
      "city_ar": "الرياض",
      "city_en": "Riyadh",
      "area": "النخيل",
      "street_name": "شارع الملك فهد",
      "building_number": "12",
      "floor_number": "3",
      "apartment_number": "5",
      "landmark": "بجوار مسجد النور",
      "latitude": 24.7136,
      "longitude": 46.6753,
      "delivery_fee": 15.00,
      "is_default": true
    }
  ]
}
```

---

## 8.2 Add Address

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/me/addresses` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Request Body

```json
{
  "label": "المنزل",
  "city": "الرياض",
  "latitude": 24.7136,
  "longitude": 46.6753,
  "street_name": "شارع الملك فهد",
  "building_number": "12",
  "floor_number": "3",
  "apartment_number": "5",
  "landmark": "بجوار مسجد النور"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `label` | `string` | ✅ | تسمية العنوان |
| `city` | `string` | ✅ | المدينة |
| `latitude` | `float` | ✅ | خط العرض |
| `longitude` | `float` | ✅ | خط الطول |
| `street_name` | `string` | ⬜ | اسم الشارع |
| `building_number` | `string` | ⬜ | رقم المبنى |
| `floor_number` | `string` | ⬜ | رقم الطابق |
| `apartment_number` | `string` | ⬜ | رقم الشقة |
| `landmark` | `string` | ⬜ | معلم قريب |

### Response `201 Created`

```json
{
  "success": true,
  "data": {
    "id": "202",
    "label": "المنزل",
    "is_default": false,
    ...
  }
}
```

---

## 8.3 Update Address

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/me/addresses/{address_id}` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `address_id` | `string` | ✅ | معرف العنوان |

### Request Body

يُرسَل فقط الـ Fields المُراد تعديلها.

### Response `200 OK`

```json
{
  "success": true,
  "data": { ... updated address ... }
}
```

---

## 8.4 Delete Address (Soft Delete)

| Property | Value |
|----------|-------|
| **Method** | `DELETE` |
| **URL** | `/me/addresses/{address_id}` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `address_id` | `string` | ✅ | معرف العنوان |

> [!IMPORTANT]
> **Soft Delete** — يُغيَّر `status` إلى `deleted`. العنوان لا يُحذف من قاعدة البيانات، ويظل مرتبطاً بالطلبات القديمة.

### Response `204 No Content`

---

## 8.5 Set Default Address

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/me/addresses/{address_id}/default` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `address_id` | `string` | ✅ | معرف العنوان |

### Response `200 OK`

```json
{
  "success": true,
  "message": "Default address updated"
}
```

---

## 8.6 Get Cities

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/cities` |
| **Auth** | ❌ Public |
| **Cache** | ✅ `Cache-Control: max-age=86400` |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "name_ar": "الرياض",
      "name_en": "Riyadh"
    },
    {
      "id": "2",
      "name_ar": "جدة",
      "name_en": "Jeddah"
    }
  ]
}
```

---

---

# 🔔 9. Notifications

---

## 9.1 Get Notifications

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/me/notifications` |
| **Auth** | ✅ Bearer Token |
| **Pagination** | ✅ Query Params (Cursor-based) |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `cursor` | `string` | ⬜ | معرف آخر إشعار مُستلَم (للتحميل التدريجي) |
| `per_page` | `integer` | `20` | عدد الإشعارات |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "unread_count": 3,
    "notifications": [
      {
        "id": "501",
        "body_ar": "تم تأكيد طلبك",
        "body_en": "Your order has been confirmed",
        "is_read": false,
        "action_type": "order",
        "action_id": "789",
        "created_at": "2026-08-01T10:00:00Z"
      }
    ]
  },
  "next_cursor": "eyJpZCI6NDk5fQ==",
  "has_more": true
}
```

| Field | Type | Description |
|-------|------|-------------|
| `unread_count` | `integer` | عدد الإشعارات غير المقروءة |
| `action_type` | `string` | `order` \| `shop` \| `general` |
| `action_id` | `string` | معرف الكيان المرتبط |

---

## 9.2 Mark Notification as Read

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/me/notifications/{notification_id}/read` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `notification_id` | `string` | ✅ | معرف الإشعار |

### Response `200 OK`

```json
{
  "success": true
}
```

---

## 9.3 Mark All Notifications as Read

| Property | Value |
|----------|-------|
| **Method** | `PATCH` |
| **URL** | `/me/notifications/read-all` |
| **Auth** | ✅ Bearer Token |

### Response `200 OK`

```json
{
  "success": true,
  "message": "All notifications marked as read"
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
| **URL** | `/search` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | `string` | ✅ | نص البحث (min: 2 chars) |
| `lat` | `float` | ⬜ | خط عرض المستخدم |
| `lng` | `float` | ⬜ | خط طول المستخدم |
| `type` | `string` | ⬜ | `all` \| `vendors` \| `products` (default: `all`) |
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد النتائج |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "vendors": [
      {
        "id": "42",
        "name_ar": "مطعم النخيل",
        "name_en": "Al Nakheel",
        "logo_url": "https://sahlaapp.com/storage/vendors/logo.jpg",
        "cover_url": "https://sahlaapp.com/storage/vendors/cover.jpg",
        "phone_number": "0501234567",
        "address": "الرياض، حي النخيل"
      }
    ],
    "products": [
      {
        "id": "101",
        "name_ar": "برجر دجاج",
        "name_en": "Chicken Burger",
        "description_ar": "وصف بالعربي",
        "description_en": "Description",
        "base_price": 35.00,
        "sale_price": 25.00,
        "is_on_sale": true,
        "discount_percent": 29,
        "image_url": "https://sahlaapp.com/storage/products/burger.jpg",
        "vendor": {
          "id": "42",
          "name_ar": "مطعم النخيل",
          "name_en": "Al Nakheel",
          "logo_url": "..."
        }
      }
    ]
  },
  "meta": {
    "vendors_count": 5,
    "products_count": 12,
    "current_page": 1,
    "has_more": false
  }
}
```

---

---

# 🎬 11. Reels (Social)

---

## 11.1 Get Reels Feed

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/reels` |
| **Auth** | ✅ Bearer Token |
| **Pagination** | ✅ Cursor-based (ضروري للـ infinite scroll) |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `cursor` | `string` | ⬜ | Cursor للتحميل التدريجي |
| `per_page` | `integer` | `10` | عدد الـ Reels (لا يزيد عن 20) |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "301",
      "vendor_id": "42",
      "video_url": "https://sahlaapp.com/storage/reels/video1.mp4",
      "status": "active",
      "description_ar": "وصف الفيديو",
      "description_en": "Video description",
      "vendor": {
        "id": "42",
        "name_ar": "مطعم النخيل",
        "name_en": "Al Nakheel",
        "logo_url": "https://sahlaapp.com/storage/vendors/logo.jpg"
      },
      "stats": {
        "likes_count": 245,
        "comments_count": 18,
        "is_liked": true
      }
    }
  ],
  "next_cursor": "eyJpZCI6MjkwfQ==",
  "has_more": true
}
```

---

## 11.2 Toggle Reel Like

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/reels/{reel_id}/like` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reel_id` | `string` | ✅ | معرف الـ Reel |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "is_liked": true,
    "likes_count": 246
  }
}
```

---

## 11.3 Get Reel Comments

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/reels/{reel_id}/comments` |
| **Auth** | ✅ Bearer Token |
| **Pagination** | ✅ Cursor-based |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reel_id` | `string` | ✅ | معرف الـ Reel |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `cursor` | `string` | ⬜ | Cursor للتحميل التدريجي |
| `per_page` | `integer` | `20` | عدد التعليقات |

### Response `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "comment": "ممتاز!",
      "parent_id": null,
      "created_at": "2026-08-01T10:00:00Z",
      "user": {
        "name": "أحمد",
        "profile_pic": "https://sahlaapp.com/storage/users/pic.jpg"
      }
    }
  ],
  "next_cursor": "eyJpZCI6ODB9",
  "has_more": false
}
```

---

## 11.4 Add Comment to Reel

| Property | Value |
|----------|-------|
| **Method** | `POST` |
| **URL** | `/reels/{reel_id}/comments` |
| **Auth** | ✅ Bearer Token |
| **Format** | `application/json` |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reel_id` | `string` | ✅ | معرف الـ Reel |

### Request Body

```json
{
  "comment": "ممتاز!",
  "parent_id": null
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `comment` | `string` | ✅ | نص التعليق |
| `parent_id` | `string \| null` | ⬜ | معرف التعليق الأب (للرد) |

### Response `201 Created`

```json
{
  "success": true,
  "data": {
    "id": "99",
    "comment": "ممتاز!",
    "created_at": "2026-08-10T14:00:00Z"
  }
}
```

---

## 11.5 Delete Reel Comment (Soft Delete)

| Property | Value |
|----------|-------|
| **Method** | `DELETE` |
| **URL** | `/reels/{reel_id}/comments/{comment_id}` |
| **Auth** | ✅ Bearer Token |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reel_id` | `string` | ✅ | معرف الـ Reel |
| `comment_id` | `string` | ✅ | معرف التعليق |

> [!IMPORTANT]
> **Soft Delete** — يُغيَّر `status` إلى `deleted`. التعليق لا يُحذف من قاعدة البيانات.

### Response `204 No Content`

---

---

# 🎁 12. Offers

---

## 12.1 Get Offer Details

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/offers/{offer_id}` |
| **Auth** | ❌ Public |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `offer_id` | `string` | ✅ | معرف العرض |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "id": "10",
    "title_ar": "وجبة عائلية",
    "title_en": "Family Meal",
    "description_ar": "وصف بالعربي",
    "description_en": "Description",
    "status": "active",
    "color": "#fb7f33",
    "image_url": "https://sahlaapp.com/storage/offers/family.jpg",
    "vendor": {
      "id": "42",
      "name_ar": "مطعم النخيل",
      "name_en": "Al Nakheel",
      "logo_url": "..."
    },
    "products": [
      {
        "offer_price": 99.00,
        "quantity": 2,
        "product_id": "101",
        "name_ar": "برجر دجاج",
        "name_en": "Chicken Burger",
        "description_ar": "وصف",
        "description_en": "Description",
        "image_url": "https://sahlaapp.com/storage/products/burger.jpg"
      }
    ]
  }
}
```

---

---

# 📁 13. Categories

---

## 13.1 Get Category Details

| Property | Value |
|----------|-------|
| **Method** | `GET` |
| **URL** | `/categories/{category_id}` |
| **Auth** | ❌ Public |
| **Pagination** | ✅ Query Params |

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category_id` | `string` | ✅ | معرف التصنيف |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `lat` | `float` | ⬜ | خط العرض (للمتاجر القريبة) |
| `lng` | `float` | ⬜ | خط الطول |
| `page` | `integer` | `1` | رقم الصفحة |
| `per_page` | `integer` | `20` | عدد النتائج |

### Response `200 OK`

```json
{
  "success": true,
  "data": {
    "category": {
      "id": "1",
      "name_ar": "مطاعم",
      "name_en": "Restaurants",
      "image_url": "..."
    },
    "vendors": [ { ... } ],
    "products": [ { ... } ]
  },
  "meta": { ... }
}
```

---

---

# 🏗️ Architecture & Performance Recommendations

---

## A. Endpoint Consolidation (تقليل عدد الـ Calls)

| بدلاً من | استخدم |
|---------|-------|
| `GET /home/sliders` + `GET /home/categories` + `GET /home/offers` + `GET /home/best-shops` + `GET /home/best-products` | **`GET /home/feed`** — endpoint واحد يُرجع كل بيانات الهوم |
| `GET /products/{id}` + `GET /products/{id}/extras` + `GET /products/{id}/similar` | ضم الـ `images` و`extras` في `/products/{id}` مع `similar` كـ lazy-loaded |
| `GET /shops/{id}` + `GET /shops/{id}/menu` | تحديد: `/shops/{id}` تُرجع الـ info فقط، القائمة lazy-loaded |

---

## B. Caching Strategy

```
GET /home/feed          → Cache-Control: max-age=300  (5 دقائق)
GET /home/sliders       → Cache-Control: max-age=3600 (ساعة)
GET /home/categories    → Cache-Control: max-age=3600 (ساعة)
GET /cities             → Cache-Control: max-age=86400 (يوم)
GET /shops/{id}         → Cache-Control: max-age=600  (10 دقائق)
GET /products/{id}      → Cache-Control: max-age=600
GET /me/*               → Cache-Control: no-cache
POST, PATCH, DELETE     → Cache-Control: no-store
```

---

## C. Rate Limiting

| Endpoint Group | الحد |
|---------------|------|
| `POST /auth/login` | 5 محاولات / دقيقة |
| `POST /auth/otp/*` | 3 محاولات / 15 دقيقة |
| `GET /search` | 60 طلب / دقيقة |
| باقي الـ Endpoints | 120 طلب / دقيقة |

---

## D. Image URLs

> [!IMPORTANT]
> يجب أن كل الـ `*_url` fields في الـ Response تكون **روابط كاملة** (absolute URLs).  
> الـ Backend مسؤول عن بناء الرابط الكامل — الـ Frontend لا يبني أي URL يدوياً.

```json
// ✅ صح
"image_url": "https://sahlaapp.com/storage/products/burger.jpg"

// ❌ غلط
"image_url": "Photo/products/burger.jpg"
```

---

## E. Naming Consistency

| التوحيد | التفصيل |
|---------|---------|
| **معرفات المستخدمين** | دائماً `id` في الـ Response، الـ Backend يعرف من الـ Token |
| **المعرفات** | `snake_case` و `string` دائماً حتى لو الـ DB يخزنها int |
| **التواريخ** | ISO 8601 دائماً: `2026-08-10T14:00:00Z` |
| **الأسعار** | `float` دائماً (لا strings) |
| **الأعلام (Flags)** | `boolean` دائماً (لا `"YES"/"NO"` أو `"1"/"0"`) |
| **الحالات** | `lowercase` دائماً: `"active"`, `"pending"`, `"deleted"` |

---

## F. Soft Delete — Consistent Pattern

كل عمليات الحذف تتبع نفس النمط:

```sql
-- بدلاً من
DELETE FROM table WHERE id = ?

-- استخدم
UPDATE table SET status = 'deleted', deleted_at = NOW() WHERE id = ?
```

الـ Endpoints التي تطبق Soft Delete:
- `DELETE /me` (حذف الحساب)
- `DELETE /me/addresses/{id}` (حذف العنوان)
- `DELETE /reels/{id}/comments/{id}` (حذف التعليق)

---

## G. Endpoints Summary Table

| # | Method | URL | Auth | Description |
|---|--------|-----|------|-------------|
| 1 | `POST` | `/auth/register` | ❌ | تسجيل مستخدم جديد |
| 2 | `POST` | `/auth/login` | ❌ | تسجيل الدخول |
| 3 | `POST` | `/auth/social-login` | ❌ | دخول Google/Apple |
| 4 | `POST` | `/auth/otp/send` | ❌ | إرسال OTP |
| 5 | `POST` | `/auth/otp/verify` | ❌ | التحقق من OTP |
| 6 | `GET` | `/auth/check-phone` | ❌ | التحقق من رقم الهاتف |
| 7 | `POST` | `/auth/logout` | ✅ | تسجيل خروج |
| 8 | `POST` | `/auth/refresh` | ✅ | تجديد الـ Token |
| 9 | `GET` | `/home/sliders` | ❌ | الشرائح الإعلانية |
| 10 | `GET` | `/home/categories` | ❌ | التصنيفات الرئيسية |
| 11 | `GET` | `/home/offers` | ❌ | العروض |
| 12 | `GET` | `/home/best-shops` | ⬜ | أفضل المتاجر |
| 13 | `GET` | `/home/best-products` | ❌ | أفضل المنتجات |
| 14 | `GET` | `/shops/{id}` | ⬜ | تفاصيل المتجر |
| 15 | `GET` | `/shops/{id}/menu` | ❌ | قائمة المنتجات |
| 16 | `GET` | `/shops/{id}/reviews` | ❌ | تقييمات المتجر |
| 17 | `GET` | `/shops/{id}/videos` | ❌ | فيديوهات المتجر |
| 18 | `POST` | `/shops/{id}/favorite` | ✅ | Toggle المفضلة |
| 19 | `POST` | `/shops/{id}/ratings` | ✅ | تقييم المتجر |
| 20 | `GET` | `/me/favorite-shops` | ✅ | المتاجر المفضلة |
| 21 | `GET` | `/products/{id}` | ❌ | تفاصيل المنتج |
| 22 | `GET` | `/products/{id}/extras` | ❌ | إضافات المنتج |
| 23 | `GET` | `/products/{id}/similar` | ❌ | منتجات مشابهة |
| 24 | `POST` | `/cart/items` | ✅ | إضافة للسلة |
| 25 | `GET` | `/cart` | ✅ | عرض السلة |
| 26 | `GET` | `/cart/count` | ✅ | عدد عناصر السلة |
| 27 | `PATCH` | `/cart/items/{id}` | ✅ | تعديل الكمية |
| 28 | `DELETE` | `/cart/items/{id}` | ✅ | حذف من السلة |
| 29 | `GET` | `/cart/delivery-fee` | ✅ | حساب التوصيل |
| 30 | `POST` | `/orders` | ✅ | تأكيد الطلب |
| 31 | `GET` | `/orders` | ✅ | طلباتي |
| 32 | `GET` | `/orders/{id}` | ✅ | تفاصيل طلب |
| 33 | `POST` | `/orders/{id}/rating` | ✅ | تقييم الطلب |
| 34 | `POST` | `/orders/offers` | ✅ | طلب عرض |
| 35 | `GET` | `/me` | ✅ | البروفايل |
| 36 | `PATCH` | `/me` | ✅ | تعديل البروفايل |
| 37 | `POST` | `/me/profile-picture` | ✅ | تغيير الصورة |
| 38 | `PATCH` | `/me/password` | ✅ | تغيير كلمة المرور |
| 39 | `DELETE` | `/me` | ✅ | حذف الحساب (Soft) |
| 40 | `GET` | `/me/favorite-reels` | ✅ | الـ Reels المفضلة |
| 41 | `GET` | `/me/addresses` | ✅ | عناوين المستخدم |
| 42 | `POST` | `/me/addresses` | ✅ | إضافة عنوان |
| 43 | `PATCH` | `/me/addresses/{id}` | ✅ | تعديل عنوان |
| 44 | `DELETE` | `/me/addresses/{id}` | ✅ | حذف عنوان (Soft) |
| 45 | `PATCH` | `/me/addresses/{id}/default` | ✅ | تعيين عنوان افتراضي |
| 46 | `GET` | `/me/notifications` | ✅ | الإشعارات |
| 47 | `PATCH` | `/me/notifications/{id}/read` | ✅ | تعليم مقروء |
| 48 | `PATCH` | `/me/notifications/read-all` | ✅ | تعليم الكل مقروء |
| 49 | `GET` | `/cities` | ❌ | قائمة المدن |
| 50 | `POST` | `/support/contact` | ❌ | تواصل معنا |
| 51 | `GET` | `/search` | ❌ | البحث |
| 52 | `GET` | `/reels` | ✅ | الـ Reels Feed |
| 53 | `POST` | `/reels/{id}/like` | ✅ | Toggle الإعجاب |
| 54 | `GET` | `/reels/{id}/comments` | ✅ | تعليقات الـ Reel |
| 55 | `POST` | `/reels/{id}/comments` | ✅ | إضافة تعليق |
| 56 | `DELETE` | `/reels/{id}/comments/{id}` | ✅ | حذف تعليق (Soft) |
| 57 | `GET` | `/offers/{id}` | ❌ | تفاصيل العرض |
| 58 | `GET` | `/categories/{id}` | ❌ | تفاصيل التصنيف |

"Photo/vendors/logo.jpg"
→ https://sahlaapp.com/LEO/AdminApiLar/public/Photo/vendors/logo.jpg
```
