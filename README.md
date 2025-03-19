# Crypto66
<a href="https://www.example.com">Crypto66</a>
npm install express mongoose bcryptjs jsonwebtoken dotenv
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_secret_key
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

// สร้าง Schema สำหรับ User
const UserSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

// ใช้ bcrypt ในการเข้ารหัสรหัสผ่านก่อนบันทึกข้อมูล
UserSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// เพิ่มฟังก์ชันในการตรวจสอบรหัสผ่าน
UserSchema.methods.matchPassword = async function(password) {
  return await bcrypt.compare(password, this.password);
};

module.exports = mongoose.model('User', UserSchema);
const User = require('../models/User');
const jwt = require('jsonwebtoken');

// ฟังก์ชันสำหรับสมัครสมาชิก
const registerUser = async (req, res) => {
  const { username, email, password } = req.body;

  try {
    const userExists = await User.findOne({ email });

    if (userExists) {
      return res.status(400).json({ message: 'User already exists' });
    }

    const newUser = new User({ username, email, password });
    await newUser.save();

    const token = jwt.sign({ id: newUser._id }, process.env.JWT_SECRET, { expiresIn: '1h' });

    res.status(201).json({ token, username: newUser.username });
  } catch (err) {
    res.status(500).json({ message: 'Server error' });
  }
};

module.exports = { registerUser };
const loginUser = async (req, res) => {
  const { email, password } = req.body;

  try {
    const user = await User.findOne({ email });

    if (!user) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }

    const isMatch = await user.matchPassword(password);
    if (!isMatch) {
      return res.status(400).json({ message: 'Invalid credentials' });
    }

    const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });

    res.status(200).json({ token, username: user.username });
  } catch (err) {
    res.status(500).json({ message: 'Server error' });
  }
};

module.exports = { loginUser };
const express = require('express');
const router = express.Router();
const { registerUser, loginUser } = require('../controllers/authController');

// API สำหรับสมัครสมาชิก
router.post('/register', registerUser);

// API สำหรับล็อกอิน
router.post('/login', loginUser);

module.exports = router;
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const authRoutes = require('./routes/authRoutes');

dotenv.config();

const app = express();

// เชื่อมต่อกับ MongoDB
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.log(err));

// ใช้ JSON parser
app.use(express.json());

// ตั้งค่า routes
app.use('/api/auth', authRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
