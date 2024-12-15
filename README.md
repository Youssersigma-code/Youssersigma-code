mkdir server
cd server
npm init -y
npm install express mongoose dotenv cors bcrypt jsonwebtoken node-schedule
/server
  ├── /controllers       # Controller logic
  ├── /middleware        # Middleware for authentication
  ├── /models            # Database schemas
  ├── /routes            # API routes
  ├── .env               # Environment variables
  ├── server.js          # Entry point for backend
  └── package.json       # Backend dependencies
MONGO_URI=mongodb+srv://username:password@cluster0.mongodb.net/todo_app?retryWrites=true&w=majority
JWT_SECRET=your_jwt_secret
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const authRoutes = require("./routes/authRoutes");
const taskRoutes = require("./routes/taskRoutes");
const analyticsRoutes = require("./routes/analyticsRoutes");

const app = express();
app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log("Connected to MongoDB"))
  .catch((err) => console.error(err));

app.use("/auth", authRoutes);
app.use("/tasks", taskRoutes);
app.use("/analytics", analyticsRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
// File: models/User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

module.exports = mongoose.model("User", userSchema);
// File: models/Task.js
const mongoose = require("mongoose");

const taskSchema = new mongoose.Schema({
  title: { type: String, required: true },
  category: { type: String },
  completed: { type: Boolean, default: false },
  deadline: { type: Date },
  userId: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
});

module.exports = mongoose.model("Task", taskSchema);
// File: routes/authRoutes.js
const express = require("express");
const { signup, login } = require("../controllers/authController");
const router = express.Router();

router.post("/signup", signup);
router.post("/login", login);

module.exports = router;
// File: middleware/authMiddleware.js
const jwt = require("jsonwebtoken");

const authMiddleware = (req, res, next) => {
  const token = req.headers.token;
  if (!token) return res.status(401).send("Access Denied: No Token Provided");

  try {
    const verified = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = verified.userId;
    next();
  } catch (error) {
    res.status(400).send("Invalid Token");
  }
};

module.exports = authMiddleware;
npx create-react-app client
cd client
npm install @mui/material @emotion/react @emotion/styled axios react-router-dom react-toastify
/client
  ├── /public
  ├── /src
      ├── /components       # Reusable components
      ├── /pages            # Page-level components
      ├── /services         # Axios calls to backend
      ├── App.js            # Main React component
      ├── index.js          # React entry point
  ├── .env                 # Frontend environment variables
  └── package.json         # Frontend dependencies
REACT_APP_API_URL=http://localhost:5000
import React, { useState, useEffect } from "react";
import { Tabs, Tab, Typography, List, ListItem, Checkbox } from "@mui/material";
import axios from "axios";

function Dashboard() {
  const [tasks, setTasks] = useState([]);
  const [tab, setTab] = useState(0);

  const categories = ["All", "Work", "Personal", "Miscellaneous"];

  const fetchTasks = async () => {
    const category = categories[tab] !== "All" ? categories[tab] : "";
    const { data } = await axios.get(`${process.env.REACT_APP_API_URL}/tasks?category=${category}`, {
      headers: { token: localStorage.getItem("token") },
    });
    setTasks(data);
  };

  useEffect(() => {
    fetchTasks();
  }, [tab]);

  return (
    <div>
      <Typography variant="h4">Dashboard</Typography>
      <Tabs value={tab} onChange={(e, newValue) => setTab(newValue)}>
        {categories.map((cat, idx) => (
          <Tab label={cat} key={idx} />
        ))}
      </Tabs>
      <List>
        {tasks.map((task) => (
          <ListItem key={task._id}>
            <Checkbox checked={task.completed} />
            {task.title}
          </ListItem>
        ))}
      </List>
    </div>
  );
}

export default Dashboard;
heroku create
git push heroku main
npm install -g vercel
vercel

