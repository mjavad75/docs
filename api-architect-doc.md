# راهنمای معماری و ساختار پروژه

## معرفی
این پروژه از معماری لایه‌ای با رعایت اصول SOLID و Django Style Guide استفاده می‌کند. هدف اصلی جداسازی منطق کسب و کار (Business Logic) از لایه‌های دیگر و ایجاد کد قابل نگهداری و توسعه است.

## ساختار پروژه

```
project/
├── utils/
│   ├── repository/
│   │   └── base_repository.py
│   └── services/
│       └── base_services.py
└── apps/
    └── your_app/
        └── services.py
```

## لایه‌های معماری

### 1. لایه Repository
فایل `base_repository.py` پیاده‌سازی الگوی Repository را ارائه می‌دهد که مسئول تعامل با پایگاه داده است.

#### ویژگی‌های اصلی:
- پیاده‌سازی اینترفیس `IBaseRepository` با متدهای CRUD پایه
- کلاس `BaseRepository` که پیاده‌سازی پایه را فراهم می‌کند
- عملیات‌های اصلی:
  - `add`: افزودن رکورد جدید
  - `get`: دریافت یک رکورد
  - `filter`: فیلتر کردن رکوردها
  - `update`: به‌روزرسانی رکورد
  - `delete`: حذف رکورد

```python
class BaseRepository(IBaseRepository):
    def __init__(self, model: models.Model):
        self.model = model

    def add(self, **kwargs):
        instance = self.model.objects.create(**kwargs)
        return instance
    # ...
```

### 2. لایه Service
فایل `base_services.py` لایه سرویس پایه را پیاده‌سازی می‌کند که منطق کسب و کار را در خود جای می‌دهد.

#### ویژگی‌های اصلی:
- کلاس `BaseService` که عملیات‌های پایه را فراهم می‌کند
- مدیریت خطاها و اعتبارسنجی
- تعامل با Repository

```python
class BaseService:
    def __init__(self, repository: IBaseRepository):
        self.repository = repository

    def create(self, **kwargs):
        try:
            return self.repository.add(**kwargs)
        except Exception as e:
            raise ValidationError({'error': f'{e}'})
    # ...
```

### 3. سرویس‌های اختصاصی (App Services)
هر اپلیکیشن جنگو دارای فایل `services.py` است که منطق کسب و کار خاص آن اپلیکیشن را پیاده‌سازی می‌کند.

#### نمونه پیاده‌سازی:
```python
class OccasionService:
    @staticmethod
    @transaction.atomic
    def create(**kwargs):
        try:
            images = kwargs.pop('images', [])
            occasion = occasion_base_service.create(**kwargs)
            # پیاده‌سازی منطق خاص
            return occasion
        except ValidationError as e:
            raise e
```

## نحوه استفاده

### 1. تعریف Repository جدید
```python
from utils.repository.base_repository import BaseRepository

class YourModelRepository(BaseRepository):
    def __init__(self):
        super().__init__(YourModel)
```

### 2. تعریف Service جدید
```python
from utils.services.base_services import BaseService

class YourModelService:
    def __init__(self):
        self.base_service = BaseService(YourModelRepository())
```

### 3. استفاده در View
```python
class YourModelViewSet(ViewSet):
    def create(self, request):
        service = YourModelService()
        return service.create(**request.data)
```

## مزایای این معماری

1. **جداسازی مسئولیت‌ها**: هر لایه وظیفه مشخصی دارد
2. **قابلیت تست**: امکان تست هر لایه به صورت مجزا
3. **قابلیت توسعه**: اضافه کردن قابلیت‌های جدید بدون تغییر در کد موجود
4. **کد تمیز**: پیروی از اصول SOLID و الگوهای طراحی
5. **مدیریت خطا**: مدیریت متمرکز خطاها در لایه سرویس

## نکات مهم

1. همیشه از `@transaction.atomic` برای عملیات‌های پیچیده استفاده کنید
2. خطاها را در لایه سرویس مدیریت کنید
3. منطق کسب و کار را در View‌ها قرار ندهید
4. از Type Hints برای افزایش خوانایی کد استفاده کنید

## مستندات مرتبط
- [Django Style Guide](https://github.com/HackSoftware/Django-Styleguide)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
