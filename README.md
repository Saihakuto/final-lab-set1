# ENGSE207 Final Lab Set 1 — Microservices + HTTPS + Basic Logging

**วิชา:** ENGSE207 Software Architecture  
**ชื่องาน:** Final Lab Set 1  
**ผู้จัดทำ:** นายทวีชัย ทิใจ รหัส 67543210029-4  
**มหาวิทยาลัย:** มหาวิทยาลัยเทคโนโลยีราชมงคลล้านนา เชียงใหม่

**กลุ่มที่:** 8

---

## ภาพรวม

ระบบ Task Board Microservices ที่ประกอบด้วย Auth Service, Task Service, Log Service, Frontend และ Nginx เป็น API Gateway รองรับ HTTPS ด้วย Self-Signed Certificate และบันทึก Log ลงฐานข้อมูลผ่าน REST API

---

## Architecture Diagram
```
Browser / Postman
       │
       │ HTTPS :443  (HTTP :80 → redirect HTTPS)
       ▼
┌─────────────────────────────────────────────┐
│  Nginx (API Gateway + TLS + Rate Limiter)   │
│  /api/auth/*   → auth-service:3001          │
│  /api/tasks/*  → task-service:3002          │
│  /api/logs/*   → log-service:3003           │
│  /             → frontend:80                │
└──────┬──────────────┬───────────────┬───────┘
       │              │               │
       ▼              ▼               ▼
  auth-service   task-service    log-service
    :3001           :3002           :3003
       │              │               │
       └──────────────┴───────────────┘
                      │
               PostgreSQL :5432
```

---

## โครงสร้าง Repository
```
final-lab-set1/
├── nginx/           # HTTPS reverse proxy config
├── frontend/        # Static HTML UI
├── auth-service/    # Login + JWT
├── task-service/    # CRUD Tasks
├── log-service/     # Logging API
├── db/init.sql      # Schema + Seed Users
├── scripts/         # gen-certs.sh
└── screenshots/     # หลักฐานการทดสอบ
```

---

## วิธีรันระบบ
```bash
# 1. สร้าง SSL Certificate
MSYS_NO_PATHCONV=1 docker run --rm \
  -v "/path/to/nginx/certs:/certs" alpine/openssl \
  req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /certs/key.pem -out /certs/cert.pem \
  -subj "/C=TH/ST=ChiangMai/O=RMUTL/CN=localhost"

# 2. สร้างไฟล์ .env
cp .env.example .env

# 3. Build และรัน
docker compose up --build

# 4. เปิด browser
# https://localhost  (กด Advanced → Proceed)
```

---

## Seed Users

| Username | Email | Password | Role |
|----------|-------|----------|------|
| alice | alice@lab.local | alice123 | member |
| bob | bob@lab.local | bob456 | member |
| admin | admin@lab.local | adminpass | admin |

---

## วิธีทดสอบ API
```bash
# Login
curl -sk -X POST https://localhost/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@lab.local","password":"alice123"}'

# Get Tasks (ต้องมี JWT)
TOKEN=<token_จาก_login>
curl -sk https://localhost/api/tasks/ \
  -H "Authorization: Bearer $TOKEN"
```

---

## HTTPS Flow

1. Browser เรียก `http://localhost` → Nginx redirect ไป `https://localhost`
2. Nginx ทำ TLS Termination ด้วย Self-Signed Certificate
3. Request ส่งต่อไปยัง service ภายใน Docker network (HTTP)

## JWT Flow

1. Client POST `/api/auth/login` → auth-service ตรวจ bcrypt → ออก JWT
2. Client ส่ง `Authorization: Bearer <token>` ทุก request
3. task-service ตรวจ JWT ก่อนทำ CRUD

## Logging Flow

1. auth-service และ task-service เรียก `POST /api/logs/internal` ภายใน Docker network
2. log-service บันทึกลง PostgreSQL
3. Admin ดู log ผ่าน `GET /api/logs/` (ต้องมี JWT + role=admin)
4. Nginx บล็อก `/api/logs/internal` จากภายนอก (return 403)

---

## Known Limitations

- ใช้ Self-Signed Certificate → browser แสดง warning
- ไม่มีระบบ Register — ใช้ Seed Users เท่านั้น
- Shared Database — ทุก service ใช้ PostgreSQL ตัวเดียวกัน
- Log Service ไม่มี authentication สำหรับ internal endpoint
