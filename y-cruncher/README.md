# y-cruncher คู่มือการใช้งาน

## 📋 ภาพรวม

y-cruncher เป็นโปรแกรมคำนวณค่าคงตัวทางคณิตศาสตร์ (Pi, e, Golden Ratio, etc.) ที่มีความแม่นยำสูง และใช้สำหรับ:
- **Benchmarking**: ทดสอบประสิทธิภาพ CPU และ RAM
- **Stress Testing**: ทดสอบความเสถียรของระบบ
- **World Records**: ตั้งสถิติโลกการคำนวณหลักทศนิยม

## 🚀 การติดตั้งและเตรียมระบบ

### ความต้องการระบบ

**ขั้นต่ำ:**
- CPU: x64 processor รองรับ SSE, SSE2, SSE3
- RAM: 4 GB ขึ้นไป
- Disk: 10 GB ว่าง
- OS: Linux 64-bit

**แนะนำ:**
- CPU: 8+ cores, รองรับ AVX2/AVX512
- RAM: 16 GB ขึ้นไป  
- Disk: SSD, 50 GB ว่าง
- OS: Ubuntu 20.04+ หรือ CentOS 8+

### การดาวน์โหลด

```bash
# ดาวน์โหลด Static Version (แนะนำ)
wget https://cdn.numberworld.org/y-cruncher-downloads/y-cruncher%20v0.8.6.9545-static.tar.xz

# หรือใช้ curl
curl -O "https://cdn.numberworld.org/y-cruncher-downloads/y-cruncher%20v0.8.6.9545-static.tar.xz"
```

### การแตกไฟล์

```bash
# แตกไฟล์
tar -xJf "y-cruncher v0.8.6.9545-static.tar.xz"

# เข้าไปในโฟลเดอร์
cd "y-cruncher v0.8.6.9545-static"

# ตรวจสอบไฟล์
ls -la
```

### การเรียกใช้งาน

```bash
# รันโปรแกรม
./y-cruncher

# หรือเรียกใช้แบบ root (สำหรับ large pages)
sudo ./y-cruncher
```

## 🎮 โหมดการทดสอบ

### 1. Benchmark Pi (ทดสอบเร็ว)

**สำหรับ:** ทดสอบประสิทธิภาพแบบมาตรฐาน

```bash
./y-cruncher
# เลือก Option 0: Benchmark Pi
# เลือก Option 1: Multi-Threaded (all cores)
# เลือกขนาด: 5 (500M หลัก) สำหรับ RAM 8GB
# เลือกขนาด: 6 (1B หลัก) สำหรับ RAM 16GB+
```

**เวลาประมาณ:** 1-5 นาที

### 2. Component Stress Tester (ทดสอบระบบ)

**สำหรับ:** ทดสอบความเสถียรระบบแบบรอบด้าน

```bash
./y-cruncher
# เลือก Option 2: Component Stress Tester
# เลือก Option 4: Time Limit
# กำหนด: 300 (5 นาที) หรือ 600 (10 นาที)
# เลือก Option 0: Start Stress-Testing!
```

**เวลาประมาณ:** 5-60 นาที (ตามที่กำหนด)

### 3. Custom Compute (ทดสอบกำหนดเอง)

**สำหรับ:** ทดสอบแบบระบุขนาดและเวลาเอง

```bash
./y-cruncher
# เลือก Option 4: Custom Compute a Constant
# เลือก Option 3: Decimal Digits
# กำหนดจำนวนหลัก (ดูตารางแนะนำด้านล่าง)
# เลือก Option 0: Start Computation!
```

## 📊 ตารางขนาดที่แนะนำ

### สำหรับ RAM 8 GB

| จำนวนหลัก | Memory ใช้ | เวลาประมาณ | เหมาะสำหรับ |
|-----------|------------|-------------|-------------|
| **250M** | ~1.1 GiB | 120-180 วิ | ทดสอบเร็ว |
| **500M** | ~2.2 GiB | 240-350 วิ | ทดสอบมาตรฐาน |
| **750M** | ~3.4 GiB | 350-500 วิ | ทดสอบหนัก |

### สำหรับ RAM 16 GB

| จำนวนหลัก | Memory ใช้ | เวลาประมาณ | เหมาะสำหรับ |
|-----------|------------|-------------|-------------|
| **1B** | ~4.4 GiB | 200-300 วิ | ทดสอบเร็ว |
| **2B** | ~8.8 GiB | 300-450 วิ | ทดสอบมาตรฐาน |
| **3B** | ~13.2 GiB | 400-600 วิ | ทดสอบหนัก |

### สำหรับ RAM 32 GB+

| จำนวนหลัก | Memory ใช้ | เวลาประมาณ | เหมาะสำหรับ |
|-----------|------------|-------------|-------------|
| **5B** | ~22 GiB | 500-800 วิ | ทดสอบระยะยาว |
| **10B** | ~44 GiB | 800-1,200 วิ | ทดสอบมืออาชีพ |

## 📖 การอ่านผลลัพธ์

### ผลลัพธ์หลัก

```
Total Computation Time:     435.779 seconds  ( 7.263 minutes )
Start-to-End Wall Time:     464.221 seconds  ( 7.737 minutes )
CPU Utilization:         1534.20 %  +    0.34 % kernel overhead
Multi-core Efficiency:     95.89 %  +    0.02 % kernel overhead
```

**การแปลผล:**
- **Total Computation Time**: เวลาคำนวณจริง (ไม่รวมการเขียนไฟล์)
- **Wall Time**: เวลารวมทั้งหมด
- **CPU Utilization**: เปอร์เซ็นต์การใช้ CPU รวม
- **Multi-core Efficiency**: ประสิทธิภาพการใช้ core (95%+ = ดีมาก)

### ความหมายของ CPU Utilization

| CPU Utilization | ความหมาย | สถานะ |
|-----------------|----------|--------|
| **1500%+** | ใช้งาน 15+ cores เต็มที่ | ✅ ยอดเยี่ยม |
| **800-1500%** | ใช้งาน 8-15 cores | ✅ ดี |
| **400-800%** | ใช้งาน 4-8 cores | ⚠️ ปกติ |
| **< 400%** | ใช้งาน < 4 cores | ❌ มีปัญหา |

### การแบ่งเวลาตามขั้นตอน

```
Series CommonP2B3...       370.252 seconds  (85.0%)
Large Division...           18.094 seconds   (4.1%)
Base Converting...          27.893 seconds   (6.4%)
InvSqrt(10005)...          11.163 seconds   (2.6%)
Large Multiply...           8.360 seconds   (1.9%)
```

**การวิเคราะห์:**
- **Series Computation**: ใช้เวลามากที่สุด (80-90%)
- **ขั้นตอนอื่น**: ใช้เวลารวม 10-20%

### การตรวจสอบความถูกต้อง

```
Spot Check: Good through 2,500,000,000
Validation File: Pi - 20250601-134534.txt
```

**สถานะ:**
- **"Good through X"**: ผลลัพธ์ถูกต้องถึงหลักที่ X
- **Validation File**: ไฟล์ยืนยันผลลัพธ์

## 🔍 การตรวจสอบระหว่างทดสอบ

### เปิด Terminal ใหม่

```bash
# ดู CPU usage realtime
htop

# ดู memory usage
watch -n1 'free -h'

# ดู disk I/O
iostat -x 1

# ดูอุณหภูมิ (ถ้ามี sensors)
watch -n5 sensors
```

### สัญญาณเตือน

**🔴 หยุดทดสอบทันทีถ้า:**
- อุณหภูมิ CPU > 85°C
- Memory usage > 95%
- ระบบค้าง/ไม่ตอบสนอง

**⚠️ ระวังถ้า:**
- CPU frequency ลดลง (thermal throttling)
- swap usage เพิ่มขึ้น
- I/O wait time สูง

## 🛠️ การแก้ไขปัญหา

### ปัญหา Out of Memory

```bash
# ตรวจสอบ memory ว่าง
free -h

# เพิ่ม swap (ถ้าจำเป็น)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### ปัญหา Dependency (Dynamic Version)

```bash
# Ubuntu/Debian
sudo apt install libatomic1 libtbb-dev libnuma-dev

# CentOS/RHEL
sudo yum install libatomic libtbb-devel numactl-devel
```

### ปัญหา Permission

```bash
# รันแบบ root สำหรับ large pages
sudo ./y-cruncher

# หรือเพิ่ม permission
sudo setcap cap_ipc_lock=+ep ./y-cruncher
```

## 📈 การเปรียบเทียบประสิทธิภาพ

### การวัดประสิทธิภาพ

**สูตรการคำนวณ:**
- **Speed per Core**: เวลารวม ÷ จำนวน cores
- **Memory Efficiency**: Memory ใช้ ÷ จำนวนหลัก
- **Performance Index**: จำนวนหลัก ÷ (เวลา × cores)

### ตัวอย่างผลลัพธ์ที่ดี

**ระบบ 16 cores, 3B หลัก:**
```
เวลา: 435 วินาที
CPU Efficiency: 95.89%
Speed per Core: 27.2 วินาที/core
Memory: 13.2 GiB / 3B หลัก = 4.4 MB/M หลัก
```

## 📁 ไฟล์ Output

### ไฟล์ที่สร้าง

```
Pi - 20250601-134534.txt     # Validation file
pi-dec-3000000000.txt        # หลักทศนิยม (ถ้าเปิดใช้)
pi-hex-2491446072.txt        # หลักฐานสิบหก (ถ้าเปิดใช้)
```

### การจัดเก็บ

```bash
# สร้างโฟลเดอร์สำหรับผลลัพธ์
mkdir ~/y-cruncher-results
mv Pi*.txt ~/y-cruncher-results/

# บีบอัดเพื่อประหยัดพื้นที่
gzip ~/y-cruncher-results/*.txt
```

## 🎯 Use Cases แนะนำ

### สำหรับ System Administrator
```bash
# ทดสอบความเสถียรเซิร์ฟเวอร์ใหม่
Component Stress Test 30 นาที
```

### สำหรับ Overclocker  
```bash
# ทดสอบ RAM และ CPU หลัง OC
Custom Pi 1-2B หลัก
```

### สำหรับ Hardware Reviewer
```bash
# เปรียบเทียบประสิทธิภาพ CPU
Benchmark Pi 500M-1B หลัก
```

### สำหรับ Data Center
```bash
# Burn-in test เซิร์ฟเวอร์
Custom Pi 5-10B หลัก หรือ Stress Test 60+ นาที
```

## ⚠️ ข้อควรระวัง

### การใช้งาน
- **อุณหภูมิ**: ตรวจสอบการระบายความร้อน
- **ไฟฟ้า**: ใช้ UPS ป้องกันไฟดับ
- **SSD**: หลีกเลี่ยง disk mode ถ้าไม่จำเป็น
- **เวลา**: การทดสอบอาจใช้เวลานานหลายชั่วโมง

### ข้อมูล
- **ลิขสิทธิ์**: อย่านำผลลัพธ์ไปใช้เชิงพาณิชย์
- **พื้นที่**: ไฟล์ผลลัพธ์อาจมีขนาดใหญ่มาก
- **แชร์**: สามารถแชร์ validation file ได้

## 🔗 ลิงก์ที่เป็นประโยชน์

- **Official Website**: https://www.numberworld.org/y-cruncher/
- **Download Page**: https://www.numberworld.org/y-cruncher/
- **GitHub**: https://github.com/Mysticial/y-cruncher
- **Forum**: https://mersenneforum.org/forumdisplay.php?f=111

## 📝 License

y-cruncher เป็น freeware สำหรับการใช้งานส่วนบุคคลและการศึกษา
การใช้งานเชิงพาณิชย์ต้องติดต่อผู้พัฒนา

---

**เขียนโดย:** AI Assistant  
**อัปเดตล่าสุด:** มิถุนายน 2025  
**เวอร์ชัน:** y-cruncher v0.8.6 Build 9545