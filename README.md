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
