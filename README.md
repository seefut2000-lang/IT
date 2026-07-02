const express = require('express');
const sql = require('mssql');
const app = express();
app.use(express.json());

const dbConfig = {
    user: 'YOUR_DB_USER',
    password: 'YOUR_DB_PASSWORD',
    server: 'YOUR_SQL_SERVER_IP_OR_HOST',
    database: 'YOUR_DATABASE_NAME',
    options: { encrypt: false, trustServerCertificate: true }
};

// สร้าง Object ใน Memory ไว้สำหรับทำ ดีเลย์ 5 วินาที แยกตามรายสินค้า
// เพื่อป้องกันการสแกนซ้ำซ้อนในระยะเวลาอันสั้น
const scanCooldown = {};

app.post('/api/scan-gateway', async (req, res) => {
    try {
        const { gateway_id, rfid_tags } = req.body;
        const currentTime = new Date();
        const pool = await sql.connect(dbConfig);

        for (let tag of rfid_tags) {
            // ─── 1. ระบบดีเลย์ 5 วินาที (Anti-Bounce) ───
            if (scanCooldown[tag]) {
                const timeDiff = (currentTime - scanCooldown[tag]) / 1000; // แปลงเป็นวินาที
                if (timeDiff < 5) {
                    console.log(`⏳ [Skip] สินค้า ${tag} เพิ่งสแกนไปเมื่อ ${timeDiff} วินาทีก่อน (ติดดีเลย์ 5 วิ)`);
                    continue; // ข้ามการทำงานของสินค้าชิ้นนี้ไปชั่วคราว
                }
            }
            // อัปเดตเวลาสแกนล่าสุดของสินค้าชิ้นนี้
            scanCooldown[tag] = currentTime;

            // ─── 2. เช็กสถานะล่าสุดของสินค้าในฐานข้อมูล ───
            const checkStatus = await pool.request()
                .input('tagId', sql.VarChar, tag)
                .query(`
                    SELECT TOP 1 Status, CheckInTime 
                    FROM ProductLogs 
                    WHERE RFID_Tag = @tagId 
                    ORDER BY CheckInTime DESC
                `);

            const lastRecord = checkStatus.recordset[0];

            // ─── 3. LOGIC: สแกนครั้งแรก (เข้า) หรือ ไม่มีประวัติเก่า หรือ ประวัติล่าสุดคือออกไปแล้ว ───
            if (!lastRecord || lastRecord.Status === 'OUT') {
                // บันทึกสถานะ "เข้า (IN)" เรียลไทม์
                await pool.request()
                    .input('tagId', sql.VarChar, tag)
                    .input('gateway', sql.VarChar, gateway_id)
                    .input('now', sql.DateTime, currentTime)
                    .query(`
                        INSERT INTO ProductLogs (RFID_Tag, Status, CheckInTime, GatewayID)
                        VALUES (@tagId, 'IN', @now, @gateway)
                    `);
                console.log(`📥 [🟢 IN] สินค้า ${tag} สแกนเข้าคลังเรียลไทม์`);
            } 
            // ─── 4. LOGIC: สแกนครั้งที่สอง (ออก) ───
            else if (lastRecord.Status === 'IN') {
                const checkInTime = new Date(lastRecord.CheckInTime);
                // คำนวณเวลาที่ค้างอยู่ (มิลลิวินาที -> นาที)
                const durationMinutes = Math.round((currentTime - checkInTime) / (1000 * 60));

                // อัปเดตสถานะเป็น "ออก (OUT)" บันทึกเวลาออก และคำนวณเวลาค้างเสร็จสรรพ
                await pool.request()
                    .input('tagId', sql.VarChar, tag)
                    .input('checkIn', sql.DateTime, lastRecord.CheckInTime)
                    .input('now', sql.DateTime, currentTime)
                    .input('duration', sql.Int, durationMinutes)
                    .query(`
                        UPDATE ProductLogs 
                        SET Status = 'OUT', 
                            CheckOutTime = @now, 
                            DurationMinutes = @duration
                        WHERE RFID_Tag = @tagId AND CheckInTime = @checkIn
                    `);
                console.log(`📤 [🔴 OUT] สินค้า ${tag} สแกนออกคลัง! ค้างอยู่ในที่เดิมมาแล้ว ${durationMinutes} นาที`);
            }
        }

        res.status(200).json({ status: "success", message: "ประมวลผลสถานะสินค้าเรียลไทม์เรียบร้อย" });

    } catch (err) {
        console.error("Error: ", err);
        res.status(500).json({ status: "error", message: err.message });
    }
});

app.listen(3000, () => console.log('🚀 Smart Storage Server กำลังรันที่พอร์ต 3000...'));
