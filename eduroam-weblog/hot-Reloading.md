โครงสร้างการพัฒนาแบบ Hot-Reloading

  โปรเจคนี้ได้ถูกตั้งค่าให้ทำงานแบบ development mode ที่รองรับ hot-reloading ซึ่งหมายความว่าคุณไม่จำเป็นต้อง rebuild หรือ restart containers ทุกครั้งที่มีการแก้ไขโค้ด

  สิ่งที่ทำให้การพัฒนาแบบนี้เป็นไปได้:

  1. Volume Mounts ที่เหมาะสม:
    - ในไฟล์ docker-compose.yml มีการ mount โฟลเดอร์ source code เข้าไปยัง container:
    volumes:
    - ./frontend:/app
    - /app/node_modules
        - โค้ดจากเครื่องโฮสต์ถูก mount เข้าไปยัง container
      - node_modules ใน container ถูกเก็บไว้ (ไม่ถูกแทนที่)
  2. Dockerfile.dev สำหรับสภาพแวดล้อมการพัฒนา:
    - ไม่มีการ copy source files เข้าไปใน image เพราะจะใช้ volumes แทน
    - มีเฉพาะการติดตั้ง dependencies เท่านั้น
  3. Development Scripts ที่รองรับ Hot-Reloading:
    - Frontend: ใช้ npm start ซึ่งเป็น React development server พร้อม hot-reloading
    - Backend: ใช้ npm run dev ซึ่งใช้ nodemon เพื่อ restart server เมื่อไฟล์เปลี่ยนแปลง
  4. การตั้งค่า Hot-Reloading:
    - Frontend: มีการตั้งค่า CHOKIDAR_USEPOLLING=true เพื่อให้ React ตรวจจับการเปลี่ยนแปลงไฟล์ได้ใน Docker
    - Backend: ใช้ nodemon ซึ่งจะตรวจจับการเปลี่ยนแปลงของไฟล์โดยอัตโนมัติ

  วิธีใช้สภาพแวดล้อมนี้:

  1. เริ่มต้นครั้งแรก (ทำเพียงครั้งเดียว):
  docker compose build
  docker compose up -d
  2. ระหว่างการพัฒนา:
    - แก้ไขโค้ด frontend หรือ backend ได้โดยตรงที่โฟลเดอร์ในเครื่องโฮสต์
    - การเปลี่ยนแปลงจะถูกตรวจจับและทำให้แอปรีโหลดโดยอัตโนมัติ:
        - Frontend: React ใช้ webpack dev server จะ hot-reload component ที่เปลี่ยนแปลง
      - Backend: nodemon จะ restart Node.js server เมื่อมีการเปลี่ยนแปลงไฟล์
  3. เมื่อมีการเปลี่ยนแปลง dependencies:
    - ถ้าเพิ่ม/แก้ไข packages ใน package.json คุณจำเป็นต้อง rebuild:
  docker compose build <service_name> # เช่น frontend หรือ backend
  docker compose up -d <service_name>
  4. การดูลอก:
  docker compose logs -f
  5. การหยุดระบบ:
  docker compose down

  ข้อควรระวัง:

  1. Database Schema Changes: ถ้ามีการเปลี่ยนแปลง schema คุณอาจต้องรัน migrations:
  docker compose exec backend npm run migrate
  2. การแก้ไข Port Mapping หรือ Environment Variables: จะต้อง restart services:
  docker compose up -d --force-recreate
  3. การแก้ไข Dockerfile หรือ docker-compose.yml: จะต้อง rebuild และ restart:
  docker compose down
  docker compose build
  docker compose up -d

  ด้วยการตั้งค่านี้ คุณสามารถแก้ไขโค้ดและเห็นการเปลี่ยนแปลงทันทีโดยไม่ต้อง rebuild หรือ restart Docker containers ทำให้การพัฒนาเร็วขึ้นมาก
