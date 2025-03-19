# Crypto66
<a href="https://www.example.com">Crypto66</a>
const express = require("express");
const bodyParser = require("body-parser");
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");

const app = express();
app.use(bodyParser.json());

// In-memory storage สำหรับผู้ใช้ (สำหรับทดลองเท่านั้น)
const users = [];

// Secret Key สำหรับ JWT (ใน production ควรเก็บไว้ใน environment variable)
const JWT_SECRET = "your_secret_key";

// Route สำหรับสมัครสมาชิก
app.post("/register", async (req, res) => {
  const { username, password } = req.body;

  // ตรวจสอบว่ามีผู้ใช้ที่ใช้ username นี้แล้วหรือไม่
  if (users.find((user) => user.username === username)) {
    return res.status(400).json({ message: "Username นี้มีผู้ใช้แล้ว" });
  }

  try {
    // แฮชรหัสผ่าน
    const hashedPassword = await bcrypt.hash(password, 10);
    // บันทึกข้อมูลผู้ใช้
    users.push({ username, password: hashedPassword });
    res.json({ message: "สมัครสมาชิกสำเร็จ" });
  } catch (err) {
    res.status(500).json({ message: "เกิดข้อผิดพลาดในการสมัครสมาชิก" });
  }
});

// Route สำหรับเข้าสู่ระบบ
app.post("/login", async (req, res) => {
  const { username, password } = req.body;
  // ค้นหาผู้ใช้ตาม username
  const user = users.find((user) => user.username === username);
  if (!user) {
    return res.status(400).json({ message: "Username หรือ Password ไม่ถูกต้อง" });
  }

  try {
    // ตรวจสอบรหัสผ่าน
    const isValid = await bcrypt.compare(password, user.password);
    if (!isValid) {
      return res.status(400).json({ message: "Username หรือ Password ไม่ถูกต้อง" });
    }

    // สร้าง JWT token
    const token = jwt.sign({ username }, JWT_SECRET, { expiresIn: "1h" });
    res.json({ message: "เข้าสู่ระบบสำเร็จ", token });
  } catch (err) {
    res.status(500).json({ message: "เกิดข้อผิดพลาดในการเข้าสู่ระบบ" });
  }
});

// Route สำหรับเข้าถึงข้อมูลที่ต้องตรวจสอบตัวตน
app.get("/profile", (req, res) => {
  const authHeader = req.headers.authorization;
  if (!authHeader) {
    return res.status(401).json({ message: "ไม่มี token ให้ตรวจสอบ" });
  }

  // ดึง token จาก header (รูปแบบ "Bearer <token>")
  const token = authHeader.split(" ")[1];
  jwt.verify(token, JWT_SECRET, (err, decoded) => {
    if (err) {
      return res.status(401).json({ message: "Token ไม่ถูกต้องหรือหมดอายุ" });
    }
    res.json({ message: "ยินดีต้อนรับ", user: decoded.username });
  });
});

app.listen(3000, () => {
  console.log("Server running on port 3000");
});