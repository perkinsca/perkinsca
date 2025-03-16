// Full-stack app code (Node.js + Express + React + MongoDB)

// Backend (Node.js + Express + MongoDB)

const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();
const path = require("path");

const app = express();
app.use(cors());
app.use(express.json()); // to parse JSON in requests

// MongoDB connection
const mongoURI = process.env.MONGODB_URI || "mongodb://localhost:27017/todolist";
mongoose.connect(mongoURI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log("Connected to MongoDB"))
  .catch(err => console.log(err));

// Task Schema
const taskSchema = new mongoose.Schema({
  name: String,
  completed: { type: Boolean, default: false }
});

const Task = mongoose.model("Task", taskSchema);

// Routes
app.get("/tasks", async (req, res) => {
  try {
    const tasks = await Task.find();
    res.json(tasks);
  } catch (err) {
    res.status(400).json({ message: "Error fetching tasks" });
  }
});

app.post("/tasks", async (req, res) => {
  try {
    const { name } = req.body;
    const newTask = new Task({ name });
    await newTask.save();
    res.status(201).json(newTask);
  } catch (err) {
    res.status(400).json({ message: "Error creating task" });
  }
});

app.put("/tasks/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const task = await Task.findByIdAndUpdate(id, { completed: true }, { new: true });
    res.json(task);
  } catch (err) {
    res.status(400).json({ message: "Error updating task" });
  }
});

// Serve frontend build files in production
if (process.env.NODE_ENV === "production") {
  app.use(express.static(path.join(__dirname, "client/build")));

  app.get("*", (req, res) => {
    res.sendFile(path.join(__dirname, "client/build", "index.html"));
  });
}

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));


// Frontend (React)

// You can put the following code in the React App (client) folder and run it using `npm start`.

// App.js (React main component)

import React, { useState, useEffect } from "react";
import axios from "axios";

function App() {
  const [tasks, setTasks] = useState([]);
  const [taskName, setTaskName] = useState("");

  // Fetch tasks on component mount
  useEffect(() => {
    axios.get("/tasks")
      .then((response) => {
        setTasks(response.data);
      })
      .catch((error) => {
        console.error("There was an error fetching the tasks:", error);
      });
  }, []);

  // Handle task submission
  const handleSubmit = (e) => {
    e.preventDefault();
    if (taskName) {
      axios.post("/tasks", { name: taskName })
        .then((response) => {
          setTasks([...tasks, response.data]);
          setTaskName(""); // Clear input field
        })
        .catch((error) => {
          console.error("There was an error creating the task:", error);
        });
    }
  };

  // Handle task completion
  const handleComplete = (taskId) => {
    axios.put(`/tasks/${taskId}`)
      .then((response) => {
        const updatedTasks = tasks.map((task) =>
          task._id === taskId ? response.data : task
        );
        setTasks(updatedTasks);
      })
      .catch((error) => {
        console.error("There was an error updating the task:", error);
      });
  };

  return (
    <div className="App">
      <h1>Task List</h1>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={taskName}
          onChange={(e) => setTaskName(e.target.value)}
          placeholder="Enter task"
        />
        <button type="submit">Add Task</button>
      </form>
      <ul>
        {tasks.map((task) => (
          <li key={task._id}>
            <span style={{ textDecoration: task.completed ? "line-through" : "none" }}>
              {task.name}
            </span>
            {!task.completed && (
              <button onClick={() => handleComplete(task._id)}>Complete</button>
            )}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
