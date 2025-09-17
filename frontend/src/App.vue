<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { ElMessage } from "element-plus";

const todos = ref([]);
const newTodo = ref("");
const editingId = ref(null);
const editingText = ref("");

const loadTodos = async () => {
  const res = await axios.get("http://127.0.0.1:8000/api/todos/");
  todos.value = res.data;
};

// ➕ 添加
const addTodo = async () => {
  if (!newTodo.value) return;
  await axios.post("http://127.0.0.1:8000/api/todos/", {
    title: newTodo.value,
    completed: false,
  });
  ElMessage.success("Task added!");
  newTodo.value = "";
  loadTodos();
};

// ✅ 切换完成状态
const toggleTodo = async (todo) => {
  await axios.put(`http://127.0.0.1:8000/api/todos/${todo.id}/`, {
    ...todo,
    completed: !todo.completed,
  });
  ElMessage.success("Task updated!");
  loadTodos();
};

// 📝 开始编辑
const startEdit = (todo) => {
  editingId.value = todo.id;
  editingText.value = todo.title;
};

// 💾 保存修改
const saveEdit = async (todo) => {
  await axios.put(`http://127.0.0.1:8000/api/todos/${todo.id}/`, {
    ...todo,
    title: editingText.value,
  });
  ElMessage.success("Task updated!");
  editingId.value = null;
  editingText.value = "";
  loadTodos();
};

// ❌ 删除
const deleteTodo = async (todo) => {
  await axios.delete(`http://127.0.0.1:8000/api/todos/${todo.id}/`);
  ElMessage.warning("Task deleted!");
  loadTodos();
};

onMounted(loadTodos);
</script>

<template>
  <div style="padding: 20px;">
    <h1 style="margin-bottom:20px;">✅ Todo List</h1>

    <!-- 输入框 + 按钮 -->
    <el-input
      v-model="newTodo"
      placeholder="Enter a task"
      style="width:300px; margin-right:10px;"
      clearable
    />
    <el-button type="primary" @click="addTodo">Add</el-button>

    <!-- 表格 -->
    <el-table :data="todos" style="width: 600px; margin-top:20px;" border>
      <el-table-column label="Done" width="80">
        <template #default="scope">
          <el-checkbox
            v-model="scope.row.completed"
            @change="toggleTodo(scope.row)"
          />
        </template>
      </el-table-column>

      <el-table-column label="Task">
        <template #default="scope">
          <div v-if="editingId !== scope.row.id">
            <span :style="{ textDecoration: scope.row.completed ? 'line-through' : 'none' }">
              {{ scope.row.title }}
            </span>
          </div>
          <div v-else>
            <el-input v-model="editingText" style="width:300px;" />
          </div>
        </template>
      </el-table-column>

      <el-table-column label="Actions" width="200">
        <template #default="scope">
          <el-button
            v-if="editingId !== scope.row.id"
            size="small"
            @click="startEdit(scope.row)"
          >Edit</el-button>
          <el-button
            v-else
            size="small"
            type="success"
            @click="saveEdit(scope.row)"
          >Save</el-button>
          <el-button
            size="small"
            type="danger"
            @click="deleteTodo(scope.row)"
          >Delete</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
