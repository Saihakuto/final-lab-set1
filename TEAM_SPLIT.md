# TEAM_SPLIT.md

## ข้อมูลกลุ่ม
- **กลุ่มที่:** 8
- **รายวิชา:** ENGSE207 Software Architecture

## รายชื่อสมาชิก
- 67543210029-4 นายทวีชัย ทิใจ

---

## การแบ่งงานหลัก

### นายทวีชัย ทิใจ (67543210029-4) — รับผิดชอบทั้งหมด

- **Auth Service** — login route, bcrypt password verify, JWT generation, logEvent helper
- **Task Service** — CRUD routes (GET/POST/PUT/DELETE), JWT authMiddleware, logEvent helper
- **Log Service** — POST /internal endpoint, GET /logs admin API, GET /stats
- **Log Service** — requireAdmin middleware ตรวจ JWT + role
- **Frontend** — index.html Task Board UI, logs.html Log Dashboard
- **Nginx** — HTTPS config, Self-Signed Certificate, reverse proxy, rate limiting, block /api/logs/internal
- **Docker Compose** — orchestration, healthcheck, network, environment variables
- **Database** — init.sql schema (users, tasks, logs tables), seed users
- **Testing** — ทดสอบทุก test case T1-T11 ด้วย curl และ browser

---

## งานที่ดำเนินการร่วมกัน
- ออกแบบ Architecture diagram
- ทดสอบระบบแบบ end-to-end
- จัดทำ README และ screenshots

---

## เหตุผลในการแบ่งงาน
ดำเนินการโดยสมาชิกเพียงคนเดียว จึงรับผิดชอบทุกส่วนของระบบ
โดยพัฒนาตามลำดับ service boundary ได้แก่ Database → Auth → Task → Log → Nginx → Frontend → Docker Compose

---

## สรุปการเชื่อมโยงงาน

Auth Service และ Task Service เชื่อมต่อกันผ่าน JWT Secret ที่ใช้ร่วมกันใน environment variable
Task Service เรียก Log Service ผ่าน `POST http://log-service:3003/api/logs/internal` ภายใน Docker network
Nginx เป็น single entry point รับ request จากภายนอกและ route ไปยัง service ที่ถูกต้อง
ทุก service เชื่อมต่อ PostgreSQL ผ่าน Docker network ชื่อ taskboard-net