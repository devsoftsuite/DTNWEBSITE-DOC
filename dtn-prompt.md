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

```
ปัญหา
- เว็บไซต์ dtn.go.th แสดง Cloudflare Error 8 "The proxy failed to connect to the web server, due to TCP connection rejection (TCP Reset)" โดยเกิดขึ้นซ้ำหลายครั้งต่อวันตั้งแต่วันที่ 22-25 มีนาคม 2026

สาเหตุ
- ionCube PHP Loader เวอร์ชัน 10.4.5 เกิด Segmentation Fault (signal 11) บน Apache child process ทำให้ process นั้นหยุดทำงานกะทันหัน Cloudflare จึงไม่สามารถเชื่อมต่อมายัง server ได้และแสดง Error 8 แทน

ขั้นตอนการตรวจสอบ
1. ตรวจสอบ Apache error log ด้วยคำสั่ง grep "Mar 24" /var/log/httpd/error_log พบข้อความ "child pid XXXXX exit signal Segmentation fault (11)" เกิดขึ้น 3 ครั้งในวันที่ 24 มีนาคม และรวมทั้งสิ้น 12 ครั้งระหว่างวันที่ 22-25 มีนาคม
2. ตรวจสอบ PHP extension ด้วยคำสั่ง php -v พบว่าติดตั้ง ionCube PHP Loader + ionCube24 v10.4.5
3. ตรวจสอบไฟล์ PHP บนเว็บไซต์ พบว่ามีการใช้ ionCube encoded files จริง เช่น Slideshow.php, Contenttag.php, MenuData.php และอื่นๆ ใน applicationfront/views/helpers/
4. ตรวจสอบทรัพยากรระบบ พบว่า Memory และ CPU อยู่ในระดับปกติ (RAM ใช้ 1.5G จาก 15G, Load average 0.38) จึงตัดสาเหตุจาก resource หมดออกได้

วิธีการแก้ไข
1. Backup ionCube เดิมไว้ก่อนด้วยคำสั่ง
cp /usr/lib64/php/modules/ioncube_loader_lin_7.3.so /usr/lib64/php/modules/ioncube_loader_lin_7.3.so.bak.v10
2. Download ionCube Loader เวอร์ชันล่าสุดสำหรับ Linux x86-64
cd /tmp && curl -O https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz
tar -xzf ioncube_loaders_lin_x86-64.tar.gz
3. Copy ไฟล์ใหม่เข้าแทนที่ (ใช้ \cp เพื่อ bypass alias cp='cp -i' ของ CentOS)
\cp -f /tmp/ioncube/ioncube_loader_lin_7.3.so /usr/lib64/php/modules/ioncube_loader_lin_7.3.so
4. Restart Apache เพื่อโหลด ionCube เวอร์ชันใหม่
systemctl restart httpd

ผลการตรวจสอบหลังแก้ไข
- ionCube PHP Loader อัปเกรดจาก v10.4.5 เป็น v15.5.0 สำเร็จ (ตรวจสอบด้วย php -v)
- Apache restart สำเร็จและกลับมาทำงานปกติ (active running) ตั้งแต่เวลา 13:26:01 น.
- ต้องติดตาม error_log ต่อ เพื่อยืนยันว่า Segmentation Fault ไม่เกิดซ้ำอีก
```