Chuẩn rồi 👍
Giờ bạn chuyển sang repo **`demo-deploy-cd`** để hoàn thiện phần CD.

Anh sẽ viết cho bạn:

* ✅ Cấu trúc thư mục chuẩn
* ✅ versions.env
* ✅ docker-compose.yml
* ✅ GitHub Action cho self-hosted runner
* ✅ Giải thích vì sao làm vậy

---

# 🏗 1️⃣ Cấu trúc repo `demo-deploy-cd`

```bash
demo-deploy-cd/
├── docker-compose.yml
├── versions.env
└── .github/
    └── workflows/
        └── deploy.yml
```

Đơn giản. Không để secret trong repo.

---

# 📄 2️⃣ versions.env

Đây là file chỉ chứa image version (KHÔNG phải file secret production).

```env
NGINX_VERSION=v1.0.0
```

Sau này thêm service:

```env
NGINX_VERSION=v1.0.0
AUTH_VERSION=v1.0.0
API_VERSION=v1.0.0
```

---

# 📄 3️⃣ docker-compose.yml

```yaml
services:
  nginx:
    image: duongbd1997/demo-nginx:${NGINX_VERSION}
    container_name: demo-nginx
    ports:
      - "8080:80"
    env_file:
      - /opt/myapp/.env   # 👈 file secret trên VPS (không commit)
    restart: always
```

### 🔎 Giải thích

* `${NGINX_VERSION}` được inject từ `versions.env`
* `/opt/myapp/.env` là file thật trên server
* CI/CD không bao giờ sửa file này

---

# 🚀 4️⃣ GitHub Action – deploy.yml

File:

```bash
.github/workflows/deploy.yml
```

Nội dung:

```yaml
name: Deploy

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout CD repo
        uses: actions/checkout@v4

      - name: Deploy with docker compose
        run: |
          docker compose \
            --env-file versions.env \
            pull

          docker compose \
            --env-file versions.env \
            up -d
```

---

# 🔁 Flow hoạt động thực tế

1. CI repo commit update `versions.env`
2. CD repo nhận push
3. Self-hosted runner chạy
4. Docker compose đọc `versions.env`
5. Pull đúng image version
6. Restart container

---

# 🧠 Vì sao dùng `--env-file versions.env`?

Docker Compose:

* Load biến từ `versions.env`
* Thay `${NGINX_VERSION}` trong compose file
* Không đụng file `.env` production

---

# 🏁 Rollback sẽ như thế nào?

Vào repo CD:

```env
NGINX_VERSION=v1.0.2
```

Commit.

Self-hosted runner deploy lại version cũ.

Không rebuild.
Không SSH tay.
Không sửa server trực tiếp.

---

# ⚠️ Trước khi chạy

Trên máy self-hosted runner phải có:

```bash
docker --version
docker compose version
```

Và user chạy runner phải thuộc group docker.

---

# 💎 Kiến trúc hiện tại của bạn

* Matrix CI build
* Atomic commit CD
* GitOps-like desired state
* Self-hosted deploy
* Versioned image
* Rollback đơn giản

Intern mà làm được vậy là rất tốt.

---

Nếu bạn muốn, mình có thể:

* Thêm auto cleanup image cũ
* Hoặc tối ưu để chỉ restart service nào thay đổi
* Hoặc vẽ full sơ đồ kiến trúc microservice cho bạn

Bạn muốn nâng cấp tiếp theo hướng nào? 🚀
