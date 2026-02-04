# P1
backend/
│
├── server.js
├── config/db.js
├── models/User.js
├── models/Task.js
├── routes/authRoutes.js
├── routes/taskRoutes.js
├── middleware/authMiddleware.js
├── controllers/authController.js
├── controllers/taskController.js
├── swagger.js
└── .env
const express = require("express");
const cors = require("cors");
const connectDB = require("./config/db");
const authRoutes = require("./routes/authRoutes");
const taskRoutes = require("./routes/taskRoutes");
const swaggerUi = require("swagger-ui-express");
const swaggerSpec = require("./swagger");

require("dotenv").config();

connectDB();
const app = express();

app.use(cors());
app.use(express.json());

app.use("/api/v1/auth", authRoutes);
app.use("/api/v1/tasks", taskRoutes);
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerSpec));

app.listen(5000, () => console.log("Server running"));
const mongoose = require("mongoose");

module.exports = async () => {
  await mongoose.connect(process.env.MONGO_URI);
  console.log("MongoDB connected");
};
const mongoose = require("mongoose");

module.exports = mongoose.model("User", {
  name: String,
  email: String,
  password: String,
  role: { type: String, default: "user" }
});
const mongoose = require("mongoose");

module.exports = mongoose.model("Task", {
  title: String,
  description: String,
  user: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
});
/*task.js*/
const jwt = require("jsonwebtoken");

exports.auth = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).send("No token");

  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).send("Invalid token");
  }
};

exports.admin = (req, res, next) => {
  if (req.user.role !== "admin")
    return res.status(403).send("Admin only");
  next();
};
const User = require("../models/User");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

exports.register = async (req, res) => {
  const hash = await bcrypt.hash(req.body.password, 10);
  const user = await User.create({ ...req.body, password: hash });
  res.json(user);
};

exports.login = async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (!user) return res.status(400).send("User not found");

  const ok = await bcrypt.compare(req.body.password, user.password);
  if (!ok) return res.status(400).send("Wrong password");

  const token = jwt.sign(
    { id: user._id, role: user.role },
    process.env.JWT_SECRET
  );

  res.json({ token });
};
const Task = require("../models/Task");

exports.create = async (req, res) => {
  const task = await Task.create({
    ...req.body,
    user: req.user.id
  });
  res.json(task);
};

exports.getAll = async (req, res) => {
  const tasks = await Task.find({ user: req.user.id });
  res.json(tasks);
};

exports.update = async (req, res) => {
  const task = await Task.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true }
  );
  res.json(task);
};

exports.remove = async (req, res) => {
  await Task.findByIdAndDelete(req.params.id);
  res.send("Deleted");
};
const router = require("express").Router();
const { register, login } = require("../controllers/authController");

router.post("/register", register);
router.post("/login", login);

module.exports = router;
const router = require("express").Router();
const { auth } = require("../middleware/authMiddleware");
const ctrl = require("../controllers/taskController");

router.post("/", auth, ctrl.create);
router.get("/", auth, ctrl.getAll);
router.put("/:id", auth, ctrl.update);
router.delete("/:id", auth, ctrl.remove);

module.exports = router;
MONGO_URI=mongodb://localhost:27017/test
JWT_SECRET=mysecret
Data base url
import Login from "./Login";
function App() {
  return <Login />;
}
export default App;
import axios from "axios";
import { useState } from "react";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const login = async () => {
    const res = await axios.post(
      "http://localhost:5000/api/v1/auth/login",
      { email, password }
    );
    localStorage.setItem("token", res.data.token);
    alert("Login success");
  };

  return (
    <div>
      <input onChange={e => setEmail(e.target.value)} placeholder="email"/>
      <input type="password" onChange={e => setPassword(e.target.value)} placeholder="password"/>
      <button onClick={login}>Login</button>
    </div>
  );
}
import axios from "axios";
import { useState } from "react";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const login = async () => {
    const res = await axios.post(
      "http://localhost:5000/api/v1/auth/login",
      { email, password }
    );
    localStorage.setItem("token", res.data.token);
    alert("Login success");
  };

  return (
    <div>
      <input onChange={e => setEmail(e.target.value)} placeholder="email"/>
      <input type="password" onChange={e => setPassword(e.target.value)} placeholder="password"/>
      <button onClick={login}>Login</button>
    </div>
  );
}
