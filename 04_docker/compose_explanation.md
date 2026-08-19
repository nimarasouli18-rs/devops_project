cat > ~/Desktop/devops_project/04_docker/compose_explanation.md << 'EOF'
# توضیح docker-compose.yml

این فایل بر پایه compose.yaml اصلی ریپو ساخته شده، با یک تغییر مهم:
**حذف سرویس "proxy"**.

## دلیل حذف سرویس proxy
سرویس proxy یک Nginx کانتینری بود که پورت 80 را اشغال می‌کرد.
طبق معماری این پروژه، Nginx باید مستقیماً روی سرور (host) نصب و
پیکربندی شود (توسط عضو دیگر تیم، مسئول فاز Nginx/SSL)، نه داخل
Docker. اجرای هم‌زمان هر دو باعث تداخل (port conflict) روی پورت 80
می‌شد، بنابراین سرویس proxy حذف و پورت backend (8000) مستقیماً
expose شد تا Nginx بیرونی بتواند به آن reverse proxy کند.

## سرویس‌ها

### db (MariaDB)
- image: mariadb:10-focal
- healthcheck: بررسی سلامت با mysqladmin ping قبل از آماده اعلام شدن
- پسورد از طریق Docker Secret (db/password.txt) نه environment variable ساده
- volume نام‌گذاری‌شده (db-data) برای persistence داده‌ها بین ری‌استارت‌ها
- فقط در شبکه داخلی backnet قابل دسترسی (expose نه publish، یعنی از
  بیرون Docker قابل دسترس نیست)

### backend (Flask)
- build از پوشه backend/ با target: builder (مرحله production، بدون dev tools)
- پورت 8000 به host منتشر می‌شود (این پورتی است که Nginx بیرونی به آن proxy می‌کند)
- در دو شبکه backnet (ارتباط با db) و frontnet (ارتباط با بیرون/Nginx) عضو است
- depends_on با condition: service_healthy تضمین می‌کند backend فقط
  زمانی بالا بیاید که db کاملاً آماده باشد

## Volumes
- db-data: ذخیره‌سازی دائمی داده‌های MariaDB

## Secrets
- db-password: از فایل db/password.txt خوانده می‌شود و در مسیر
  /run/secrets/db-password داخل کانتینرها در دسترس قرار می‌گیرد

## Networks
- backnet: شبکه داخلی بین backend و db
- frontnet: شبکه‌ای که backend را برای دسترسی بیرونی (از طریق Nginx host) در دسترس می‌گذارد
EOF
