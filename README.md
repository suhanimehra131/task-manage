# task-manage
html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Task Manager</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      padding: 0;
      background-color: #f4f4f9;
    }
    h1 {
      color: #333;
    }
    form {
      margin-bottom: 20px;
    }
    input, textarea, button {
      margin: 5px 0;
      padding: 10px;
      width: 100%;
      box-sizing: border-box;
    }
    button {
      background-color: #28a745;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background-color: #218838;
    }
    ul {
      list-style-type: none;
      padding: 0;
    }
    li {
      background: #fff;
      margin: 10px 0;
      padding: 15px;
      border-radius: 5px;
      box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .completed {
      text-decoration: line-through;
      color: #888;
    }
    .error {
      color: red;
    }
    .loading {
      color: #007bff;
    }
    .task-actions {
      display: flex;
      gap: 10px;
    }
    .edit-form {
      display: flex;
      gap: 10px;
    }
    .edit-form input {
      flex: 1;
    }
  </style>
</head>
<body>
  <div id="app">
    <h1>Task Manager</h1>
    <form id="task-form">
      <input type="text" id="title" placeholder="Title" required />
      <textarea id="description" placeholder="Description"></textarea>
      <input type="date" id="dueDate" />
      <button type="submit">Add Task</button>
    </form>
    <p class="loading" id="loading">Loading...</p>
    <p class="error" id="error"></p>
    <ul id="task-list"></ul>
  </div>

  <script>
    const taskForm = document.getElementById('task-form');
    const taskList = document.getElementById('task-list');
    const loadingMessage = document.getElementById('loading');
    const errorMessage = document.getElementById('error');

    let tasks = [];
    let isLoading = false;

    // Simulate server actions with a mock API
    const mockServerActions = {
      createTask: async (task) => {
        return new Promise((resolve) => setTimeout(() => resolve({ ...task, id: Date.now().toString() }), 500));
      },
      getTasks: async () => {
        return new Promise((resolve) => setTimeout(() => resolve(tasks), 500));
      },
      updateTask: async (id, updates) => {
        return new Promise((resolve) => {
          tasks = tasks.map(task => task.id === id ? { ...task, ...updates } : task);
          setTimeout(() => resolve(), 500);
        });
      },
      deleteTask: async (id) => {
        return new Promise((resolve) => {
          tasks = tasks.filter(task => task.id !== id);
          setTimeout(() => resolve(), 500);
        });
      }
    };

    const fetchTasks = async () => {
      try {
        setLoading(true);
        const data = await mockServerActions.getTasks();
        tasks = data;
        renderTasks();
      } catch (err) {
        setError('Failed to load tasks.');
      } finally {
        setLoading(false);
      }
    };

    const addTask = async (task) => {
      try {
        setLoading(true);
        const newTask = await mockServerActions.createTask(task);
        tasks.push(newTask);
        renderTasks();
      } catch (err) {
        setError('Failed to add task.');
      } finally {
        setLoading(false);
      }
    };

    const updateTask = async (id, updates) => {
      try {
        setLoading(true);
        await mockServerActions.updateTask(id, updates);
        renderTasks();
      } catch (err) {
        setError('Failed to update task.');
      } finally {
        setLoading(false);
      }
    };

    const deleteTask = async (id) => {
      try {
        setLoading(true);
        await mockServerActions.deleteTask(id);
        renderTasks();
      } catch (err) {
        setError('Failed to delete task.');
      } finally {
        setLoading(false);
      }
    };

    const renderTasks = () => {
      taskList.innerHTML = '';
      tasks.forEach(task => {
        const li = document.createElement('li');
        if (task.editing) {
          li.innerHTML = `
            <form class="edit-form" data-id="${task.id}">
              <input type="text" value="${task.title}" />
              <button type="submit">Save</button>
            </form>
          `;
          const editForm = li.querySelector('.edit-form');
          editForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const newTitle = editForm.querySelector('input').value.trim();
            if (newTitle) {
              await updateTask(task.id, { title: newTitle, editing: false });
            }
          });
        } else {
          li.innerHTML = `
            <div>
              <h3>${task.title}</h3>
              <p>${task.description || 'No description'}</p>
              <p>Due: ${task.dueDate || 'No due date'}</p>
            </div>
            <div class="task-actions">
              <label>
                Completed:
                <input type="checkbox" ${task.completed ? 'checked' : ''} />
              </label>
              <button class="edit-btn">Edit</button>
              <button class="delete-btn">Delete</button>
            </div>
          `;
          li.querySelector('input[type="checkbox"]').addEventListener('change', () => {
            updateTask(task.id, { completed: !task.completed });
          });
          li.querySelector('.edit-btn').addEventListener('click', () => {
            updateTask(task.id, { editing: true });
          });
          li.querySelector('.delete-btn').addEventListener('click', () => {
            deleteTask(task.id);
          });
        }
        if (task.completed) li.classList.add('completed');
        taskList.appendChild(li);
      });
    };

    const setLoading = (state) => {
      isLoading = state;
      loadingMessage.style.display = state ? 'block' : 'none';
    };

    const setError = (message) => {
      errorMessage.textContent = message;
      errorMessage.style.display = message ? 'block' : 'none';
    };

    taskForm.addEventListener('submit', async (e) => {
      e.preventDefault();
      const title = document.getElementById('title').value.trim();
      const description = document.getElementById('description').value.trim();
      const dueDate = document.getElementById('dueDate').value;
      if (title) {
        await addTask({ title, description, dueDate, completed: false });
        taskForm.reset();
      }
    });

    // Initial fetch
    fetchTasks();
  </script>
</body>
</html>
