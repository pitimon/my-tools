# Hey: เครื่องมือทดสอบประสิทธิภาพ HTTP

Hey เป็นเครื่องมือทดสอบประสิทธิภาพ HTTP ที่พัฒนาด้วยภาษา Go โดย Jaana Dogan (rakyll) จาก Google ซึ่งถูกออกแบบมาให้ใช้งานง่าย มีความยืดหยุ่น และมีประสิทธิภาพสูง สำหรับทดสอบ web service และ API

## คุณสมบัติหลัก

- **น้ำหนักเบา**: เขียนด้วย Go ทำให้ประหยัดทรัพยากรและทำงานได้รวดเร็ว
- **ใช้งานง่าย**: คำสั่งไม่ซับซ้อน เหมาะสำหรับการทดสอบเบื้องต้น
- **รองรับการทดสอบแบบขนาน**: สามารถจำลองผู้ใช้งานพร้อมกันได้หลายคน (concurrent users)
- **วิเคราะห์ผลละเอียด**: แสดงสถิติที่ครอบคลุมเกี่ยวกับประสิทธิภาพ

## การติดตั้ง

```bash
# ใช้ Go สำหรับติดตั้ง
go install github.com/rakyll/hey@latest

# หรือ ด้วย apt (Linux)
sudo apt install hey

# หรือ ด้วย Homebrew (macOS)
brew install hey

# หรือดาวน์โหลด binary จาก GitHub releases
```

## การใช้งานพื้นฐาน

รูปแบบคำสั่งทั่วไป:

```bash
hey [options] URL
```

### ตัวอย่างคำสั่งพื้นฐาน

```bash
# ทดสอบงานง่ายๆ - 200 requests กับ 10 concurrent users
hey -n 200 -c 10 https://example.com/api/endpoint
```

### ตัวเลือกที่ใช้บ่อย

```
-n  จำนวน requests ทั้งหมด (default: 200)
-c  จำนวน concurrent requests (default: 50)
-q  rate limit (requests/sec) (default: ไม่จำกัด)
-z  ระยะเวลาทดสอบ (เช่น 10s, 3m) (ใช้ -z หรือ -n อย่างใดอย่างหนึ่ง)
-m  HTTP method (GET, POST, PUT, DELETE, ...) (default: GET)
-H  HTTP header (เช่น "Authorization: Bearer token123")
-T  Content-type (default: "text/html")
-d  ข้อมูลสำหรับ request body (สำหรับ POST/PUT)
-D  ใช้ไฟล์เป็น request body
```

### ตัวอย่างการใช้งานขั้นสูง

```bash
# ทดสอบ API ด้วย Authorization header และ JSON content-type
hey -n 1000 -c 50 \
    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
    -H "Content-Type: application/json" \
    https://api.example.com/users

# ทดสอบ POST request ด้วย JSON data
hey -n 500 -c 20 -m POST \
    -H "Content-Type: application/json" \
    -d '{"email":"test@example.com"}' \
    https://api.example.com/auth/login

# ทดสอบตามระยะเวลา (แทนที่จะใช้จำนวน requests)
hey -z 30s -c 100 https://api.example.com/status
```

## การอ่านและวิเคราะห์ผลลัพธ์

เมื่อทดสอบเสร็จสิ้น Hey จะแสดงรายงานโดยละเอียด ซึ่งมีหัวข้อหลักดังนี้:

### 1. สรุปภาพรวม (Summary)

```
Summary:
  Total:        8.8521 secs
  Slowest:      0.8509 secs
  Fastest:      0.0147 secs
  Average:      0.1305 secs
  Requests/sec: 112.9718
```

- **Total**: ระยะเวลารวมที่ใช้ในการทดสอบ
- **Slowest**: เวลาตอบสนองที่ช้าที่สุด
- **Fastest**: เวลาตอบสนองที่เร็วที่สุด
- **Average**: เวลาตอบสนองเฉลี่ย
- **Requests/sec**: จำนวน requests ต่อวินาที (throughput)

### 2. การกระจายตัวของเวลาตอบสนอง (Response time distribution)

```
Response time histogram:
  0.015 [1]     |
  0.098 [324]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.182 [452]   |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.266 [93]    |■■■■■■■
  0.349 [18]    |■
  0.433 [6]     |
  0.517 [4]     |
  0.600 [0]     |
  0.684 [0]     |
  0.767 [0]     |
  0.851 [2]     |
```

นี่เป็นกราฟแท่งที่แสดงการกระจายตัวของเวลาตอบสนอง ตัวเลขในวงเล็บคือจำนวน requests ที่มีเวลาตอบสนองอยู่ในช่วงนั้น ช่วยให้เห็นว่าส่วนใหญ่ของ requests มีประสิทธิภาพเป็นอย่างไร

### 3. ละเอียดเปอร์เซ็นต์ไทล์ (Latency distribution)

```
Latency distribution:
  10% in 0.0634 secs
  25% in 0.0780 secs
  50% in 0.1131 secs
  75% in 0.1631 secs
  90% in 0.2267 secs
  95% in 0.2763 secs
  99% in 0.4272 secs
```

- **X% in Y secs**: X% ของ requests ใช้เวลาไม่เกิน Y วินาที
- ค่า 50% คือ **ค่ามัธยฐาน** (median) หรือค่ากลาง
- ค่า 95% และ 99% (**P95, P99**) สำคัญมากในการวัดประสิทธิภาพ เพราะแสดงประสบการณ์ของผู้ใช้ที่แย่ที่สุด

### 4. รายละเอียด HTTP Status (HTTP Response Status Code)

```
Status code distribution:
  [200] 895 responses
  [404] 5 responses
```

แสดงการกระจายตัวของรหัสสถานะ HTTP ช่วยให้เห็นว่ามี errors หรือไม่ในระหว่างการทดสอบ

### 5. รายละเอียดทางเทคนิค (Details)

```
Details (average, fastest, slowest):
  DNS+dialup:   0.0001 secs, 0.0000 secs, 0.0022 secs
  DNS-lookup:   0.0000 secs, 0.0000 secs, 0.0000 secs
  req write:    0.0000 secs, 0.0000 secs, 0.0003 secs
  resp wait:    0.1303 secs, 0.0146 secs, 0.8508 secs
  resp read:    0.0001 secs, 0.0000 secs, 0.0003 secs
```

แสดงรายละเอียดเวลาที่ใช้ในแต่ละขั้นตอนของการส่ง request:
- **DNS+dialup**: เวลาที่ใช้ในการเชื่อมต่อ
- **DNS-lookup**: เวลาที่ใช้ในการค้นหา DNS
- **req write**: เวลาที่ใช้ในการเขียน request
- **resp wait**: เวลาที่รอ response (ส่วนที่สำคัญที่สุด - แสดงเวลาประมวลผลของเซิร์ฟเวอร์)
- **resp read**: เวลาที่ใช้ในการอ่าน response

## การนำข้อมูลไปใช้

1. **เปรียบเทียบข้อมูลก่อน-หลัง**: ใช้ผลลัพธ์จาก Hey เปรียบเทียบประสิทธิภาพก่อนและหลังการปรับปรุงระบบ (เช่น การเพิ่ม Redis)

2. **มุ่งเน้นที่ค่า P95 และ P99**: ค่าเปอร์เซ็นต์ไทล์ที่ 95% และ 99% สำคัญมากเพราะสะท้อนประสบการณ์ของผู้ใช้ส่วนใหญ่

3. **วิเคราะห์ความสามารถในการรองรับภาระงาน**: ค่า Requests/sec แสดงถึงความสามารถในการรองรับภาระงานของระบบ

4. **ตรวจสอบ Error rates**: สัดส่วนของรหัสสถานะ error (4xx, 5xx) ที่เพิ่มขึ้นเมื่อภาระงานสูงขึ้นบ่งบอกถึงจุดอ่อนของระบบ

## ตัวอย่างกรณีศึกษา: วัดประสิทธิภาพเมื่อใช้ Redis

```bash
# ก่อนเพิ่ม Redis
hey -n 1000 -c 50 -H "Authorization: Bearer [TOKEN]" http://localhost:8080/api/auth/domains

# หลังเพิ่ม Redis
hey -n 1000 -c 50 -H "Authorization: Bearer [TOKEN]" http://localhost:8080/api/auth/domains
```

### ตารางเปรียบเทียบ (ตัวอย่าง)

| เมตริก | ก่อนใช้ Redis | หลังใช้ Redis | การปรับปรุง |
|--------|--------------|--------------|-------------|
| Average Response Time | 120ms | 25ms | 79.2% ดีขึ้น |
| P95 Response Time | 180ms | 40ms | 77.8% ดีขึ้น |
| Requests/sec | 400 | 1,800 | 350% ดีขึ้น |
| Error Rate | 0.5% | 0% | 100% ดีขึ้น |

Hey เป็นเครื่องมือที่ทรงพลังแต่ใช้งานง่าย เหมาะสำหรับการทดสอบประสิทธิภาพขั้นต้นของ API และ web services โดยเฉพาะอย่างยิ่งกับระบบที่เขียนด้วย Go เนื่องจากมีแนวคิดการออกแบบที่คล้ายคลึงกัน

---