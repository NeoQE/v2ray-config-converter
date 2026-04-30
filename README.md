# V2Ray Server Tool

ابزار آنلاین برای راه‌اندازی V2Ray و نصب هر چیزی روی سرور لینوکس (Ubuntu) از طریق VPN

ساخته شده توسط **NeoQ**

🔗 [Live Tool](https://justneoq.github.io/v2ray-server-tool/)
📱 [کانال تلگرام](https://t.me/Just_Neo_Q)

---

## چی کار می‌کنه؟

سرورهای ایران به خیلی از سرویس‌ها (Docker Hub، GitHub releases، PyPI و...) دسترسی ندارن. این ابزار یه راه‌حل کامل می‌ده:

1. **کانفیگ V2Ray رو می‌گیره** و دستور نصب کامل می‌سازه
2. **هر چیزی که بخوای نصب کنی** — لینک GitHub، پکیج apt، پکیج pip، فایل دانلودی — خودکار از طریق VPN رد می‌کنه
3. **مدیریت VPN** روی سرور (روشن/خاموش/تست/تعویض کانفیگ)

---

## چطور استفاده کنم؟

### مرحله ۱ — کانفیگ V2Ray رو روی سرور نصب کن

تب اول سایت → کانفیگت (`vless://...` یا JSON یا هر فرمت دیگه) رو paste کن → دستور کامل بگیر → روی سرور بزن.

دستور این کارها رو می‌کنه:
- DNS رو تنظیم می‌کنه
- mirror ایران‌سرور رو set می‌کنه
- v2ray + proxychains4 نصب می‌کنه
- `config.json` می‌نویسه
- v2ray رو روشن می‌کنه

### مرحله ۲ — ابزار `vpn-run` رو نصب کن (یک بار)

تب دوم سایت → بنر زرد بالا «نصب ابزار مستر vpn-run» → کپی کن → روی سرور بزن.

این کار:
- `privoxy` به‌عنوان HTTP proxy روی پورت 8118 نصب می‌کنه
- `apt` رو برای دامین‌های خارجی (docker.com، nodesource، cloudflare، ...) configure می‌کنه
- **Docker** رو از طریق VPN کامل نصب و راه‌اندازی می‌کنه
- اسکریپت `vpn-run` رو در `/usr/local/bin/` می‌سازه

### مرحله ۳ — هر چیزی می‌خوای نصب کن

از تب دوم، چیزی که می‌خوای نصب کنی رو بنویس → دستور بگیر → روی سرور paste کن. خروجی همیشه با `vpn-run` شروع می‌شه که یعنی **خودکار از VPN رد می‌شه**.

```bash
vpn-run apt install -y nginx                 # apt با proxy
vpn-run docker pull alpine                   # docker با proxy
vpn-run pip3 install requests                # pip با proxy
vpn-run bash <(curl -sL https://...)         # هر اسکریپت با proxy
```

---

## بخش‌های سایت

### ۱. 🔄 تبدیل کانفیگ V2Ray

ورودی‌های قابل قبول:
- لینک‌های `vless://`، `vmess://`، `trojan://`، `ss://`
- JSON خالص
- دستور bash کامل (`cat << EOF ... EOF`)

خروجی: یک دستور تک‌خطی که همه چیز رو روی سرور تازه نصب و راه‌اندازی می‌کنه.

---

### ۲. 📥 دستور نصب پکیج / GitHub

نوع‌های پشتیبانی‌شده:

#### 🤖 خودکار
نوع ورودی رو خودش تشخیص می‌ده. هر چیزی بدی، می‌فهمه.

#### 🐙 لینک GitHub
- **ریپوهای well-known**: 3x-ui، x-ui، Marzban، Marzneshin، Marznode، Hiddify، reality-ezpz، s-ui — اسکریپت دقیق و آرگومان درست خودش انتخاب می‌شه
- **ریپوهای ناشناخته**: ۸ مسیر متداول رو خودکار جستجو می‌کنه (`install.sh`، `script.sh`، `setup.sh`، `{repo}.sh` در branch های master و main)
- **پشتیبانی از idiomهای پیچیده**:
  - `bash -c "$(curl -sL URL)" @ install --database mysql` ✅
  - `bash <(curl URL) install` ✅
  - `curl URL | bash -s -- args` ✅
  - `wget -O- URL | bash` ✅
- آرگومان‌ها (مثل `install`, `--database mysql`) از دستور استخراج می‌شن و حفظ می‌شن

#### 📦 پکیج apt
اسم پکیج → `vpn-run apt install -y <pkg>` — از mirror ایران (محلی) و در صورت نیاز از VPN رد می‌شه.

#### 🐍 پکیج pip
اسم پکیج → اول چک می‌کنه `python3-pip` نصب باشه (اگه نه خودش نصب می‌کنه)، بعد پکیج رو با proxy نصب می‌کنه.

#### 🔗 لینک دانلود
URL مستقیم → دانلود با retry (۵ بار) و resume (`-C -`) برای فایل‌های بزرگ. اگه `.tar.gz` بود اکسترکت می‌کنه، اگه `.deb` بود نصب می‌کنه، اگه `.sh` بود اجرا می‌کنه.

---

### ۳. 🛠️ مدیریت VPN روی سرور

چهار کارت تعاملی:

| کارت | کار |
|---|---|
| ▶️ **روشن کردن VPN** | `enable` + `start` v2ray + set کردن متغیرهای proxy |
| ⏹️ **خاموش کردن VPN** | `stop` + `disable` v2ray + `unset` متغیرها |
| 🧪 **تست کانفیگ** | چک می‌کنه v2ray فعاله، پورت 1080 باز، **IP بدون VPN vs IP با VPN**، latency |
| 🔄 **تعویض کانفیگ** | از کانفیگ فعلی backup می‌گیره، JSON جدید می‌گیره، اگه شکست خورد **خودش به نسخه قبلی برمی‌گرده** |

---

## معماری فنی — `vpn-run` چطور کار می‌کنه؟

`vpn-run` یه wrapper script هست که:

1. **متغیرهای محیطی proxy** رو set می‌کنه (`http_proxy`, `https_proxy`, `ALL_PROXY`)
2. **wrapper موقت برای curl/wget** در `/tmp/vpnrun-bin/` می‌سازه و ابتدای `PATH` می‌ذاره
3. curl wrapper خودکار `--socks5-hostname 127.0.0.1:1080 --retry 5 --retry-delay 3 -C -` رو اضافه می‌کنه
4. چک می‌کنه `privoxy` روشن باشه و v2ray فعال باشه
5. دستور کاربر رو با محیط آماده اجرا می‌کنه

نتیجه: هر ابزار nested (curl داخل اسکریپت، wget داخل docker، apt داخل سرویس) خودکار از VPN رد می‌شه.

---

## مشکلات حل شده توسط این ابزار

| مشکل | راه‌حل |
|---|---|
| `apt` با socks5 کار نمی‌کنه | privoxy → HTTP proxy → forward به socks5 |
| `docker pull` به Docker Hub نمی‌رسه | systemd proxy.conf برای Docker daemon |
| دانلود فایل بزرگ قطع می‌شه | `--retry 5 -C -` (resume) در curl wrapper |
| `bash <(curl ...)` با curl-proxied هنگ می‌کنه | اول دانلود → بعد اجرا |
| اسکریپت GitHub در path غیر استاندارد | جستجوی هوشمند ۸ مسیر متداول |
| `bash -c "$(curl URL)" @ install --database mysql` | استخراج URL + حفظ آرگومان `install --database mysql` |

---

## ریپوهای تست‌شده

✅ **3x-ui** (mhsanaei/3x-ui)
✅ **Marzban** (Gozargah/Marzban + Marzban-scripts)
✅ **Marzneshin** (marzneshin/Marzneshin)
✅ **nginx, htop, git, nano** و سایر پکیج‌های apt
✅ **Docker pull** برای image از Docker Hub

---

## نکات مهم

| ابزار | نکته |
|---|---|
| `vpn-run` | فقط یک بار نصب می‌شه، permanent در `/usr/local/bin/` |
| `privoxy` | با systemd auto-start می‌شه بعد از reboot |
| Docker | proxy config در `/etc/systemd/system/docker.service.d/proxy.conf` permanent |
| `curl` | داخل `vpn-run` خودکار `--socks5-hostname` و retry می‌گیره |
| `apt` | برای دامین‌های خارجی از privoxy رد می‌شه (config در `/etc/apt/apt.conf.d/99vpn-proxy`) |
| پیش‌نیاز | v2ray باید روشن باشه (پورت 1080) قبل از استفاده از `vpn-run` |

---

## ❓ سؤالات متداول

**Q: بعد از reboot سرور، باید دوباره چیزی نصب کنم؟**
نه. همه چیز permanent ذخیره شده — privoxy، Docker، vpn-run همه auto-start می‌شن.

**Q: اگه ترمینال رو ببندم، vpn-run باز کار می‌کنه؟**
آره. `/usr/local/bin/vpn-run` در فایل‌سیستمه و ربطی به session نداره.

**Q: می‌تونم کانفیگ VPN رو عوض کنم؟**
آره. از بخش «مدیریت VPN» → کارت 🔄 تعویض کانفیگ. خودش backup می‌گیره و اگه کانفیگ جدید کار نکرد، به قبلی برمی‌گرده.

**Q: اگه دانلود فایل بزرگ قطع شد چیکار کنم؟**
دستور رو دوباره بزن. `vpn-run` خودش با `-C -` از نقطه قطع ادامه می‌ده.

---

*ساخته شده توسط [NeoQ](https://t.me/Just_Neo_Q)*
