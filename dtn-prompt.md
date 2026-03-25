## DTN Prompt 

## เว็บไซต์ช้าตอนเข้าใช้งาน
### Prompt 1.1
```
Environment:
- OS: CentOS Linux 7 (Core)
- Hypervisor: VMware
- CPU: Intel(R) Xeon(R) CPU E5-2697 v3 @ 2.60GHz  8 Core
- RAM: 16 GB
- Web service: Apache
- php version: 7.3.33
- Zend Engine: 3.3.33
- Server IP: 172.16.2.62
- Database IP: 172.16.2.63

Website path:
- /home/website/dtn/public
- /home/website/dtn/applicationadmin
- /home/website/dtn/applicationfront

Code:
- D:\Works\DTNWEBSITE

log path (Maybe) : /var/log/message

Problem:
1. เว็บไซต์ช้าตอนเข้าใช้งาน
2. sudo reboot แล้วเร็วขึ้น
3. ssh เข้าไปตรวจสอบได้
```

### Summary 1.1
```
ปัญหา
- เว็บไซต์ dtn.go.th ทำงานช้าผิดปกติในช่วงที่มีผู้ใช้งานพร้อมกันจำนวนมาก โดยพบว่าการ Reboot server ทำให้เว็บไซต์กลับมาทำงานได้ปกติ แต่ปัญหาเกิดซ้ำขึ้นอีกในภายหลัง

สาเหตุ
- ตารางฐานข้อมูล content_article_info ใช้ประเภท MyISAM การล็อคทั้งตาราง ทุกครั้งที่มีการ Query ขนาดใหญ่ ส่งผลให้คำสั่งอื่นๆ ที่เข้ามาพร้อมกันทั้งหมดต้องหยุดรอ จนเกิดการสะสมของ Request ที่ค้าง และทำให้ระบบตอบสนองช้าหรือไม่ตอบสนอง

ขั้นตอนการตรวจสอบ
1. ตรวจสอบ Processlist ของฐานข้อมูลด้วยคำสั่ง SHOW PROCESSLIST พบ Query ค้างนาน 185 วินาทีในสถานะ Sending data และมี Query อีก 8 รายการอยู่ในสถานะ Waiting for table level lock พร้อมกัน
2. ตรวจสอบโครงสร้างตารางด้วยคำสั่ง SHOW CREATE TABLE content_article_info แสดงข้อมูล Engine เป็น MyISAM

วิธีการแก้ไข
1. เปลี่ยนประเภท Engine ของตาราง content_article_info จาก MyISAM เป็น InnoDB ด้วยคำสั่ง
ALTER TABLE content_article_info ENGINE=InnoDB;
เนื่องจาก InnoDB รองรับ Row-Level Lock ทำให้คำสั่งต่างๆ ทำงานพร้อมกันได้โดยไม่ต้องรอกัน

ผลการตรวจสอบหลังแก้ไข
- ตรวจสอบ Processlist หลังดำเนินการเปลี่ยน Engine ของตาราง content_article_info แล้ว พบว่าไม่มี Query ค้างหรืออยู่ในสถานะรอ Table Lock
```

```
ช่วยสรุปตามรูปแบบนี้ได้ไหมครับ

ปัญหา
- ปัญหา

สาเหตุ
- สาเหตุ

ขั้นตอนการตรวจสอบ
1. ขั้นตอนการตรวจสอบ 1
2. ขั้นตอนการตรวจสอบ 2

วิธีการแก้ไข
1. วิธีการแก้ไข

ผลการตรวจสอบหลังแก้ไข
- ผลการตรวจสอบหลังแก้ไข

ตัวอย่าง
ปัญหา
- เว็บไซต์ dtn.go.th ทำงานช้าผิดปกติในช่วงที่มีผู้ใช้งานพร้อมกันจำนวนมาก โดยพบว่าการ Reboot server ทำให้เว็บไซต์กลับมาทำงานได้ปกติ แต่ปัญหาเกิดซ้ำขึ้นอีกในภายหลัง

สาเหตุ
- ตารางฐานข้อมูล content_article_info ใช้ประเภท MyISAM การล็อคทั้งตาราง ทุกครั้งที่มีการ Query ขนาดใหญ่ ส่งผลให้คำสั่งอื่นๆ ที่เข้ามาพร้อมกันทั้งหมดต้องหยุดรอ จนเกิดการสะสมของ Request ที่ค้าง และทำให้ระบบตอบสนองช้าหรือไม่ตอบสนอง

ขั้นตอนการตรวจสอบ
1. ตรวจสอบ Processlist ของฐานข้อมูลด้วยคำสั่ง SHOW PROCESSLIST พบ Query ค้างนาน 185 วินาทีในสถานะ Sending data และมี Query อีก 8 รายการอยู่ในสถานะ Waiting for table level lock พร้อมกัน
2. ตรวจสอบโครงสร้างตารางด้วยคำสั่ง SHOW CREATE TABLE content_article_info แสดงข้อมูล Engine เป็น MyISAM

วิธีการแก้ไข
1. เปลี่ยนประเภท Engine ของตาราง content_article_info จาก MyISAM เป็น InnoDB ด้วยคำสั่ง
ALTER TABLE content_article_info ENGINE=InnoDB;
เนื่องจาก InnoDB รองรับ Row-Level Lock ทำให้คำสั่งต่างๆ ทำงานพร้อมกันได้โดยไม่ต้องรอกัน

ผลการตรวจสอบหลังแก้ไข
- ตรวจสอบ Processlist หลังดำเนินการเปลี่ยน Engine ของตาราง content_article_info แล้ว พบว่าไม่มี Query ค้างหรืออยู่ในสถานะรอ Table Lock
```