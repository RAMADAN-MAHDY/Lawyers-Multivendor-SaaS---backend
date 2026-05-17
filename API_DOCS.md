# 📚 API Documentation — Central Domain (Super Admin Panel)

> **Base URL:** `https://{central-domain}/api`  
> **Auth:** Bearer Token (Sanctum) — يُضاف في الـ Header لكل request محمي  
> **Content-Type:** `application/json`

---

## 🔐 Setup Axios Instance

```js
// api.js
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://your-central-domain.com/api',
  headers: { 'Content-Type': 'application/json' },
});

// إضافة التوكن تلقائياً لكل request
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

export default api;
```

---

## 🟢 1. Auth — المصادقة (Public Routes)

---

### `POST /register` — تسجيل مكتب جديد

**الوصف:** يُسجّل مستخدم جديد ويُقدّم طلب لإنشاء مكتب محاماة، يظل في حالة `pending` حتى يوافق Super Admin.

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | ✅ | اسم المستخدم |
| `email` | string | ✅ | بريد إلكتروني فريد |
| `password` | string | ✅ | minimum 8 chars |
| `password_confirmation` | string | ✅ | يجب يطابق `password` |
| `tenant_name` | string | ✅ | اسم المكتب — فريد، max 50 chars |

**مثال:**
```js
await api.post('/register', {
  name: 'محمد أحمد',
  email: 'mohamed@example.com',
  password: 'secret123',
  password_confirmation: 'secret123',
  tenant_name: 'nabil-law',
});
```

**Response `201`:**
```json
{
  "status": true,
  "message": "تم استلام طلبك، في انتظار موافقة السوبر أدمن لتفعيل مكتبك.",
  "user": {
    "id": 5,
    "name": "محمد أحمد",
    "email": "mohamed@example.com",
    "status": "pending",
    "requested_tenant_name": "nabil-law",
    "created_at": "2025-01-01T10:00:00Z"
  }
}
```

> ⚠️ إذا كان أول مستخدم في السيستم، يُصبح تلقائياً `super_admin` وحالته `approved`.

---

### `POST /login` — تسجيل الدخول

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `email` | string | ✅ | |
| `password` | string | ✅ | |
| `latitude` | number | ❌ | موقع جغرافي اختياري |
| `longitude` | number | ❌ | موقع جغرافي اختياري |
| `biometric_key` | string | ❌ | مفتاح البصمة |

**مثال:**
```js
const { data } = await api.post('/login', {
  email: 'admin@example.com',
  password: 'secret123',
});
localStorage.setItem('token', data.access_token);
```

**Response `200`:**
```json
{
  "status": true,
  "access_token": "1|xxxxxxxxxxxxxxxx",
  "user": {
    "id": 1,
    "name": "محمد أحمد",
    "email": "admin@example.com",
    "status": "approved",
    "tenant_id": "nabil-law",
    "all_permissions": ["view_users", "create_users", "edit_users"],
    "all_roles": ["super_admin"]
  },
  "subscription": {
    "id": 3,
    "status": "active",
    "expires_at": "2026-01-01T00:00:00Z",
    "plan": {
      "id": 1,
      "name": "الباقة الذهبية",
      "max_users": 10,
      "max_cases": 100
    }
  }
}
```

> **ملاحظة:** `subscription` يكون `null` لو المستخدم super_admin أو ليس له مكتب بعد.

**Response `401`:** بيانات خاطئة  
**Response `403`:** الحساب لم يُوافق عليه بعد

---

### `POST /forgot-password` — نسيت كلمة المرور

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `email` | string | ✅ |

**مثال:**
```js
await api.post('/forgot-password', { email: 'user@example.com' });
```

**Response `200`:**
```json
{
  "status": true,
  "message": "تم إرسال رمز التحقق إلى بريدك الإلكتروني بنجاح."
}
```

> OTP صالح لمدة **15 دقيقة** فقط.

---

### `POST /verify-otp` — التحقق من الكود

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `email` | string | ✅ | |
| `otp` | number | ✅ | 4 أرقام |

**مثال:**
```js
await api.post('/verify-otp', { email: 'user@example.com', otp: 1234 });
```

**Response `200`:**
```json
{ "status": true, "message": "رمز التحقق صحيح." }
```

**Response `400`:** الكود غلط أو منتهي

---

### `POST /reset-password` — إعادة تعيين كلمة المرور

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `email` | string | ✅ |
| `otp` | number | ✅ |
| `password` | string | ✅ |
| `password_confirmation` | string | ✅ |

**مثال:**
```js
await api.post('/reset-password', {
  email: 'user@example.com',
  otp: 1234,
  password: 'newSecret123',
  password_confirmation: 'newSecret123',
});
```

**Response `200`:**
```json
{
  "status": true,
  "message": "تم تغيير كلمة المرور بنجاح ويرجى تسجيل الدخول."
}
```

---

### `POST /contact-us` — تواصل معنا

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `name` | string | ✅ |
| `email` | string | ✅ |
| `subject` | string | ✅ |
| `message` | string | ✅ |

**مثال:**
```js
await api.post('/contact-us', {
  name: 'أحمد',
  email: 'ahmed@example.com',
  subject: 'استفسار',
  message: 'أريد الاستفسار عن الباقات.',
});
```

**Response `200`:**
```json
{
  "status": true,
  "message": "تم استلام رسالتك بنجاح، شكرًا لتواصلك معنا!"
}
```

---

## 🔒 2. Auth — محمية بـ Token (auth:sanctum)

> أضف `Authorization: Bearer {token}` في كل الـ requests التالية.

---

### `POST /logout` — تسجيل الخروج

**مثال:**
```js
await api.post('/logout');
localStorage.removeItem('token');
```

**Response `200`:**
```json
{ "status": true }
```

---

### `POST /change-password` — تغيير كلمة المرور

**Request Body:**
| Field | Type | Required |
|-------|------|----------|
| `current_password` | string | ✅ |
| `new_password` | string | ✅ |
| `new_password_confirmation` | string | ✅ |

**مثال:**
```js
await api.post('/change-password', {
  current_password: 'old123',
  new_password: 'new123456',
  new_password_confirmation: 'new123456',
});
```

**Response `200`:**
```json
{
  "status": true,
  "message": "تم تغيير كلمة المرور بنجاح",
  "access_token": "2|yyyyyyyyyyyy"
}
```

> ⚠️ التوكن القديم يُحذف وبيرجع توكن جديد — حدّث التوكن المخزن.

**Response `400`:** كلمة المرور الحالية غلط

---

## 👑 3. Admin Routes — خاصة بـ Super Admin فقط

> `Authorization: Bearer {token}` + الحساب لازم role = `super_admin`

---

### `GET /admin/pending-vendors` — طلبات المكاتب المعلقة

**مثال:**
```js
const { data } = await api.get('/admin/pending-vendors');
```

**Response `200`:**
```json
{
  "status": true,
  "data": [
    {
      "id": 5,
      "name": "محمد أحمد",
      "email": "mo@example.com",
      "status": "pending",
      "requested_tenant_name": "nabil-law",
      "created_at": "2025-01-01T10:00:00Z"
    }
  ]
}
```

---

### `POST /admin/approve-vendor/{userId}` — قبول طلب مكتب

**مثال:**
```js
await api.post('/admin/approve-vendor/5');
```

**Response `200`:**
```json
{
  "status": true,
  "message": "تم تفعيل مكتب المحاماة بنجاح، ويمكن للمستخدم الدخول الآن."
}
```

> يُنشئ Tenant جديد في قاعدة البيانات ويعطي المستخدم دور `owner`.

**Response `400`:** الطلب تم معالجته مسبقاً

---

### `POST /admin/reject-vendor/{userId}` — رفض طلب مكتب

**مثال:**
```js
await api.post('/admin/reject-vendor/5');
```

**Response `200`:**
```json
{
  "status": true,
  "message": "تم رفض طلب إنشاء المكتب."
}
```

---

### `POST /admin/profile/update` — تحديث بروفايل السوبر أدمن

**Content-Type:** `multipart/form-data` (لو فيه صورة)

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | ❌ | |
| `email` | string | ❌ | |
| `latitude` | number | ❌ | |
| `longitude` | number | ❌ | |
| `profile_image` | file | ❌ | jpeg/png/jpg/gif, max 2MB |

**مثال (بدون صورة):**
```js
await api.post('/admin/profile/update', {
  name: 'اسم جديد',
  email: 'new@example.com',
});
```

**مثال (مع صورة):**
```js
const formData = new FormData();
formData.append('name', 'اسم جديد');
formData.append('profile_image', fileInput.files[0]);

await api.post('/admin/profile/update', formData, {
  headers: { 'Content-Type': 'multipart/form-data' },
});
```

**Response `200`:**
```json
{
  "status": true,
  "user": {
    "id": 1,
    "name": "اسم جديد",
    "email": "new@example.com",
    "profile_image": "profiles/xxxxxx.jpg"
  }
}
```

---

## 📦 4. Subscriptions (Plans) — الباقات

> **Public:** `GET /subscriptions` و `GET /subscriptions/{id}` — بدون توكن  
> **Protected:** باقي العمليات تحتاج Super Admin

---

### `GET /subscriptions` — كل الباقات المتاحة

**مثال:**
```js
const { data } = await api.get('/subscriptions');
```

**Response `200`:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "الباقة الأساسية",
      "price_monthly": 99,
      "price_yearly": 999,
      "trial_days": 14,
      "max_users": 3,
      "max_clients": 50,
      "max_cases": 30,
      "max_sessions": 50,
      "max_tasks": 100,
      "has_templates": false,
      "has_financial_management": false,
      "has_attendance": false,
      "has_lawyer_reports": false,
      "has_notifications": true
    }
  ]
}
```

---

### `GET /subscriptions/{id}` — تفاصيل باقة محددة

**مثال:**
```js
const { data } = await api.get('/subscriptions/1');
```

**Response `200`:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "الباقة الأساسية",
    "price_monthly": 99,
    "price_yearly": 999,
    "trial_days": 14,
    "max_users": 3,
    "max_clients": 50,
    "max_cases": 30,
    "max_sessions": 50,
    "max_tasks": 100,
    "has_templates": false,
    "has_financial_management": false,
    "has_attendance": false,
    "has_lawyer_reports": false,
    "has_notifications": true
  }
}
```

---

### `POST /subscriptions` — إنشاء باقة جديدة 🔒 Super Admin

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | ✅ | فريد |
| `price_monthly` | number | ✅ | |
| `price_yearly` | number | ✅ | |
| `trial_days` | integer | ✅ | عدد أيام التجربة، 0 = بدون تجربة |
| `max_users` | integer | ✅ | |
| `max_clients` | integer | ✅ | |
| `max_cases` | integer | ✅ | |
| `max_sessions` | integer | ✅ | |
| `max_tasks` | integer | ✅ | |
| `has_templates` | boolean | ❌ | default: false |
| `has_financial_management` | boolean | ❌ | |
| `has_attendance` | boolean | ❌ | |
| `has_lawyer_reports` | boolean | ❌ | |
| `has_notifications` | boolean | ❌ | |

**مثال:**
```js
await api.post('/subscriptions', {
  name: 'الباقة الذهبية',
  price_monthly: 299,
  price_yearly: 2999,
  trial_days: 7,
  max_users: 10,
  max_clients: 200,
  max_cases: 500,
  max_sessions: 500,
  max_tasks: 1000,
  has_templates: true,
  has_financial_management: true,
  has_attendance: true,
  has_lawyer_reports: true,
  has_notifications: true,
});
```

**Response `201`:**
```json
{
  "success": true,
  "message": "تم إنشاء الباقة بنجاح",
  "data": { "id": 2, "name": "الباقة الذهبية", "..." : "..." }
}
```

---

### `PUT /subscriptions/{id}` — تعديل باقة 🔒 Super Admin

**Request Body:** نفس الـ POST، كل الحقول اختيارية (`sometimes`)

**مثال:**
```js
await api.put('/subscriptions/1', { price_monthly: 149 });
```

**Response `200`:**
```json
{
  "success": true,
  "message": "تم تحديث الباقة بنجاح",
  "data": { "id": 1, "price_monthly": 149, "...": "..." }
}
```

---

### `DELETE /subscriptions/{id}` — حذف باقة 🔒 Super Admin

**مثال:**
```js
await api.delete('/subscriptions/1');
```

**Response `200`:**
```json
{
  "success": true,
  "message": "تم حذف الباقة بنجاح"
}
```

---

## 🏢 5. Tenant Subscriptions — اشتراكات المكاتب

---

### `GET /subscriptions/status` — كل اشتراكات المكاتب 🔒 Super Admin

**Query Params (اختياري):**
| Param | Values |
|-------|--------|
| `status` | `pending` / `active` / `canceled` |

**مثال:**
```js
// كل الاشتراكات
const { data } = await api.get('/subscriptions/status');

// الاشتراكات المعلقة فقط
const { data } = await api.get('/subscriptions/status?status=pending');
```

**Response `200`:**
```json
{
  "success": true,
  "status": "pending",
  "count": 2,
  "data": [
    {
      "id": 3,
      "tenant_id": "nabil-law",
      "status": "pending",
      "type": "monthly",
      "amount_paid": 99,
      "starts_at": null,
      "expires_at": null,
      "notes": "بانتظار التفعيل اليدوي بعد الدفع",
      "plan": { "id": 1, "name": "الباقة الأساسية" },
      "tenant": { "id": "nabil-law" }
    }
  ]
}
```

---

### `POST /subscriptions/{id}/activate` — تفعيل اشتراك مكتب 🔒 Super Admin

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `payment_method` | string | ✅ | مثال: `cash`, `vodafone_cash`, `bank` |
| `notes` | string | ❌ | ملاحظات الأدمن |

**مثال:**
```js
await api.post('/subscriptions/3/activate', {
  payment_method: 'vodafone_cash',
  notes: 'تم الدفع بتاريخ 2025-01-01',
});
```

**Response `200`:**
```json
{
  "success": true,
  "message": "تم تفعيل الاشتراك بنجاح.",
  "expires_at": "2025-02-01"
}
```

**Response `400`:** الاشتراك مفعل بالفعل

---

### `POST /subscriptions/{id}/cancel` — إلغاء اشتراك مكتب 🔒 Super Admin

**Request Body:**
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `notes` | string | ✅ | سبب الإلغاء — minimum 3 chars |

**مثال:**
```js
await api.post('/subscriptions/3/cancel', {
  notes: 'المكتب لم يجدد الاشتراك',
});
```

**Response `200`:**
```json
{
  "success": true,
  "message": "تم إلغاء الاشتراك بنجاح.",
  "reason": "المكتب لم يجدد الاشتراك",
  "expires_at": "2025-01-15"
}
```

**Response `400`:** الاشتراك ملغى بالفعل

---

### `GET /subscriptions/{id}/details` — تفاصيل اشتراك محدد 🔒 Super Admin

**مثال:**
```js
const { data } = await api.get('/subscriptions/3/details');
```

**Response `200`:**
```json
{
  "success": true,
  "is_active": true,
  "message": "هذا الاشتراك فعال وصلاحيته سارية.",
  "data": {
    "id": 3,
    "tenant_id": "nabil-law",
    "status": "active",
    "type": "monthly",
    "amount_paid": 99,
    "starts_at": "2025-01-01T00:00:00Z",
    "expires_at": "2025-02-01T00:00:00Z",
    "payment_transaction_id": "MANUAL-XXXX",
    "notes": "Payment Method: cash. Admin Notes: ...",
    "plan": {
      "id": 1,
      "name": "الباقة الأساسية",
      "max_users": 3,
      "max_cases": 30
    }
  }
}
```

---

## ❗ Error Responses — الأخطاء الشائعة

| Status | المعنى | مثال |
|--------|--------|------|
| `401` | Unauthenticated — التوكن غلط أو منتهي | `{ "message": "Unauthenticated." }` |
| `403` | Forbidden — الحساب pending أو ليس لديه الصلاحية | `{ "status": false, "message": "حسابك في انتظار الموافقة" }` |
| `422` | Validation Error | انظر المثال أدناه |
| `400` | Business Logic Error | `{ "success": false, "message": "..." }` |
| `404` | Resource Not Found | `{ "message": "No query results for model..." }` |

**مثال Validation Error `422`:**
```json
{
  "message": "The name field is required.",
  "errors": {
    "name": ["The name field is required."],
    "email": ["The email has already been taken."]
  }
}
```

**Axios Error Handling:**
```js
try {
  const { data } = await api.post('/login', credentials);
} catch (error) {
  if (error.response?.status === 422) {
    const errors = error.response.data.errors;
    console.log(errors); // { email: [...], password: [...] }
  } else if (error.response?.status === 401) {
    // redirect to login
  }
}
```

---

## 🔑 Roles Summary

| Role | الصلاحيات |
|------|-----------|
| `super_admin` | إدارة كاملة للمنصة — قبول/رفض المكاتب، إدارة الباقات، تفعيل الاشتراكات |
| `owner` | أدمن المكتب — يُعطى بعد موافقة السوبر أدمن |
| `user` | مستخدم عادي في انتظار الموافقة |

---

> **ملاحظة:** الـ endpoints الخاصة بالمكاتب (Tenant Routes) موجودة على subdomain المكتب وليس الـ Central Domain.  
> مثال: `https://nabil-law.your-domain.com/api/...`
