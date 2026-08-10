# 🛠️ Implementation Plan — DealsFoodApp User API

## Overview

بناء **User API** كامل لتطبيق DealsFoodApp بـ **pure PHP** (نفس نهج Merchant App)، يعمل على نفس قاعدة بيانات `Deals_DB`، ويُوفر 58 endpoint موثقة في `api_documentation.md`.

---

## ⚠️ ملاحظة مهمة

> [!IMPORTANT]
> **المشروع ليس Laravel** — الـ Merchant App مبني بـ **pure PHP** (بدون framework) مع classes مخصصة لـ Database, Controller, Middleware. سنتبع نفس النمط بالضبط لأن:
> - نفس قاعدة البيانات مشتركة
> - نفس نمط الـ Token-based auth في جدول `user_tokens`
> - نفس نمط الـ Routing اليدوي

> [!IMPORTANT]
> **جداول إضافية مطلوبة** — جدول `users` موجود لكن هناك جداول تحتاج إنشاء:
> - `user_addresses` — لعناوين المستخدمين
> - `user_otps` — للـ OTP codes
> - `cart_items` — لعناصر السلة
> - `user_favorites` — للمتاجر المفضلة
> - `reel_likes` — للإعجابات على الـ Reels
> - `user_notifications` — إشعارات العميل (الموجودة `notifications` للـ Merchant)
> - `support_messages` — للتواصل

> [!WARNING]
> **الجداول الموجودة بالفعل:** `users`, `user_tokens`, `orders`, `order_items`, `restaurants`, `menu_items`, `menu_sections`, `categories`, `reels`, `reel_comments`, `reviews`, `notifications` — لا نعدل عليها بل نضيف جداول جديدة فقط.

---

## Open Questions

> [!IMPORTANT]
> **جدول `users`** — الـ Column الحالي اسمه `name` (وليس `first_name` + `last_name`)، هل نضيف columns جديدة أم نستخدم `name` ونُقسمه؟ **الاقتراح:** نضيف `first_name`, `last_name`, `auth_method`, `latitude`, `longitude`, `wallet_balance`, `is_verified`, `app_language` كـ ALTER TABLE.

> [!IMPORTANT]
> **جدول `notifications`** — الموجود مرتبط بـ `merchant_users` وليس `users`. نحتاج جدول `user_notifications` منفصل للعملاء.

---

## Proposed Changes

### المرحلة 1 — هيكل المشروع

```
DealsUsers/
├── api/
│   └── v1/
│       └── index.php          ← Router رئيسي
├── config/
│   ├── database.php           ← نسخة من Merchant (نفس DB)
│   ├── env.php                ← نسخة من Merchant
│   └── cors.php               ← CORS headers
├── middleware/
│   ├── AuthMiddleware.php     ← Customer-only auth
│   └── RateLimiter.php        ← Rate limiting
├── controllers/
│   ├── Controller.php         ← Base controller
│   ├── Auth/
│   │   └── AuthController.php
│   ├── HomeController.php
│   ├── ShopController.php
│   ├── ProductController.php
│   ├── CartController.php
│   ├── OrderController.php
│   ├── ProfileController.php
│   ├── AddressController.php
│   ├── NotificationController.php
│   ├── SearchController.php
│   ├── ReelController.php
│   ├── OfferController.php
│   └── CategoryController.php
├── migrations/
│   └── user_api_tables.sql    ← الجداول الجديدة
├── Photos/                    ← صور البروفايل (موجودة)
└── .env                       ← نفس بيانات DB
```

---

### المرحلة 2 — قاعدة البيانات (Migrations)

#### [MODIFY] جدول `users` — إضافة columns جديدة

```sql
ALTER TABLE `users`
    ADD COLUMN `first_name` VARCHAR(75) DEFAULT NULL AFTER `name`,
    ADD COLUMN `last_name` VARCHAR(75) DEFAULT NULL AFTER `first_name`,
    ADD COLUMN `auth_method` ENUM('phone','google','apple') NOT NULL DEFAULT 'phone' AFTER `last_name`,
    ADD COLUMN `social_id` VARCHAR(255) DEFAULT NULL AFTER `auth_method`,
    ADD COLUMN `latitude` DECIMAL(10,8) DEFAULT NULL AFTER `social_id`,
    ADD COLUMN `longitude` DECIMAL(11,8) DEFAULT NULL AFTER `latitude`,
    ADD COLUMN `wallet_balance` DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    ADD COLUMN `is_verified` TINYINT(1) NOT NULL DEFAULT 0,
    ADD COLUMN `app_language` ENUM('ar','en') NOT NULL DEFAULT 'ar';
```

#### [NEW] جداول جديدة

- `user_otps` — تخزين OTP codes مع expiry
- `user_addresses` — عناوين المستخدمين
- `cart_items` — عناصر السلة (مع extras كـ JSON)
- `user_favorites` — pivot للمتاجر المفضلة
- `reel_likes` — إعجابات الـ Reels
- `user_notifications` — إشعارات العميل
- `support_messages` — رسائل الدعم
- `order_ratings` — تقييمات الطلبات

---

### المرحلة 3 — Controllers (58 Endpoint)

#### [NEW] AuthController.php
- `POST /auth/register` — تسجيل مستخدم جديد
- `POST /auth/login` — تسجيل دخول
- `POST /auth/social-login` — Google/Apple login
- `POST /auth/otp/send` — إرسال OTP
- `POST /auth/otp/verify` — التحقق من OTP
- `GET /auth/check-phone` — التحقق من رقم الهاتف
- `POST /auth/logout` — تسجيل خروج
- `POST /auth/refresh` — تجديد token

#### [NEW] HomeController.php
- `GET /home/sliders` — الشرائح الإعلانية
- `GET /home/categories` — التصنيفات
- `GET /home/offers` — العروض
- `GET /home/best-shops` — أفضل المتاجر
- `GET /home/best-products` — أفضل المنتجات

#### [NEW] ShopController.php
- `GET /shops/{id}` — تفاصيل المتجر
- `GET /shops/{id}/menu` — قائمة المنتجات
- `GET /shops/{id}/reviews` — التقييمات
- `GET /shops/{id}/videos` — الفيديوهات
- `POST /shops/{id}/favorite` — Toggle المفضلة
- `POST /shops/{id}/ratings` — تقييم المتجر
- `GET /me/favorite-shops` — المتاجر المفضلة

#### [NEW] ProductController.php
- `GET /products/{id}` — تفاصيل المنتج
- `GET /products/{id}/extras` — الإضافات
- `GET /products/{id}/similar` — منتجات مشابهة

#### [NEW] CartController.php
- `POST /cart/items` — إضافة للسلة
- `GET /cart` — عرض السلة
- `GET /cart/count` — عدد العناصر
- `PATCH /cart/items/{id}` — تعديل الكمية
- `DELETE /cart/items/{id}` — حذف من السلة
- `GET /cart/delivery-fee` — حساب رسوم التوصيل

#### [NEW] OrderController.php
- `POST /orders` — تأكيد الطلب
- `GET /orders` — طلباتي
- `GET /orders/{id}` — تفاصيل طلب
- `POST /orders/{id}/rating` — تقييم الطلب
- `POST /orders/offers` — طلب عرض

#### [NEW] ProfileController.php
- `GET /me` — البروفايل
- `PATCH /me` — تعديل البروفايل
- `POST /me/profile-picture` — رفع صورة
- `PATCH /me/password` — تغيير كلمة المرور
- `DELETE /me` — حذف الحساب (Soft Delete)
- `GET /me/favorite-reels` — الـ Reels المفضلة

#### [NEW] AddressController.php
- `GET /me/addresses` — العناوين
- `POST /me/addresses` — إضافة عنوان
- `PATCH /me/addresses/{id}` — تعديل عنوان
- `DELETE /me/addresses/{id}` — حذف عنوان (Soft)
- `PATCH /me/addresses/{id}/default` — تعيين افتراضي
- `GET /cities` — قائمة المدن

#### [NEW] NotificationController.php
- `GET /me/notifications` — الإشعارات
- `PATCH /me/notifications/{id}/read` — تعليم مقروء
- `PATCH /me/notifications/read-all` — تعليم الكل مقروء

#### [NEW] SearchController.php
- `GET /search` — البحث

#### [NEW] ReelController.php
- `GET /reels` — الـ Reels Feed
- `POST /reels/{id}/like` — Toggle الإعجاب
- `GET /reels/{id}/comments` — التعليقات
- `POST /reels/{id}/comments` — إضافة تعليق
- `DELETE /reels/{id}/comments/{id}` — حذف تعليق (Soft)

#### [NEW] OfferController.php
- `GET /offers/{id}` — تفاصيل العرض

#### [NEW] CategoryController.php
- `GET /categories/{id}` — تفاصيل التصنيف

#### [NEW] SupportController.php
- `POST /support/contact` — تواصل معنا

---

### المرحلة 4 — Middleware & Infrastructure

#### [NEW] AuthMiddleware.php — Customer-only
- يتحقق من `user_tokens` فقط (ليس `merchant_user_tokens`)
- يُرجع بيانات المستخدم من `users` table

#### [NEW] RateLimiter.php
- Auth: 5 requests/minute
- OTP: 3 requests/15 minutes
- Search: 60 requests/minute
- API: 120 requests/minute

#### [NEW] Router (index.php)
- يربط الـ URLs بالـ Controllers
- يدعم Path Parameters مثل `/shops/{id}`
- يدعم CORS headers

---

## Verification Plan

### الجداول الجديدة
- التأكد من إنشاء الجداول بدون أخطاء
- التأكد من عدم تأثير الـ ALTER TABLE على بيانات Merchant App

### Testing بالترتيب
1. `POST /auth/register` → يُنشئ user وtoken
2. `POST /auth/login` → يُعيد token
3. `GET /home/sliders` → يُعيد sliders من DB
4. `GET /shops/{id}` → يُعيد بيانات restaurant
5. `POST /cart/items` → يضيف للسلة
6. `POST /orders` → يُنشئ order

### Response Format
- كل الـ responses تتبع الـ Standard Envelope
- الـ IDs دائماً `string`
- الصور دائماً Absolute URLs
- التواريخ ISO 8601
