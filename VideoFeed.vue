<template>
  <div class="video-feed-container" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd">
    <!-- 主屏视频区域 -->
    <transition-group name="video-slide" tag="div" class="video-stack">
      <div 
        v-for="(job, index) in visibleVideos" 
        :key="job.id"
        class="video-card"
        :class="{ 'active': index === currentIndex }"
      >
        <video
          :ref="`video-${index}`"
          :src="job.video_url"
          class="video-element"
          autoplay
          muted
          playsinline
          @ended="nextVideo"
        />

        <!-- 右侧交互按钮 -->
        <div class="interaction-buttons">
          <button class="icon-btn" @click="toggleFavorite(job.id)">
            <span class="icon">❤️</span>
            <span class="count">{{ job.likes }}</span>
          </button>
          <button class="icon-btn" @click="shareVideo(job.id)">
            <span class="icon">📤</span>
          </button>
          <div class="company-avatar" @click="goToCompany(job.company_id)">
            <img :src="job.company_logo" :alt="job.company_name" />
          </div>
        </div>

        <!-- 底部信息浮层 -->
        <div class="job-info-overlay">
          <h3 class="job-title">{{ job.title }}</h3>
          <p class="job-meta">
            💰 {{ job.salary }} | 📍 {{ job.city }} | {{ job.company_name }}
          </p>
          <div class="benefit-tags">
            <span v-for="tag in job.benefits" :key="tag" class="tag">{{ tag }}</span>
          </div>
        </div>

        <!-- 底部发送按钮 -->
        <button class="send-profile-btn" @click="openLeftPanel">
          📋 发送人才成型模块
        </button>
      </div>
    </transition-group>

    <!-- 左滑面板 -->
    <LeftPanel 
      v-if="showLeftPanel" 
      :job="currentJob"
      :userProfile="userProfile"
      @close="showLeftPanel = false"
      @submit="submitApplication"
    />

    <!-- 右滑菜单 -->
    <RightPanel 
      v-if="showRightPanel" 
      :filters="filters"
      @filter-change="applyFilters"
      @close="showRightPanel = false"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useUserStore } from '@/stores/userStore'
import { useVideoStore } from '@/stores/videoStore'
import LeftPanel from './LeftPanel.vue'
import RightPanel from './RightPanel.vue'
import { fetchJobs, submitApplication as apiSubmitApplication } from '@/api/client'

interface Job {
  id: string
  title: string
  video_url: string
  company_id: string
  company_name: string
  company_logo: string
  salary: string
  city: string
  benefits: string[]
  requirements: Record<string, any>
  address: string
  contact: string
  likes: number
}

const userStore = useUserStore()
const videoStore = useVideoStore()

// 状态
const jobs = ref<Job[]>([])
const currentIndex = ref(0)
const showLeftPanel = ref(false)
const showRightPanel = ref(false)
const touchStartX = ref(0)
const touchStartY = ref(0)
const filters = ref({})

// 计算属性
const currentJob = computed(() => jobs.value[currentIndex.value])
const userProfile = computed(() => userStore.profile)
const visibleVideos = computed(() => {
  // 预加载当前、前一个、后一个视频（优化性能）
  return jobs.value.slice(
    Math.max(0, currentIndex.value - 1),
    Math.min(jobs.value.length, currentIndex.value + 2)
  )
})

// 加载视频列表
onMounted(async () => {
  const data = await fetchJobs({
    limit: 30,
    offset: 0,
    filters: filters.value,
  })
  jobs.value = data.jobs
  videoStore.setCurrentIndex(0)
})

// 手势处理
const onTouchStart = (e: TouchEvent) => {
  touchStartX.value = e.touches[0].clientX
  touchStartY.value = e.touches[0].clientY
}

const onTouchMove = (e: TouchEvent) => {
  const currentX = e.touches[0].clientX
  const currentY = e.touches[0].clientY
  const diffX = currentX - touchStartX.value
  const diffY = currentY - touchStartY.value

  // 水平滑动阈值 > 20%，触发左右面板
  const threshold = window.innerWidth * 0.2

  if (Math.abs(diffX) > Math.abs(diffY) && Math.abs(diffX) > threshold) {
    if (diffX > 0) {
      // 右滑打开右侧菜单
      showRightPanel.value = true
    } else {
      // 左滑打开左侧人才成型模块
      showLeftPanel.value = true
    }
  }
}

const onTouchEnd = (e: TouchEvent) => {
  // 触摸结束时的处理逻辑
}

// 视频导航
const nextVideo = () => {
  if (currentIndex.value < jobs.value.length - 1) {
    currentIndex.value++
    videoStore.setCurrentIndex(currentIndex.value)
  }
}

const prevVideo = () => {
  if (currentIndex.value > 0) {
    currentIndex.value--
    videoStore.setCurrentIndex(currentIndex.value)
  }
}

// 交互功能
const toggleFavorite = (jobId: string) => {
  const job = jobs.value.find(j => j.id === jobId)
  if (job) {
    job.likes = (job.likes || 0) + 1
    userStore.addFavorite(jobId)
  }
}

const shareVideo = (jobId: string) => {
  const job = jobs.value.find(j => j.id === jobId)
  if (navigator.share && job) {
    navigator.share({
      title: job.title,
      text: `查看这个职位：${job.title} - ${job.salary}`,
      url: `${window.location.origin}/?job=${jobId}`,
    })
  }
}

const goToCompany = (companyId: string) => {
  window.location.href = `/company/${companyId}`
}

const openLeftPanel = () => {
  showLeftPanel.value = true
}

const applyFilters = async (newFilters: Record<string, any>) => {
  filters.value = newFilters
  const data = await fetchJobs({
    limit: 30,
    offset: 0,
    filters: newFilters,
  })
  jobs.value = data.jobs
  currentIndex.value = 0
}

const submitApplication = async (applicationData: any) => {
  try {
    await apiSubmitApplication({
      job_id: currentJob.value.id,
      user_id: userStore.userId,
      profile_snapshot: applicationData,
    })
    showLeftPanel.value = false
    // 显示成功提示
    alert('✅ 人才成型模块已发送给HR！')
    nextVideo()
  } catch (error) {
    alert('❌ 投递失败，请重试')
  }
}
</script>

<style scoped lang="scss">
.video-feed-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background: #000;
}

.video-stack {
  position: relative;
  width: 100%;
  height: 100%;
}

.video-card {
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 0;
  transform: translateY(100%);
  transition: all 0.4s ease-out;

  &.active {
    opacity: 1;
    transform: translateY(0);
  }
}

.video-element {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.interaction-buttons {
  position: absolute;
  right: 16px;
  bottom: 100px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  z-index: 10;

  .icon-btn {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.2);
    border: none;
    color: white;
    font-size: 20px;
    cursor: pointer;
    backdrop-filter: blur(10px);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transition: background 0.3s;

    &:active {
      background: rgba(255, 255, 255, 0.3);
    }

    .count {
      font-size: 10px;
    }
  }

  .company-avatar {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    overflow: hidden;
    cursor: pointer;
    border: 2px solid white;

    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
  }
}

.job-info-overlay {
  position: absolute;
  bottom: 80px;
  left: 0;
  right: 0;
  padding: 16px;
  background: linear-gradient(180deg, transparent, rgba(0, 0, 0, 0.8));
  color: white;
  z-index: 5;

  .job-title {
    font-size: 24px;
    font-weight: bold;
    margin: 0 0 8px 0;
  }

  .job-meta {
    font-size: 14px;
    margin: 0 0 12px 0;
    opacity: 0.9;
  }

  .benefit-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;

    .tag {
      background: rgba(255, 255, 255, 0.2);
      padding: 4px 10px;
      border-radius: 16px;
      font-size: 12px;
    }
  }
}

.send-profile-btn {
  position: absolute;
  bottom: 20px;
  left: 16px;
  right: 16px;
  padding: 14px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  z-index: 10;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  transition: transform 0.2s;

  &:active {
    transform: scale(0.95);
  }
}

.video-slide-enter-active,
.video-slide-leave-active {
  transition: all 0.4s ease;
}

.video-slide-enter-from {
  transform: translateY(100%);
  opacity: 0;
}

.video-slide-leave-to {
  transform: translateY(-100%);
  opacity: 0;
}
</style>
