<template>
    <div class="container">
      <!-- 添加学生模块 -->
      <div class="add-section">
        <input 
          v-model.trim="newName"
          placeholder="输入姓名"
          @keyup.enter="handleAdd"
        >
        <input
          v-model.number="newScore"
          type="number"
          min="0"
          max="100"
          placeholder="输入分数"
          @keyup.enter="handleAdd"
        >
        <button @click="handleAdd">添加学生</button>
      </div>
  
      <!-- 过滤控制 -->
      <div class="filter-controls">
        <input 
          v-model.trim="searchKeyword" 
          placeholder="搜索姓氏（如：张）"
        >
        <label>
          <input type="checkbox" v-model="showPassed"> 仅显示及格（≥60）
        </label>
        <div class="score-range">
          分数范围：
          <input v-model.number="minScore" placeholder="最低分">
          -
          <input v-model.number="maxScore" placeholder="最高分">
        </div>
      </div>
  
      <!-- 学生列表 -->
      <div class="student-list">
        <table v-if="filteredStudents.length">
          <thead>
            <tr>
              <th>姓名</th>
              <th>分数</th>
              <th>操作</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="student in filteredStudents" 
              :key="student.id"
            >
              <td>{{ student.name }}</td>
              <td 
                :class="{ 'text-red': student.score < 60 }"
                @dblclick="startEdit(student)"
              >
                <template v-if="editingId === student.id">
                  <input
                    type="number"
                    v-model.number="editingScore"
                    @blur="saveEdit(student)"
                    @keyup.enter="saveEdit(student)"
                    min="0"
                    max="100"
                    autofocus
                  >
                </template>
                <span v-else>{{ student.score }}</span>
              </td>
              <td>
                <button 
                  @click="deleteStudent(student.id)"
                  class="delete-btn"
                >
                  删除
                </button>
              </td>
            </tr>
          </tbody>
        </table>
        <div v-else class="empty-tip">
          {{ searchKeyword ? '没有找到匹配的学生' : '暂无学生数据' }}
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed } from 'vue'
  
  // 响应式数据
  const students = ref([])
  const newName = ref('')
  const newScore = ref(null)
  const searchKeyword = ref('')
  const showPassed = ref(false)
  const minScore = ref(null)
  const maxScore = ref(null)
  const editingId = ref(null)
  const editingScore = ref(null)
  
  // 添加学生
  const handleAdd = () => {
    if (!newName.value.trim()) return alert('请输入姓名')
    const score = Number(newScore.value)
    if (isNaN(score) || score < 0 || score > 100) {
      return alert('请输入0-100之间的有效分数')
    }
  
    students.value.unshift({
      id: Date.now(),
      name: newName.value.trim(),
      score: score
    })
    
    newName.value = ''
    newScore.value = null
  }
  
  // 删除学生
  const deleteStudent = (id) => {
    students.value = students.value.filter(s => s.id !== id)
  }
  
  // 编辑功能
  const startEdit = (student) => {
    editingId.value = student.id
    editingScore.value = student.score
  }
  
  const saveEdit = (student) => {
    const newScore = Number(editingScore.value)
    if (!isNaN(newScore) && newScore >= 0 && newScore <= 100) {
      student.score = newScore
    }
    editingId.value = null
  }
  
  // 计算属性处理过滤
  const filteredStudents = computed(() => {
    return students.value.filter(student => {
      const nameMatch = searchKeyword.value 
        ? student.name.startsWith(searchKeyword.value)
        : true
      
      const passMatch = showPassed.value 
        ? student.score >= 60 
        : true
      
      const min = minScore.value ?? 0
      const max = maxScore.value ?? 100
      const rangeMatch = student.score >= min && student.score <= max
      
      return nameMatch && passMatch && rangeMatch
    })
  })
  </script>
  
  <style scoped>
  .container {
    max-width: 800px;
    margin: 20px auto;
    padding: 20px;
  }
  
  .add-section {
    margin-bottom: 20px;
  }
  
  input {
    padding: 8px;
    margin-right: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  button {
    padding: 8px 16px;
    background: #409eff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  .delete-btn {
    background: #f56c6c;
  }
  
  table {
    width: 100%;
    margin-top: 20px;
    border-collapse: collapse;
  }
  
  th, td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
  }
  
  .text-red {
    color: #f56c6c;
  }
  
  .empty-tip {
    color: #999;
    margin-top: 20px;
    text-align: center;
  }
  
  .score-range input {
    width: 80px;
    margin: 0 5px;
  }
  </style>