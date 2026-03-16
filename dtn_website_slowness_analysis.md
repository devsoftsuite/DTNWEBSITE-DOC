# 🔍 วิเคราะห์ปัญหาเว็บไซต์ช้า — Server 172.16.2.62

> วันที่วิเคราะห์: 16 มีนาคม 2026, 17:09 น.

---

## 🚨 สาเหตุหลัก: Table-Level Lock บน `content_article_info`

### หลักฐานจาก SHOW PROCESSLIST

| Id   | Command | Time (วินาที) | State                          | Query                                               |
|------|---------|--------------|--------------------------------|-----------------------------------------------------|
| 8395 | Execute | **185**      | `Sending data`                 | `SELECT count(T.ID) ... FROM tblcontent_info` (slow) |
| 8747 | Execute | **117**      | `Waiting for table level lock` | `UPDATE content_article_info SET ViewAmount=...`   |
| 8973 | Execute | **72**       | `Waiting for table level lock` | `UPDATE content_article_info SET ViewAmount=...`   |
| 8993 | Execute | 67           | `Waiting for table level lock` | `SELECT ... content_article_info`                  |
| 9215 | Execute | 23           | `Waiting for table level lock` | `SELECT count ...`                                  |
| 9233 | Execute | 21           | `Waiting for table level lock` | `SELECT count ...`                                  |

**สรุป:** Query Id 8395 ค้างอยู่ **185 วินาที** กำลัง `SELECT count(*)` แบบ Full Table Scan บน `content_article_info` ทำให้ Query ที่ต้อง `UPDATE ViewAmount` ทุกตัวต้องรอ table-level lock จนเป็น chain reaction ปิดกั้น request ใหม่ทั้งหมด

---

## 🔍 สาเหตุที่พบอื่นๆ (Secondary Issues)

### 1. MySQL "Server has gone away"
```
PHP Fatal error: Zend_Db_Statement_Mysqli_Exception: 
Mysqli prepare error: MySQL server has gone away
```
- เกิดขึ้นซ้ำในช่วงก่อน reboot (ก่อน 16:41)
- Connection ถูกตัดโดย MySQL เพราะ query ค้างนานเกินค่า `wait_timeout`

### 2. PHP Execution Timeout
```
PHP Fatal error: Maximum execution time of 30 seconds exceeded
in Zend/Db/Statement/Mysqli.php on line 208
```
- PHP timeout 30 วินาที ถูก trigger เพราะรอ DB lock

### 3. Apache MPM Prefork: MaxRequestWorkers เต็ม
```
WARNING: MaxRequestWorkers of 300 exceeds ServerLimit value of 256
```
- มี **57 httpd processes** รันอยู่ (บวก grep = 58)
- Config ตั้ง MaxRequestWorkers=300 แต่ limit จริงที่ 256
- เมื่อ request รอ DB lock → process ถูกกิน → slot Apache หมด → เว็บช้า/ไม่ตอบสนอง

### 4. DNS ภายในไม่สามารถ resolve ชื่อ external ได้
```
curl: (6) Could not resolve host: notify-api.line.me; Unknown error
nslookup notify-api.line.me 10.2.1.12 → NXDOMAIN
```
- DNS Server ภายใน `10.2.1.12` ไม่มี record ของ `notify-api.line.me`
- แต่ Google DNS `8.8.8.8` resolve ได้ปกติ
- หาก มีฟีเจอร์ Line Notify ที่ต้องเรียก API ออก internet → **อาจทำให้ request ค้าง**

### 5. /home disk ใกล้เต็ม
```
/dev/mapper/centos-home   192G  186G  5.2G  98%  /home
```
- Website และ uploads อยู่ใน `/home/website/` → เหลือพื้นที่เพียง 5.2GB
- เสี่ยง disk full ซึ่งจะทำให้ session/file fail ทันที

---

## 🌐 สถานะปัจจุบัน (ณ เวลาวิเคราะห์)

| รายการ | ค่า |
|--------|-----|
| Uptime | 21 นาที (เพิ่ง reboot) |
| Load Average | 0.15, 0.36, 0.53 (ปกติ) |
| RAM ว่าง | ~12.7 GB จาก 16 GB |
| DB Connection | เชื่อมต่อได้ปกติ (0.237s) |
| httpd processes | 58 |
| MPM | prefork |

---

## ✅ แนวทางแก้ไข (เรียงตามความเร่งด่วน)

### 🔴 ด่วนมาก — แก้สาเหตุหลัก

#### 1. เพิ่ม Index บน `content_article_info`
Query `SELECT count(T.ID) FROM tblcontent_info` ทำ Full Table Scan เพราะขาด index

```sql
-- เชื่อมต่อที่ DB Server 172.16.2.63
-- ตรวจ index ที่มี
SHOW INDEX FROM content_article_info;

-- เพิ่ม index ที่น่าจะช่วย
ALTER TABLE content_article_info ADD INDEX idx_siteid (SiteID);
ALTER TABLE content_article_info ADD INDEX idx_siteid_id (SiteID, ID);
```

#### 2. แก้ปัญหา ViewAmount UPDATE — ใช้ MyISAM → InnoDB
Table-level lock เกิดจาก MyISAM engine ให้ตรวจ:
```sql
SHOW CREATE TABLE content_article_info;
```
หาก engine เป็น **MyISAM** → เปลี่ยนเป็น **InnoDB** เพื่อให้ใช้ row-level lock แทน:
```sql
-- ทำในช่วง traffic น้อย
ALTER TABLE content_article_info ENGINE=InnoDB;
ALTER TABLE tblcontent_info ENGINE=InnoDB;
```
> ⚠️ ก่อน ALTER ให้ backup ก่อนทุกครั้ง

#### 3. ลอง optimize ViewAmount ด้วย async / background update
แทนที่จะ UPDATE ทันทีทุก pageview ให้พิจารณา:
- บันทึก ViewAmount ด้วย queue หรือ cron job
- หรือลดความบ่อยของการ update

---

### 🟡 สำคัญ — แก้ภายในสัปดาห์นี้

#### 4. แก้ DNS: เพิ่ม nameserver backup หรือเปลี่ยน resolv.conf
```bash
# /etc/resolv.conf ปัจจุบัน:
# nameserver 10.2.1.12  ← resolve external domain ไม่ได้
# nameserver 2001:4860:4860::8888

# แก้ไข: เพิ่ม Google DNS IPv4 เป็น fallback
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```
> หรือแจ้ง Admin เครือข่ายให้เพิ่ม record ของ Line Notify ใน internal DNS

#### 5. แก้ Apache: เพิ่ม ServerLimit พร้อม MaxRequestWorkers
```bash
# แก้ไฟล์ /etc/httpd/conf/httpd.conf
# ในส่วน <IfModule mpm_prefork_module>
ServerLimit    300        # เพิ่มจาก 256 → 300
MaxRequestWorkers 300     # ให้ตรงกัน
```
```bash
apachectl configtest && systemctl reload httpd
```

#### 6. ทำความสะอาด /home disk
```bash
# ดูขนาด directory
du -sh /home/website/dtn/tmp/*
du -sh /home/website/dtn/public/uploads/*

# ลบ session เก่า
find /home/website/dtn/tmp/session/ -mtime +7 -delete
find /home/website/dtn/tmp/mail/ -mtime +7 -delete
```

---

### 🟢 แนะนำระยะยาว

#### 7. สร้าง Monitoring
- ตั้ง cron ตรวจ MySQL slow query ทุก 5 นาที
- เพิ่ม `slow_query_log` ใน MySQL config ที่ DB server
- พิจารณาใช้ `mod_status` เพื่อ monitor Apache worker

#### 8. พิจารณา OPcache สำหรับ PHP
```bash
# ตรวจว่า OPcache เปิดใช้อยู่หรือไม่
php -r "echo opcache_get_status()['opcache_enabled'] ? 'ON' : 'OFF';"
```

---

## 📋 คำสั่งแก้ไขเร่งด่วน (Emergency Fix)

สามารถรันตอนนี้ได้เลยเพื่อบรรเทาปัญหาชั่วคราว:

```bash
# SSH เข้า DB Server หรือรันจาก web server
php -r '
$m = new mysqli("172.16.2.63", "dtn", "dtn#P@ssw0rd", "dtn_d2msp_db");
$r = $m->query("SHOW PROCESSLIST");
while($row = $r->fetch_assoc()) {
    if ($row["Time"] > 60 && $row["Command"] == "Execute") {
        echo "KILLING: " . $row["Id"] . " (". $row["Time"] . "s)\n";
        $m->query("KILL " . $row["Id"]);
    }
}
'
```

ถ้าจะทำระยะยาวให้ตั้ง MySQL `max_execution_time` และ `wait_timeout` ที่เหมาะสม:
```sql
SET GLOBAL wait_timeout = 60;
SET GLOBAL interactive_timeout = 60;
SET GLOBAL max_execution_time = 30000; -- 30 seconds (milliseconds)
```

