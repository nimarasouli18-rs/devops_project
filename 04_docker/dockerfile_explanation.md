cat > ~/Desktop/devops_project/04_docker/dockerfile_explanation.md << 'EOF'
# توضیح Dockerfile

این Dockerfile از قبل در ریپوی انتخابی (backend/Dockerfile) وجود داشت
و پس از بررسی، بدون تغییر استفاده شد چون از best practice های استاندارد پیروی می‌کند.

## ساختار کلی
از multi-stage build استفاده شده با دو مرحله:
- stage "builder": مرحله اصلی که ایمیج production را می‌سازد
- stage "dev-envs": مرحله اضافی برای محیط توسعه (شامل ابزارهای git و docker cli)
  که در این پروژه استفاده نمی‌شود و فقط "builder" build می‌شود.

## توضیح خط‌به‌خط

| خط | توضیح |
|---|---|
| `FROM --platform=$BUILDPLATFORM python:3.13-alpine AS builder` | ایمیج پایه سبک (alpine) پایتون، با نام‌گذاری stage به "builder" |
| `WORKDIR /code` | تنظیم پوشه کاری داخل کانتینر |
| `COPY requirements.txt /code` | کپی جداگانه فایل dependencies قبل از کد، برای بهره‌گیری از layer caching |
| `RUN --mount=type=cache,target=/root/.cache/pip pip3 install -r requirements.txt` | نصب کتابخانه‌ها با cache mount که سرعت build مجدد را افزایش می‌دهد |
| `COPY . .` | کپی باقی سورس کد اپلیکیشن |
| `ENV FLASK_APP hello.py` | معرفی فایل اصلی اپلیکیشن به Flask |
| `ENV FLASK_RUN_HOST 0.0.0.0` | اجازه دسترسی از بیرون کانتینر (نه فقط localhost داخلی) |
| `EXPOSE 8000` | مستندسازی پورتی که اپلیکیشن روی آن اجرا می‌شود |
| `CMD ["flask", "run"]` | دستور اجرای نهایی اپلیکیشن |

## نکته درباره build
هنگام build باید از `--target builder` استفاده شود تا مرحله dev-envs
(که برای production غیرضروری است) اجرا نشود:
