# ⭐ **FULL GUIDE DEPLOY SPRING BOOT BACKEND LÊN EC2 (DOCKER + GITHUB ACTIONS CI/CD)**

Áp dụng cho project có cấu trúc:

```
backend/
   Dockerfile
   docker-compose.yml
```

---

# 🟩 **1. Chuẩn Bị EC2**

### 1.1. Cài Docker + Docker Compose v2

SSH vào EC2:

```bash
sudo apt update
sudo apt install -y docker.io
```

Cài docker-compose plugin (nếu không có, dùng bản binary):

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.29.7/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Test:

```bash
docker-compose version
```

Phải thấy **Docker Compose version v2.x.x**

---

# 🟩 **2. Chuẩn bị thư mục deploy trên EC2**

Tạo thư mục chứa docker-compose:

```bash
sudo mkdir -p /opt/spring-api
sudo chown ubuntu:ubuntu /opt/spring-api
```

Upload file `docker-compose.yml` của backend vào đây (qua WinSCP hoặc copy từ repo).

Example file:

```yaml
services:
  api:
    image: vuducthanh0115/chinese-website-api:latest
    container_name: chinese-website-api
    restart: always
    ports:
      - "8080:8080"
    env_file:
      - .env

    # Push log lên CloudWatch (nếu dùng)
    logging:
      driver: awslogs
      options:
        awslogs-region: ap-northeast-1
        awslogs-group: /chinese-website/api
        awslogs-stream: backend-api
```

---

# 🟩 **3. Tạo SSH Key cho GitHub Actions**

Chạy trên máy local:

```bash
ssh-keygen -t ed25519 -C "github-actions-ec2"
```

Chọn đường dẫn:

```
~/.ssh/id_ed25519_ec2
```

Copy **public key**:

```bash
cat ~/.ssh/id_ed25519_ec2.pub
```

Dán vào EC2:

```bash
nano ~/.ssh/authorized_keys
```

Paste → Save.

Copy **private key** vào GitHub Secrets:

```
EC2_SSH_KEY = <private key>
```

---

# 🟩 **4. Thêm Secrets vào GitHub**

Repo → Settings → Secrets → Actions:

| Key                  | Value                          |
| -------------------- | ------------------------------ |
| `EC2_HOST`           | IP EC2 (chỉ IP, không http://) |
| `EC2_USER`           | ubuntu                         |
| `EC2_SSH_KEY`        | private key                    |
| `DOCKERHUB_USERNAME` | tên Docker Hub                 |
| `DOCKERHUB_TOKEN`    | token Docker Hub               |

---

# 🟩 **5. Tạo GitHub Actions CI/CD**

Tạo file:

`.github/workflows/deploy-app.yml`

→ Đây là bản chuẩn: có **2 job** (build/push → deploy), **chọn nhánh khi chạy**.

```yaml
name: Build & Deploy Backend to EC2

on:
  workflow_dispatch:
    inputs:
      branch:
        description: "Chọn nhánh để build & deploy"
        required: true
        default: "main"

jobs:
  # -------------------- JOB 1: BUILD + PUSH IMAGE --------------------
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.inputs.branch }}

      - name: Normalize branch name
        run: |
          SAFE_BRANCH=${{ github.event.inputs.branch }}
          SAFE_BRANCH=${SAFE_BRANCH//\//-}
          echo "BRANCH_TAG=$SAFE_BRANCH" >> $GITHUB_ENV

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build & Push Docker Image
        uses: docker/build-push-action@v6
        with:
          context: ./backend
          file: ./backend/Dockerfile
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/chinese-website-api:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/chinese-website-api:${{ env.BRANCH_TAG }}

  # -------------------- JOB 2: DEPLOY TO EC2 --------------------
  deploy:
    runs-on: ubuntu-latest
    needs: build-and-push

    steps:
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H "${{ secrets.EC2_HOST }}" >> ~/.ssh/known_hosts

      - name: Deploy to EC2
        run: |
          ssh -i ~/.ssh/id_ed25519 ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }} "
            cd /opt/spring-api && \
            docker-compose pull && \
            docker-compose up -d
          "
```

---

# 🟩 **6. Chạy CI/CD: chọn nhánh để deploy**

Đi đến:

**GitHub → Actions → Build & Deploy Backend → Run workflow**

Bạn sẽ thấy:

```
Branch: [main ▼]
```

→ Chọn nhánh → Run
GitHub sẽ:

1. Build Docker image từ `backend/Dockerfile`
2. Push lên Docker Hub
3. SSH vào EC2
4. `docker-compose pull`
5. `docker-compose up -d`
6. Backend live!

---

# 🟩 (Optional) **7. Đẩy log Docker lên CloudWatch**

Nếu bạn muốn xem log không cần SSH:

### 7.1. Tạo IAM Role

`CloudWatchLogsFullAccess`

Gán cho EC2:

EC2 → Actions → Security → Modify IAM role

### 7.2. Tạo Log Group

CloudWatch Logs → Create Log Group:

```
/chinese-website/api
```

### 7.3. Thêm logging vào docker-compose

(đã có phần này ở trên)

### 7.4. Restart container

```bash
cd /opt/spring-api
docker-compose down
docker-compose up -d
```

### 7.5. Xem log

CloudWatch → /chinese-website/api → backend-api
