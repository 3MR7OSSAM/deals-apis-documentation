# 🛠️ Implementation Plan — DealsFoodApp User API (Laravel)

## نبذة عن المشروع
بناء **User API** كامل لتطبيق DealsFoodApp باستخدام **Laravel + Sanctum**، يشترك في نفس قاعدة البيانات `Deals_DB` مع التطبيق الموجود للتاجر (PHP خام — ليس Laravel).

---

## 🔍 ملاحظات مهمة بعد قراءة الملفات

### قاعدة البيانات (Deals_DB.sql)
الجداول الرئيسية المرتبطة بالـ User API:

| الجدول | الوصف |
|--------|-------|
| `users` | المستخدمون — يحتوي: `id, name, email, phone_key, phone, password_hash, profile_image, fcm_token, is_active, deleted_at` |
| `user_tokens` | توكنات المستخدمين (custom tokens — **ليس Sanctum**) |
| `restaurants` | المطاعم/المتاجر — يحتوي: `id, owner_id, name_ar, name_en, address, latitude, longitude, phone, logo, cover_image, is_online, is_active, deleted_at` |
| `menu_sections` | أقسام المنيو |
| `menu_items` | المنتجات — يحتوي: `id, merchant_user_id, section_id, name_ar, name_en, price, image, is_active, deleted_at` |
| `extra_groups` | مجموعات الإضافات |
| `extra_items` | عناصر الإضافات |
| `menu_item_extra_groups` | ربط المنتجات بالإضافات |
| `orders` | الطلبات — يحتوي: `id, merchant_user_id, restaurant_id, customer_id, customer_name, customer_phone, customer_address, customer_latitude, customer_longitude, total_amount, delivery_fee, status, deleted_at` |
| `order_items` | عناصر الطلب — `order_id, menu_item_id, quantity, price` |
| `order_item_extras` | الإضافات في الطلب |
| `reviews` | التقييمات — `id, user_id, restaurant_id, menu_item_id, rating, comment` |
| `reels` | الريلز — `id, merchant_user_id, video_url, thumbnail_url, title_ar, title_en, likes_count, comments_count, is_visible, deleted_at` |
| `reel_comments` | تعليقات الريلز — `id, reel_id, parent_id, user_id, customer_name, comment_text` |
| `notifications` | الإشعارات (حالياً مرتبطة بـ merchant_users — ستحتاج جدول جديد للـ users) |
| `categories` | التصنيفات — `id, name_ar, name_en, image, is_active` |
| `restaurant_categories` | ربط المطاعم بالتصنيفات |
| `restaurant_working_hours` | أوقات العمل |

### ⚠️ نقاط تحتاج قرار
> [!IMPORTANT]
> **جدول الإشعارات الحالي (`notifications`)** مرتبط بـ `merchant_users`، لكن الـ User API يحتاج إشعارات للـ users العاديين.
> **القرار المقترح:** إنشاء جدول `user_notifications` جديد منفصل للـ users.

> [!IMPORTANT]
> **جدول `users`** يحتوي `name` (اسم واحد)، لكن الـ API يتوقع `first_name + last_name`.
> **القرار المقترح:** إضافة عمود `first_name` و`last_name` أو تقسيم `name` ديناميكياً في الـ Resource.
> سنضيف migration يضيف الأعمدة الناقصة للـ `users` table.

> [!IMPORTANT]
> **`user_tokens`** جدول Custom Tokens موجود — سنستخدم **Laravel Sanctum** بدلاً منه (يُنشئ جدول `personal_access_tokens` منفصل).

> [!IMPORTANT]
> **الـ Sliders**: لا يوجد جدول `sliders` في الـ DB الحالي.
> **القرار المقترح:** إنشاء migration لجدول `sliders` جديد.

> [!IMPORTANT]
> **الـ Offers**: لا يوجد جدول `offers` مستقل في الـ DB الحالي.
> **القرار المقترح:** إنشاء migration لجدول `offers` جديد.

> [!IMPORTANT]
> **الـ user_addresses**: لا يوجد جدول عناوين للمستخدمين.
> **القرار المقترح:** إنشاء migration لجدول `user_addresses`.

> [!IMPORTANT]
> **الـ Cart**: لا يوجد جدول سلة تسوق.
> **القرار المقترح:** إنشاء migration لجدول `user_carts` و`user_cart_items`.

> [!IMPORTANT]
> **الـ user_favorites**: لا يوجد جدول للمتاجر المفضلة.
> **القرار المقترح:** إنشاء migration لجدول `user_favorite_shops` و`user_favorite_reels`.

> [!IMPORTANT]
> **الـ OTPs**: لا يوجد جدول OTP.
> **القرار المقترح:** إنشاء migration لجدول `user_otps`.

> [!IMPORTANT]
> **الـ cities**: لا يوجد جدول مدن.
> **القرار المقترح:** إنشاء migration لجدول `cities` وإضافة بيانات أولية.

> [!IMPORTANT]
> **الـ reel_likes**: لا يوجد جدول لإعجابات الريلز.
> **القرار المقترح:** إنشاء migration لجدول `user_reel_likes`.

> [!IMPORTANT]
> **الـ support/contact**: لا يوجد جدول دعم للمستخدم.
> **القرار المقترح:** إنشاء migration لجدول `user_support_messages`.

---

## 📁 هيكل المشروع المقترح

```
E:\EngazTechnology\Deals User Group\DealsUsers\   ← Laravel project جديد
├── app/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   │   ├── Auth/
│   │   │   │   ├── AuthController.php
│   │   │   │   └── OtpController.php
│   │   │   ├── HomeController.php
│   │   │   ├── ShopController.php
│   │   │   ├── ProductController.php
│   │   │   ├── CartController.php
│   │   │   ├── OrderController.php
│   │   │   ├── ProfileController.php
│   │   │   ├── AddressController.php
│   │   │   ├── NotificationController.php
│   │   │   ├── SearchController.php
│   │   │   ├── ReelController.php
│   │   │   ├── OfferController.php
│   │   │   └── CategoryController.php
│   │   ├── Requests/Api/V1/         ← Form Requests
│   │   └── Resources/Api/V1/        ← API Resources
│   ├── Models/
│   │   ├── User.php
│   │   ├── Restaurant.php
│   │   ├── MenuItem.php
│   │   ├── MenuSection.php
│   │   ├── ExtraGroup.php
│   │   ├── ExtraItem.php
│   │   ├── Order.php
│   │   ├── OrderItem.php
│   │   ├── Reel.php
│   │   ├── ReelComment.php
│   │   ├── Review.php
│   │   ├── Category.php
│   │   ├── Slider.php
│   │   ├── Offer.php
│   │   ├── UserAddress.php
│   │   ├── UserCart.php
│   │   ├── UserCartItem.php
│   │   ├── UserFavoriteShop.php
│   │   ├── UserFavoriteReel.php
│   │   ├── UserRealLike.php
│   │   ├── UserNotification.php
│   │   ├── UserOtp.php
│   │   ├── City.php
│   │   └── UserSupportMessage.php
│   └── Traits/
│       └── ApiResponse.php          ← Trait للـ Response الموحد
├── database/
│   └── migrations/                  ← Migrations للجداول الجديدة فقط
├── routes/
│   └── api.php
└── config/
    └── sanctum.php
```

---

## 🗂️ خطوات التنفيذ

### المرحلة 1 — Setup المشروع
1. إنشاء Laravel project جديد في `DealsUsers/`
2. تثبيت Laravel Sanctum
3. ربط نفس الـ DB (`Deals_DB`) في `.env`
4. إنشاء `ApiResponse` Trait

### المرحلة 2 — Migrations (للجداول الجديدة فقط)
إنشاء migrations للجداول التالية (لا تمس الجداول الموجودة):
- `alter_users_add_missing_columns` → يضيف `first_name, last_name, auth_method, latitude, longitude, app_language`
- `create_sliders_table`
- `create_offers_table` + `create_offer_products_table`
- `create_user_addresses_table`
- `create_user_carts_table` + `create_user_cart_items_table`
- `create_user_favorite_shops_table`
- `create_user_favorite_reels_table`
- `create_user_reel_likes_table`
- `create_user_notifications_table`
- `create_user_otps_table`
- `create_cities_table`
- `create_user_support_messages_table`

### المرحلة 3 — Models & Relationships
إنشاء جميع Models مع الـ Relationships الصحيحة.

### المرحلة 4 — Rate Limiters & Middleware
إعداد الـ Rate Limiters في `RouteServiceProvider`:
- `auth` → 5 req/min
- `otp` → 3 req/15min
- `search` → 60 req/min
- `api` → 120 req/min

### المرحلة 5 — Routes (api.php)
تعريف جميع الـ Routes منظمة في groups.

### المرحلة 6 — تنفيذ الـ Endpoints بالترتيب

#### أولاً: Auth (Priority 1)
| # | Endpoint | Method | Notes |
|---|----------|--------|-------|
| 1.1 | `/api/v1/auth/register` | POST | Phone uniqueness check → 409 |
| 1.2 | `/api/v1/auth/login` | POST | Rate limited |
| 1.3 | `/api/v1/auth/social-login` | POST | Google/Apple — upsert |
| 1.4 | `/api/v1/auth/otp/send` | POST | Generate 4-digit OTP, store in `user_otps` |
| 1.5 | `/api/v1/auth/otp/verify` | POST | Check OTP + expiry |
| 1.6 | `/api/v1/auth/check-phone` | GET | Returns `available: bool` |
| 1.7 | `/api/v1/auth/logout` | POST | Revoke current token |
| 1.8 | `/api/v1/auth/refresh` | POST | Delete + create new Sanctum token |

#### ثانياً: Home (Priority 1)
| # | Endpoint | Method | Cache |
|---|----------|--------|-------|
| 2.1 | `/api/v1/home/sliders` | GET | 1hr |
| 2.2 | `/api/v1/home/categories` | GET | 1hr |
| 2.3 | `/api/v1/home/offers` | GET | 5min + Paginated |
| 2.4 | `/api/v1/home/best-shops` | GET | 5min + Paginated + optional lat/lng |
| 2.5 | `/api/v1/home/best-products` | GET | 5min + Paginated |

#### ثالثاً: Shops
| # | Endpoint | Auth |
|---|----------|------|
| 3.1 | `GET /api/v1/shops/{id}` | Optional |
| 3.2 | `GET /api/v1/shops/{id}/menu` | Public |
| 3.3 | `GET /api/v1/shops/{id}/reviews` | Public |
| 3.4 | `GET /api/v1/shops/{id}/videos` | Public |
| 3.5 | `POST /api/v1/shops/{id}/favorite` | 🔒 Toggle |
| 3.6 | `POST /api/v1/shops/{id}/ratings` | 🔒 |
| 3.7 | `GET /api/v1/me/favorite-shops` | 🔒 |

#### رابعاً: Products
| # | Endpoint | Auth |
|---|----------|------|
| 4.1 | `GET /api/v1/products/{id}` | Public |
| 4.2 | `GET /api/v1/products/{id}/extras` | Public |
| 4.3 | `GET /api/v1/products/{id}/similar` | Public |

#### خامساً: Cart & Orders
| # | Endpoint | Auth | Notes |
|---|----------|------|-------|
| 5.1 | `POST /api/v1/cart/items` | 🔒 | Cart Isolation — vendor check |
| 5.2 | `GET /api/v1/cart` | 🔒 |  |
| 5.3 | `GET /api/v1/cart/count` | 🔒 |  |
| 5.4 | `PATCH /api/v1/cart/items/{id}` | 🔒 |  |
| 5.5 | `DELETE /api/v1/cart/items/{id}` | 🔒 | 204 |
| 5.6 | `GET /api/v1/cart/delivery-fee` | 🔒 | Haversine formula |
| 5.7 | `POST /api/v1/orders` | 🔒 | Place order from cart |
| 6.1 | `GET /api/v1/orders` | 🔒 | Filter by status |
| 6.2 | `GET /api/v1/orders/{id}` | 🔒 |  |
| 6.3 | `POST /api/v1/orders/{id}/rating` | 🔒 |  |
| 6.4 | `POST /api/v1/orders/offers` | 🔒 | Order from offer directly |

#### سادساً: Profile (Me)
| # | Endpoint | Auth |
|---|----------|------|
| 7.1 | `GET /api/v1/me` | 🔒 |
| 7.2 | `PATCH /api/v1/me` | 🔒 |
| 7.3 | `POST /api/v1/me/profile-picture` | 🔒 multipart |
| 7.4 | `PATCH /api/v1/me/password` | 🔒 |
| 7.5 | `DELETE /api/v1/me` | 🔒 Soft Delete |
| 7.6 | `GET /api/v1/me/favorite-reels` | 🔒 |

#### سابعاً: Addresses
| # | Endpoint | Auth |
|---|----------|------|
| 8.1 | `GET /api/v1/me/addresses` | 🔒 |
| 8.2 | `POST /api/v1/me/addresses` | 🔒 |
| 8.3 | `PATCH /api/v1/me/addresses/{id}` | 🔒 |
| 8.4 | `DELETE /api/v1/me/addresses/{id}` | 🔒 Soft |
| 8.5 | `PATCH /api/v1/me/addresses/{id}/default` | 🔒 |
| 8.6 | `GET /api/v1/cities` | Public |

#### ثامناً: Notifications
| # | Endpoint | Auth | Pagination |
|---|----------|------|-----------|
| 9.1 | `GET /api/v1/me/notifications` | 🔒 | Cursor-based |
| 9.2 | `PATCH /api/v1/me/notifications/{id}/read` | 🔒 |  |
| 9.3 | `PATCH /api/v1/me/notifications/read-all` | 🔒 |  |

#### تاسعاً: Search
| # | Endpoint | Auth | Rate Limit |
|---|----------|------|-----------|
| 10.1 | `GET /api/v1/search` | Public | 60/min |

#### عاشراً: Reels
| # | Endpoint | Auth | Pagination |
|---|----------|------|-----------|
| 11.1 | `GET /api/v1/reels` | 🔒 | Cursor-based |
| 11.2 | `POST /api/v1/reels/{id}/like` | 🔒 | Toggle |
| 11.3 | `GET /api/v1/reels/{id}/comments` | 🔒 | Cursor-based |
| 11.4 | `POST /api/v1/reels/{id}/comments` | 🔒 |  |
| 11.5 | `DELETE /api/v1/reels/{id}/comments/{cid}` | 🔒 | Soft |

#### حادي عشر: Misc
| # | Endpoint | Auth |
|---|----------|------|
| 12.1 | `GET /api/v1/offers/{id}` | Public |
| 13.1 | `GET /api/v1/categories/{id}` | Public |
| 14.1 | `POST /api/v1/support/contact` | Public |

---

## 🔒 Business Logic المهمة

### Cart Isolation
```
عند إضافة item من vendor مختلف عن الـ vendor الموجود في السلة:
→ رجع 409 Conflict مع رسالة: "Cart contains items from another shop. Clear cart first."
```

### Delivery Fee Calculation (Haversine)
```
استخدام Haversine formula لحساب المسافة بين:
- إحداثيات العنوان المختار (lat/lng من الـ request)
- إحداثيات أقرب فرع (restaurant.latitude/longitude)
ثم تطبيق سعر التوصيل المناسب.
```

### Image URLs
```php
// المنتجات والمتاجر
'https://dealsapps.net/DealsAppsGroup/DealsMerchant/public/' . $path

// صور الـ Users
'https://dealsapps.net/DealsAppsGroup/DealsUsers/Photos/' . $filename
```

### OTP Flow
```
1. POST /auth/otp/send → generate 4-digit code
2. Store in user_otps: {phone_key, phone, code, expires_at: now()+5min}
3. Send via SMS/WhatsApp (log only في البيئة التطويرية)
4. POST /auth/otp/verify → check code + not expired
5. Mark user as verified → update users.is_active = true (if needed)
```

### Soft Delete Pattern
```php
// users: users.deleted_at = now(), revoke all tokens
// addresses: user_addresses.deleted_at = now()
// reel_comments: reel_comments.deleted_at = now() (نضيف العمود)
```

---

## 📋 خطة الـ Migrations الجديدة

| Migration | الجداول الجديدة / التعديلات |
|-----------|--------------------------|
| `2026_08_10_000001_alter_users_table` | يضيف: `first_name, last_name, auth_method (enum), latitude, longitude, app_language` |
| `2026_08_10_000002_create_sliders_table` | `id, image, text_ar, text_en, action_type, action_value, slide_type, expires_at, is_active` |
| `2026_08_10_000003_create_offers_table` | `id, merchant_user_id, title_ar, title_en, description_ar, description_en, image, color, status, deleted_at` |
| `2026_08_10_000004_create_offer_products_table` | `id, offer_id, menu_item_id, offer_price, quantity` |
| `2026_08_10_000005_create_user_addresses_table` | `id, user_id, label, city, latitude, longitude, street_name, building_number, floor_number, apartment_number, landmark, is_default, deleted_at` |
| `2026_08_10_000006_create_user_carts_table` | `id, user_id, vendor_id, created_at` |
| `2026_08_10_000007_create_user_cart_items_table` | `id, cart_id, menu_item_id, quantity, unit_price` |
| `2026_08_10_000008_create_user_cart_item_extras_table` | `id, cart_item_id, extra_item_id` |
| `2026_08_10_000009_create_user_favorite_shops_table` | `id, user_id, restaurant_id, created_at` |
| `2026_08_10_000010_create_user_favorite_reels_table` | `id, user_id, reel_id, created_at` |
| `2026_08_10_000011_create_user_reel_likes_table` | `id, user_id, reel_id, created_at` |
| `2026_08_10_000012_create_user_notifications_table` | `id, user_id, type, title_ar, title_en, body_ar, body_en, action_type, action_id, is_read, created_at` |
| `2026_08_10_000013_create_user_otps_table` | `id, phone_key, phone, code, expires_at, is_used, created_at` |
| `2026_08_10_000014_create_cities_table` | `id, name_ar, name_en, is_active` |
| `2026_08_10_000015_create_user_support_messages_table` | `id, name, email, phone, message, created_at` |
| `2026_08_10_000016_alter_reel_comments_add_deleted_at` | يضيف `deleted_at` لجدول `reel_comments` |

---

## ✅ Verification Plan

### Automated
```bash
php artisan test --filter=ApiTest
```

### Manual (Postman)
- [ ] Auth endpoints — Register/Login/OTP
- [ ] Protected endpoints ترفض بدون Token → 401
- [ ] Pagination يعمل مع `?page=2&per_page=5`
- [ ] Soft Delete — السجل موجود في DB بعد الحذف
- [ ] Cart Isolation — خطأ عند إضافة من vendor مختلف
- [ ] Image URLs كاملة (Absolute)
- [ ] Response Format موحد في جميع الحالات

---

## 📌 ملاحظات ختامية

> [!NOTE]
> الـ Merchant App الموجود هو **PHP خام** وليس Laravel — لذا لن يكون هناك تعارض في الـ Models أو Config.

> [!NOTE]
> سيتم تشغيل المشروع الجديد في **نفس قاعدة البيانات** `Deals_DB` دون المساس بالبيانات الموجودة.

> [!NOTE]
> الـ OTP سيُرسل عبر **SMS/WhatsApp** — في بيئة التطوير سيُلوجَّر فقط دون إرسال فعلي. يمكن لاحقاً ربطه بـ Twilio أو Vonage.

> [!WARNING]
> جدول `notifications` الموجود مرتبط بـ `merchant_users` — سنُنشئ `user_notifications` منفصل تماماً ولن نمس الجدول الأصلي.
