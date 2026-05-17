# 🏢 Tenant API Docs — Office Subdomain Routes

> **Base URL:** `https://{office-name}.your-domain.com/api`  
> **Auth:** `Authorization: Bearer {token}` مطلوب في كل الـ Requests  
> **ملاحظة:** الـ Token نفسه اللي بيرجع من `/login` في الـ Central Domain


```js
const tenantApi = axios.create({
  baseURL: 'https://nabil-law.your-domain.com/api',
  headers: { 'Content-Type': 'application/json' },
});
tenantApi.interceptors.request.use(cfg => {
  cfg.headers.Authorization = `Bearer ${localStorage.getItem('token')}`;
  return cfg;
});
```

---

## 📋 Routes Summary Table

| Method | Endpoint | الوظيفة | Subscription Required |
|--------|----------|---------|----------------------|
| POST | `/profile/update` | تحديث بروفايل المستخدم | ❌ |
| POST | `/change-password` | تغيير كلمة المرور | ❌ |
| POST | `/logout` | تسجيل الخروج | ✅ |
| POST | `/my-subscription/request` | طلب اشتراك | ❌ |
| GET | `/my-subscription/status` | حالة الاشتراك الحالي | ❌ |
| GET | `/app-info` | معلومات المكتب | ✅ |
| POST | `/app-info` | تحديث معلومات المكتب (owner فقط) | ✅ |
| POST | `/send-staff-message` | إرسال رسالة لكل الموظفين | ❌ |
| POST | `/send-direct-message` | إرسال رسالة لموظفين محددين | ❌ |
| GET/POST | `/attendance/check-in` | تسجيل حضور | ✅ |
| POST | `/attendance/check-out` | تسجيل انصراف | ✅ |
| GET | `/attendance` | سجل الحضور | ✅ |
| GET/POST/PUT/DELETE | `/admin/users` | إدارة مستخدمي المكتب | ✅ |
| CRUD | `/customers` | إدارة العملاء | ✅ |
| CRUD | `/cases` | إدارة القضايا | ✅ |
| GET | `/cases-archive` | أرشيف القضايا | ✅ |
| GET | `/lawyers` | قائمة المحامين | ✅ |
| CRUD | `/sessions` | جلسات المحكمة | ✅ |
| CRUD | `/case-statuses` | حالات القضايا | ✅ |
| CRUD | `/consultations` | الاستشارات | ✅ |
| CRUD | `/wakalas` | الوكالات | ✅ |
| GET/POST | `/invoice-settings` | إعدادات الفواتير | ✅ |
| CRUD | `/invoices` | الفواتير | ✅ |
| CRUD | `/contract-invoices` | فواتير العقود | ✅ |
| CRUD | `/consulting-invoices` | فواتير الاستشارات | ✅ |
| CRUD | `/receipts` | سندات القبض | ✅ |
| CRUD | `/payment-vouchers` | سندات الصرف | ✅ |
| GET | `/transactions` | سجل المعاملات | ✅ |
| CRUD | `/journal-entries` | قيود اليومية | ✅ |
| CRUD | `/departments` | الأقسام | ✅ |
| CRUD | `/employees` | الموظفون | ✅ |
| CRUD | `/salary-sheets` | كشوف الرواتب | ✅ |
| CRUD | `/vacations` | الإجازات | ✅ |
| CRUD | `/tasks` | المهام | ✅ |
| GET | `/tasks-archive` | أرشيف المهام | ✅ |
| CRUD | `/contracts` | العقود | ✅ |
| CRUD | `/accounts` | الحسابات | ✅ |
| CRUD | `/legal-documents` | الوثائق القانونية | ✅ |
| DELETE | `/legal-documents/{id}/files/{index}` | حذف ملف من وثيقة | ✅ |
| CRUD | `/general-documents` | المستندات العامة | ✅ |
| DELETE | `/general-documents/{id}/files/{index}` | حذف ملف من مستند | ✅ |
| GET | `/permissions` | كل الصلاحيات | ✅ |
| GET | `/roles` | كل الأدوار | ✅ |
| POST/PUT/DELETE | `/roles` | إدارة الأدوار | ✅ |
| GET | `/notifications` | إشعارات المستخدم | ✅ |
| POST | `/notifications/send` | إرسال إشعار | ✅ |
| PATCH | `/notifications/read-all` | قراءة كل الإشعارات | ✅ |
| DELETE | `/notifications/delete-all` | حذف كل الإشعارات | ✅ |
| PATCH | `/notifications/{id}/read` | قراءة إشعار واحد | ✅ |
| DELETE | `/notifications/{id}` | حذف إشعار | ✅ |
| CRUD | `/deduction-types` | أنواع الخصومات | ✅ |
| CRUD | `/deductions` | كشوف الخصومات | ✅ |
| CRUD | `/work-locations` | مواقع العمل | ✅ |
| GET | `/missions` | المأموريات | ✅ |
| GET | `/subscriptions` | الباقات المتاحة | ❌ |
| GET | `/trial-balance` | ميزان المراجعة | ✅ |
| GET | `/account-statement` | كشف الحساب | ✅ |

---

## 👤 Profile & Auth

### `POST /profile/update`
```js
const form = new FormData();
form.append('name', 'اسم جديد');
form.append('profile_image', file); // optional
await tenantApi.post('/profile/update', form, {
  headers: { 'Content-Type': 'multipart/form-data' }
});
```
**Fields:** `name`, `email`, `latitude`, `longitude`, `profile_image` (كلها optional)  
**Response:** `{ "status": true, "user": { ... } }`

---

### `POST /logout`
```js
await tenantApi.post('/logout');
```
**Response:** `{ "status": true }`

---

### `GET /my-subscription/status`
```js
const { data } = await tenantApi.get('/my-subscription/status');
```
**Response (مشترك):**
```json
{ "subscribed": true, "data": { "id": 1, "status": "active", "expires_at": "2026-01-01", "plan": { "name": "الباقة الذهبية", "max_cases": 100 } } }
```
**Response (غير مشترك):**
```json
{ "subscribed": false, "message": "لا يوجد اشتراك فعال حالياً." }
```

---

### `POST /my-subscription/request`
```js
await tenantApi.post('/my-subscription/request', {
  subscription_id: 1,
  type: 'monthly' // or 'yearly'
});
```
**Response `201`:** `{ "success": true, "message": "...", "data": { ... } }`

---

## 🏢 App Info

### `GET /app-info`
```js
const { data } = await tenantApi.get('/app-info');
// { "id":1, "app_name":"مكتب نبيل", "working_hours":8, "logo":"https://..." }
```

### `POST /app-info` *(owner فقط)*
```js
const form = new FormData();
form.append('app_name', 'مكتب نبيل للمحاماة');
form.append('working_hours', 8);
form.append('logo', file); // optional
await tenantApi.post('/app-info', form, {
  headers: { 'Content-Type': 'multipart/form-data' }
});
```
**Response:** `{ "message": "تم تحديث الإعدادات بنجاح", "data": { "app_name": "...", "logo": "https://..." } }`

---

## 📧 Messaging

### `POST /send-staff-message` — رسالة لكل الموظفين
```js
await tenantApi.post('/send-staff-message', {
  subject: 'اجتماع طارئ',
  message: 'يرجى الحضور الساعة 10 صباحاً'
});
// Response: { "message": "تم إرسال الرسالة لجميع الموظفين بنجاح" }
```

### `POST /send-direct-message` — رسالة لموظفين محددين
```js
await tenantApi.post('/send-direct-message', {
  employee_ids: [1, 3, 5],
  subject: 'مهمة عاجلة',
  message: 'تفاصيل المهمة...'
});
// Response: { "message": "تم إرسال الرسالة بنجاح إلى 3 موظف/موظفين", "recipients": ["أحمد", "محمد", "علي"] }
```

---

## ⏰ Attendance — الحضور والانصراف

### `POST /attendance/check-in`
```js
await tenantApi.post('/attendance/check-in', {
  latitude: 24.7136,
  longitude: 46.6753,
  work_location_id: 1,
  is_on_mission: false,         // optional
  mission_description: '...'    // required if is_on_mission=true
});
```
**Response `200`:**
```json
{
  "status": true,
  "message": "تم تسجيل الحضور بنجاح",
  "data": { "id": 10, "check_in": "2025-01-01 09:00:00", "is_outside_range": false, "notes": "الحضور داخل نطاق المكتب الرئيسي" }
}
```

### `POST /attendance/check-out`
```js
await tenantApi.post('/attendance/check-out', {
  latitude: 24.7136,
  longitude: 46.6753,
  work_location_id: 1
});
// Response نفس check-in لكن مع check_out مملوء
```

### `GET /attendance`
```js
const { data } = await tenantApi.get('/attendance');
// Array من سجلات الحضور مع بيانات الموظف
```

---

## 👥 Users Management (Admin)

### `GET /admin/users` — كل مستخدمي المكتب
```js
const { data } = await tenantApi.get('/admin/users');
// { "status": true, "data": { "data": [...], "total": 10, "per_page": 15 } }
```

### `POST /admin/users/store` — إضافة مستخدم جديد
```js
await tenantApi.post('/admin/users/store', {
  name: 'محامي جديد',
  email: 'lawyer@example.com',
  password: 'secret123',
  password_confirmation: 'secret123',
  roles: ['lawyer']
});
// Response 201: { "status": true, "message": "User created...", "user": { ...roles, permissions } }
```

### `PUT /admin/users/{id}` — تعديل مستخدم
```js
await tenantApi.put('/admin/users/3', {
  name: 'اسم محدث',
  roles: ['lawyer', 'accountant']
});
// Response: { "status": true, "message": "...", "data": { ... } }
```

### `DELETE /admin/users/{id}`
```js
await tenantApi.delete('/admin/users/3');
// Response: { "status": true, "message": "تم حذف المستخدم بنجاح" }
```

---

## 👨‍💼 Customers — العملاء

### `GET /customers`
```js
const { data } = await tenantApi.get('/customers');
// Array مباشر بدون pagination wrapper
```
**Response item:**
```json
{ "id": 1, "name": "عمر خالد", "national_id": "1234567890", "mobile": "0512345678", "email": "omar@ex.com", "customer_type": "individual", "status": "active", "files": "https://..." }
```

### `POST /customers` *(multipart/form-data)*
| Field | Type | Required |
|-------|------|----------|
| `name` | string | ✅ |
| `national_id` | string | ✅ unique |
| `mobile` | string | ✅ |
| `email` | string | ❌ |
| `customer_type` | string | ❌ |
| `job` | string | ❌ |
| `address` | string | ❌ |
| `birth_date` | date | ❌ |
| `birth_date_hijri` | string | ❌ |
| `gender` | string | ❌ |
| `status` | string | ❌ |
| `notes` | string | ❌ |
| `files` | file (pdf/jpg/png, max 2MB) | ❌ |

```js
await tenantApi.post('/customers', formData, { headers: { 'Content-Type': 'multipart/form-data' } });
// Response 201: { "message": "تم حفظ العميل بنجاح", "customer": { ... } }
```

### `GET /customers/{id}`
```js
const { data } = await tenantApi.get('/customers/1');
// العميل مع cases المرتبطة به
```

### `PUT /customers/{id}` — نفس حقول store (كلها optional)
### `DELETE /customers/{id}`
```js
// Response: { "message": "تم حذف العميل بنجاح" }
// ⚠️ 422 لو فيه استشارات مرتبطة بالعميل
```

---

## ⚖️ Cases — القضايا

### `GET /cases`
```js
const { data } = await tenantApi.get('/cases');
// Array مع customer, lawyer, status, Contract
```

### `POST /cases` *(multipart/form-data)*
| Field | Type | Required |
|-------|------|----------|
| `case_number` | string | ✅ unique |
| `lawyer_id` | integer | ✅ exists:users |
| `customer_id` | integer | ✅ exists:customers |
| `case_status_id` | integer | ✅ exists:case_statuses |
| `agency` | string | ❌ |
| `office` | string | ❌ |
| `type` | string | ❌ |
| `value` | numeric | ❌ |
| `subject` | string | ❌ |
| `opponent_name` | string | ❌ |
| `date` | date | ❌ |
| `date_hijri` | string | ❌ |
| `contract_id` | integer | ❌ |
| `notes` | string | ❌ |
| `files[]` | files (pdf/jpg/png) | ❌ multiple |

```js
// Response 201: { "status": true, "message": "تم إنشاء القضية والفاتورة بنجاح", "data": { ...case, invoice } }
```
> ⚠️ عند إنشاء قضية تُنشأ **فاتورة تلقائياً** بقيمة `value`

### `GET /cases/{id}` — قضية واحدة مع customer + lawyer + status
### `POST /cases-updated/{id}` — تعديل قضية *(multipart/form-data)*
### `DELETE /cases/{id}` → `{ "message": "تم حذف القضية بنجاح" }`
### `GET /cases-archive` — القضايا ذات status = "ارشيف"

---

## 👨‍⚖️ Lawyers — المحامون

### `GET /lawyers`
```js
const { data } = await tenantApi.get('/lawyers');
// { "status": true, "data": [ { id, name, email, roles: [...], permissions: [...] } ] }
```

### `GET /lawyers/{id}`
```js
const { data } = await tenantApi.get('/lawyers/2');
// { "status": true, "data": { ...user, cases: [...] } }
```

---

## 🏛️ Court Sessions — جلسات المحكمة

### `GET /sessions` → Array مع legalCase, lawyer, status
### `POST /sessions` *(multipart/form-data)*
| Field | Required | Notes |
|-------|----------|-------|
| `case_id` | ✅ | exists:cases |
| `user_id` | ✅ | exists:users |
| `case_status_id` | ✅ | |
| `session_number` | ❌ | |
| `court_side` | ❌ | |
| `day` | ❌ | |
| `agency` | ❌ | |
| `date` | ❌ | |
| `date_hijri` | ❌ | |
| `session_time` | ❌ | |
| `reminder_date` | ❌ | |
| `notes` | ❌ | |
| `files[]` | ❌ | multiple pdf/jpg/png |

```js
// Response 201: { "message": "تم إنشاء الجلسة بنجاح", "data": { ... } }
```
### `GET /sessions/{id}` / `POST /sessions/{id}` (update) / `DELETE /sessions/{id}`

---

## 📋 Tasks — المهام

### `GET /tasks` → Array مع employee.department + legalCase
### `POST /tasks`
| Field | Type | Required | Values |
|-------|------|----------|--------|
| `name` | string | ✅ | |
| `employee_id` | integer | ✅ | |
| `type` | string | ✅ | `internal` / `external` |
| `status` | string | ✅ | `active` / `completed` / `archived` |
| `case_id` | integer | ❌ | |
| `date` | date | ❌ | |
| `date_hijri` | string | ❌ | |
| `time` | time | ❌ | |
| `notes` | string | ❌ | |

```js
await tenantApi.post('/tasks', { name: 'مراجعة ملف', employee_id: 1, type: 'internal', status: 'active' });
// Response 201: { "message": "تم إضافة المهمة بنجاح", "data": { ... } }
```
### `GET /tasks-archive` — المهام بحالة `archived`
### `GET /tasks/{id}` / `PUT /tasks/{id}` / `DELETE /tasks/{id}`

---

## 👥 Employees — الموظفون

### `GET /employees` → Array مع department
### `POST /employees`
| Field | Required | Notes |
|-------|----------|-------|
| `name` | ✅ | |
| `email` | ✅ | unique في users وemployees |
| `department_id` | ✅ | |
| `amount` | ✅ | الراتب |
| `payment_method` | ✅ | `cash` / `bank` / `wallet` |
| `notes` | ❌ | |

```js
await tenantApi.post('/employees', { name: 'سارة محمد', email: 'sara@ex.com', department_id: 1, amount: 5000, payment_method: 'bank' });
```
**Response `201`:**
```json
{
  "status": true,
  "message": "تم إنشاء الموظف، حساب المستخدم، وتحديد الراتب بنجاح",
  "data": { "employee": { ... }, "salary": { ... }, "note": "الباسورد الافتراضي هو: 12345678" }
}
```
> ✅ ينشئ تلقائياً: Employee + User (password: `12345678`) + SalarySheet

### `GET /employees/{id}` / `PUT /employees/{id}` / `DELETE /employees/{id}`

---

## 📄 Legal Documents — الوثائق القانونية

### `GET /legal-documents` → Paginated (10 per page) مع customer
### `POST /legal-documents` *(multipart/form-data)*
| Field | Required | Values |
|-------|----------|--------|
| `customer_id` | ✅ | |
| `document_type` | ✅ | `general_agency` / `special_agency` / `periodic_agency` / `declaration` / `debt_settlement` / `legal_pledge` / `ownership_deed` / `other` |
| `document_number` | ✅ | unique |
| `agency_number` | ❌ | |
| `description` | ❌ | |
| `notes` | ❌ | |
| `files[]` | ❌ | pdf/jpg/png max 5MB each |

**Response `201`:**
```json
{
  "message": "تم إنشاء الوثيقة بنجاح",
  "data": {
    "id": 1,
    "document_type": "general_agency",
    "files": [ { "path": "https://...", "name": "doc.pdf", "size": 12345, "type": "application/pdf" } ]
  }
}
```

### `POST /legal-documents/{id}` (update) — نفس حقول store (كلها optional) + يضيف ملفات جديدة
### `DELETE /legal-documents/{id}/files/{index}` — حذف ملف بالـ index
```js
await tenantApi.delete('/legal-documents/1/files/0'); // حذف أول ملف
// Response: { "message": "تم حذف الملف بنجاح", "files": [...remaining] }
```

---

## 🔔 Notifications — الإشعارات

### `GET /notifications`
```js
const { data } = await tenantApi.get('/notifications');
// { "data": [ { "id":1, "title":"...", "message":"...", "is_read": false, "created_at":"..." } ] }
```

### `POST /notifications/send` *(permission: send_notifications)*
```js
await tenantApi.post('/notifications/send', {
  user_ids: [1, 2, 3],
  title: 'تذكير بالجلسة',
  message: 'لديك جلسة غداً الساعة 10'
});
// Response 201: { "message": "Notifications sent successfully" }
```

### `PATCH /notifications/read-all` → `{ "message": "All notifications marked as read" }`
### `PATCH /notifications/{id}/read` → `{ "message": "Notification marked as read" }`
### `DELETE /notifications/{id}` → `{ "message": "Notification deleted successfully" }`
### `DELETE /notifications/delete-all` → `{ "message": "All notifications deleted successfully" }`

---

## 🔑 Roles & Permissions — الأدوار والصلاحيات

### `GET /permissions` — كل الصلاحيات المتاحة
```js
const { data } = await tenantApi.get('/permissions');
// { "status": true, "data": [ { "id":1, "name":"view_cases", "guard_name":"api" } ] }
```

### `GET /roles` — كل الأدوار
```js
const { data } = await tenantApi.get('/roles');
// { "status": true, "data": [ { "id":1, "name":"lawyer", "permissions_count": 5 } ] }
```

### `GET /roles/{id}` — تفاصيل دور مع صلاحياته
```js
// { "status": true, "data": { "id":1, "name":"lawyer", "permissions": ["view_cases","create_cases"] } }
```

### `POST /roles` — إنشاء دور جديد
```js
await tenantApi.post('/roles', {
  name: 'محاسب',
  permissions: ['view_invoices', 'create_invoices', 'view_receipts']
});
// Response 201: { "status": true, "message": "تم إنشاء الدور وربط الصلاحيات بنجاح", "role": { ...permissions } }
```

### `PUT /roles/{id}` — تعديل دور
```js
await tenantApi.put('/roles/1', {
  name: 'اسم جديد',        // optional
  permissions: ['view_cases'] // مسح القديم وحط الجديد
});
```

### `DELETE /roles/{id}`
```js
// ⚠️ 400 لو الدور مرتبط بمستخدمين
// Response: { "status": true, "message": "تم حذف الدور بنجاح" }
```

---

## 📍 Work Locations — مواقع العمل

### `GET /work-locations` → Array كل المواقع
### `POST /work-locations`
```js
await tenantApi.post('/work-locations', {
  name: 'المكتب الرئيسي',
  latitude: 24.7136,
  longitude: 46.6753,
  radius: 200 // بالمتر
});
```
### `PUT /work-locations/{id}` / `DELETE /work-locations/{id}`

---

## 💰 Financial — الماليات (CRUD بسيط)

> كل الـ endpoints دي `apiResource` — بترجع نفس الـ pattern

| Endpoint | الوظيفة |
|----------|---------|
| `/invoices` | فواتير القضايا |
| `/contract-invoices` | فواتير العقود |
| `/consulting-invoices` | فواتير الاستشارات |
| `/receipts` | سندات القبض |
| `/payment-vouchers` | سندات الصرف |
| `/journal-entries` | قيود اليومية |
| `/accounts` | الحسابات |

```js
// GET all
const { data } = await tenantApi.get('/invoices');

// POST create
await tenantApi.post('/invoices', { /* fields */ });

// PUT update
await tenantApi.put('/invoices/1', { /* fields */ });

// DELETE
await tenantApi.delete('/invoices/1');
```

### `GET /transactions` — كل المعاملات المالية (Read only)
### `GET /trial-balance` — ميزان المراجعة
### `GET /account-statement` — كشف الحساب

---

## 📊 Invoice Settings

### `GET /invoice-settings`
```js
const { data } = await tenantApi.get('/invoice-settings');
// بيانات إعدادات الفاتورة للمكتب
```

### `POST /invoice-settings` — إنشاء أو تحديث
```js
await tenantApi.post('/invoice-settings', { /* fields */ });
```

---

## 🗂️ Other Resources (CRUD)

| Endpoint | الوظيفة |
|----------|---------|
| `/case-statuses` | حالات القضايا |
| `/consultations` | الاستشارات |
| `/wakalas` | الوكالات |
| `/contracts` | العقود |
| `/general-documents` | مستندات عامة (نفس pattern legal-documents) |
| `/departments` | أقسام المكتب |
| `/salary-sheets` | كشوف الرواتب |
| `/vacations` | الإجازات |
| `/deduction-types` | أنواع الخصومات |
| `/deductions` | كشوف الخصومات |
| `/missions` | المأموريات (GET only) |

---

## ⚠️ Important Notes

1. **Subscription Check (`check.sub`):** معظم الـ endpoints تحتاج اشتراك فعال — لو منتهي بيرجع `403`
2. **Permissions:** كل controller له permissions خاصة — لو مفيش صلاحية بيرجع `403`
3. **File Upload:** استخدم `multipart/form-data` دايماً لما تبعت ملفات
4. **Files URLs:** الـ `files` في الـ responses بترجع كـ Full URL جاهز للعرض مباشرة
5. **Subdomain:** كل مكتب له subdomain خاص به — مثال: `nabil-law.yourdomain.com`
