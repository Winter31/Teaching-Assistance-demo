<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import * as authApi from '@/hooks/api/auth'
import Header from '@/components/Header.vue'
import CourseManage from '@/components/CourseManage.vue'
import FunctionSelect from '@/components/FunctionSelect.vue'
import CourseInfo from '@/components/CourseInfo.vue'
import CourseDescription from '@/components/CourseDescription.vue'
import CourseOutline from '@/components/CourseOutline.vue'
import TeachingLecture from '@/components/TeachingLecture.vue'

const router = useRouter()
const username = ref('')
const isLoggedIn = ref(false)
const showFunctionSelect = ref(false)
const showCourseInfo = ref(false)
const showCourseDescription = ref(false)
const showCourseOutline = ref(false)
const showTeachingLecture = ref(false)
const selectedCourseTitle = ref('')
const selectedCourseId = ref<number | undefined>(undefined)
const selectedModuleId = ref('')

// 从本地存储恢复状态
const restoreState = () => {
  const storedShowFunctionSelect = localStorage.getItem('showFunctionSelect')
  const storedShowCourseInfo = localStorage.getItem('showCourseInfo')
  const storedShowCourseDescription = localStorage.getItem('showCourseDescription')
  const storedShowCourseOutline = localStorage.getItem('showCourseOutline')
  const storedShowTeachingLecture = localStorage.getItem('showTeachingLecture')
  const storedCourseTitle = localStorage.getItem('selectedCourseTitle')
  const storedCourseId = localStorage.getItem('selectedCourseId')
  const storedModuleId = localStorage.getItem('selectedModuleId')
  
  // 检查上次页面导航来源
  const fromLogin = sessionStorage.getItem('fromLogin') === 'true'
  
  // 如果是从登录页面来的，不恢复视图状态，直接展示课程管理
  if (fromLogin) {
    // 清除标记
    sessionStorage.removeItem('fromLogin')
    return
  }
  
  // 否则按照正常逻辑恢复状态
  if (storedShowFunctionSelect === 'true' && storedCourseTitle) {
    showFunctionSelect.value = true
    selectedCourseTitle.value = storedCourseTitle
    if (storedCourseId) {
      selectedCourseId.value = parseInt(storedCourseId)
    }
  }
  
  if (storedShowCourseInfo === 'true') {
    showCourseInfo.value = true
  }

  if (storedShowCourseDescription === 'true' && storedModuleId) {
    showCourseDescription.value = true
    selectedModuleId.value = storedModuleId
  }
  
  if (storedShowCourseOutline === 'true') {
    showCourseOutline.value = true
  }
  
  if (storedShowTeachingLecture === 'true') {
    showTeachingLecture.value = true
  }
}

// 保存状态到本地存储
watch([showFunctionSelect, showCourseInfo, showCourseDescription, showCourseOutline, showTeachingLecture, selectedCourseTitle, selectedCourseId, selectedModuleId], () => {
  localStorage.setItem('showFunctionSelect', showFunctionSelect.value.toString())
  localStorage.setItem('showCourseInfo', showCourseInfo.value.toString())
  localStorage.setItem('showCourseDescription', showCourseDescription.value.toString())
  localStorage.setItem('showCourseOutline', showCourseOutline.value.toString())
  localStorage.setItem('showTeachingLecture', showTeachingLecture.value.toString())
  localStorage.setItem('selectedCourseTitle', selectedCourseTitle.value)
  if (selectedCourseId.value !== undefined) {
    localStorage.setItem('selectedCourseId', selectedCourseId.value.toString())
  }
  localStorage.setItem('selectedModuleId', selectedModuleId.value)
})

onMounted(() => {
  // 检查是否已登录
  checkLoginStatus()
  
  // 恢复状态
  restoreState()

  // 监听登录状态变化
  window.addEventListener('login-state-changed', checkLoginStatus)

  return () => {
    window.removeEventListener('login-state-changed', checkLoginStatus)
  }
})

// 检查登录状态
const checkLoginStatus = () => {
  const token = sessionStorage.getItem('access_token') || localStorage.getItem('access_token')
  isLoggedIn.value = !!token
  const isAdmin = localStorage.getItem('isAdmin') === 'true'
  
  console.log('[Home] 检查登录状态:', { isLoggedIn: isLoggedIn.value, isAdmin })
  
  if (!isLoggedIn.value) {
    // 如果未登录，跳转到登录页
    console.log('[Home] 未登录，跳转到登录页')
    router.push('/auth')
  } else if (isAdmin) {
    // 如果是管理员，跳转到管理页
    console.log('[Home] 管理员用户，跳转到管理页')
    router.push('/admin')
  } else {
    // 从本地存储获取用户名
    const storedUsername = localStorage.getItem('username') || sessionStorage.getItem('username')
    username.value = storedUsername || '教学用户'
    
    // 确保userId被正确存储
    const userId = sessionStorage.getItem('userId') || localStorage.getItem('userId')
    
    if (!userId) {
      // 如果没有找到userId，尝试从其他地方获取或使用默认值
      const teacherId = sessionStorage.getItem('teacherId') || localStorage.getItem('teacherId')
      
      if (teacherId) {
        // 如果有teacherId，将其同时保存为userId
        console.log('从teacherId获取并保存为userId:', teacherId)
        sessionStorage.setItem('userId', teacherId)
        localStorage.setItem('userId', teacherId)
      } else {
        // 如果也没有teacherId，使用默认值
        console.warn('未找到用户ID，设置默认值')
        const defaultId = '2' // 默认教师ID，根据实际情况调整
        sessionStorage.setItem('userId', defaultId)
        localStorage.setItem('userId', defaultId)
      }
    }
  }
}

// 打开课程功能选择界面
const openFunctionSelect = (course: { name: string, id: number }) => {
  selectedCourseTitle.value = course.name
  selectedCourseId.value = course.id
  showFunctionSelect.value = true
  showCourseInfo.value = false
}

// 返回课程管理界面
const backToCourseManage = () => {
  showFunctionSelect.value = false
  showCourseInfo.value = false
}

// 显示课程信息
const showCourseInfoPanel = () => {
  showCourseInfo.value = true
  showCourseDescription.value = false
}

// 隐藏课程信息
const hideCourseInfoPanel = () => {
  showCourseInfo.value = false
}

// 显示课程模块内容
const showModule = (moduleId: string) => {
  selectedModuleId.value = moduleId
  if (moduleId === 'basic') {
    showCourseDescription.value = true
    showCourseInfo.value = false
    showCourseOutline.value = false
    showTeachingLecture.value = false
  } else if (moduleId === 'outline') {
    showCourseOutline.value = true
    showCourseDescription.value = false
    showCourseInfo.value = false
    showTeachingLecture.value = false
  } else if (moduleId === 'lecture') {
    showTeachingLecture.value = true
    showCourseOutline.value = false
    showCourseDescription.value = false
    showCourseInfo.value = false
  }
  // 其他模块的处理可以在这里添加
}

// 返回功能选择
const backToFunctionSelect = () => {
  showCourseDescription.value = false
  showCourseOutline.value = false
  showTeachingLecture.value = false
}

</script>

<template>
  <div class="home-container">
    <Header />
    
    <div v-if="isLoggedIn" class="content-area">
      <!-- 根据当前状态显示课程管理、功能选择或课程信息 -->
      <CourseManage v-if="!showFunctionSelect && !showCourseInfo && !showCourseDescription && !showCourseOutline && !showTeachingLecture" @course-selected="openFunctionSelect" />
      <FunctionSelect 
        v-else-if="showFunctionSelect && !showCourseInfo && !showCourseDescription && !showCourseOutline && !showTeachingLecture" 
        :courseTitle="selectedCourseTitle" 
        :courseId="selectedCourseId"
        @back="backToCourseManage"
        @show-course-info="showCourseInfoPanel"
        @show-module="showModule"
      />
      <CourseInfo 
        v-else-if="showCourseInfo" 
        @back="hideCourseInfoPanel" 
      />
      <CourseDescription
        v-else-if="showCourseDescription"
        :courseId="selectedCourseId"
        @back="backToFunctionSelect"
      />
      <CourseOutline
        v-else-if="showCourseOutline"
        :courseId="selectedCourseId"
        @back="backToFunctionSelect"
      />
      <TeachingLecture
        v-else-if="showTeachingLecture"
        :courseId="selectedCourseId"
        @back="backToFunctionSelect"
      />
    </div>
  </div>
</template>

<style scoped>
.home-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  position: relative;
}

.content-area {
  flex: 1;
  position: relative;
  margin-top: 50px; /* 添加与header高度相等的上边距 */
}
</style>
