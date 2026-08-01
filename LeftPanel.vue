<template>
  <transition name="panel-slide-left">
    <div class="left-panel-wrapper">
      <!-- 半透明背景遮罩 -->
      <div class="panel-overlay" @click="$emit('close')"></div>

      <!-- 左侧面板主体 -->
      <div class="left-panel">
        <!-- ==================== 顶部：关闭按钮 & 标题 ==================== -->
        <div class="panel-header">
          <button class="close-btn" @click="$emit('close')">
            <span class="close-icon">✕</span>
          </button>
          <h2 class="panel-title">您的人才成型模块</h2>
          <div class="header-spacer"></div>
        </div>

        <!-- ==================== 匹配度状态栏 ==================== -->
        <div class="match-status-bar">
          <div class="match-header">
            <h3>岗位匹配度分析</h3>
            <span class="match-badge" :class="matchLevelClass">
              {{ matchRate }}% 匹配
            </span>
          </div>

          <div class="match-status-text">
            <p v-if="matchRate >= 80" class="status-good">
              ✅ 恭喜！您完全符合此岗位的核心要求
            </p>
            <p v-else-if="matchRate >= 60" class="status-medium">
              ⚠️ 您与此岗位的匹配度还不错，建议补充以下技能
            </p>
            <p v-else class="status-low">
              ❌ 您与此岗位的匹配度较低，建议先完成相关课程培训
            </p>
          </div>

          <div class="quick-stats">
            <div class="stat-item">
              <span class="stat-label">已拥有技能</span>
              <span class="stat-value">{{ ownedSkillsCount }}/{{ totalSkillsCount }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">已完成课程</span>
              <span class="stat-value">{{ completedCoursesCount }}/{{ requiredCoursesCount }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">测试成绩</span>
              <span class="stat-value">{{ testScore || '未考' }}</span>
            </div>
          </div>
        </div>

        <!-- ==================== 滚动内容区域 ==================== -->
        <div class="panel-scroll-content">
          <!-- -------- 板块一：技能与证书 -------- -->
          <section class="content-section">
            <div class="section-header">
              <h3 class="section-title">📚 技能与证书要求</h3>
              <span class="section-progress">{{ ownedSkillsCount }}/{{ totalSkillsCount }}</span>
            </div>

            <div class="skills-container">
              <div 
                v-for="skill in jobRequirements.skills" 
                :key="skill.id"
                class="skill-card"
                :class="{ 'skill-owned': userHasSkill(skill.name) }"
              >
                <div class="skill-icon">
                  <span v-if="userHasSkill(skill.name)" class="check-icon">✓</span>
                  <span v-else class="lock-icon">🔒</span>
                </div>
                <div class="skill-info">
                  <p class="skill-name">{{ skill.name }}</p>
                  <p class="skill-level">{{ skill.level || '中级' }}</p>
                </div>
                <div v-if="!userHasSkill(skill.name)" class="learn-btn">
                  <button @click="gotoLearnCourse(skill.course_id)" class="btn-small">
                    学习
                  </button>
                </div>
              </div>
            </div>
          </section>

          <!-- -------- 板块二：相关课程 -------- -->
          <section class="content-section">
            <div class="section-header">
              <h3 class="section-title">🎓 推荐课程</h3>
              <span class="section-progress">{{ completedCoursesCount }}/{{ requiredCoursesCount }}</span>
            </div>

            <div class="courses-container">
              <div 
                v-for="course in jobRequirements.courses" 
                :key="course.id"
                class="course-card"
                :class="{ 'course-completed': userCompletedCourse(course.id) }"
              >
                <div class="course-image">
                  <img :src="course.cover_image" :alt="course.name" />
                  <div v-if="userCompletedCourse(course.id)" class="course-badge">
                    ✓ 已完成
                  </div>
                </div>
                <div class="course-details">
                  <h4 class="course-name">{{ course.name }}</h4>
                  <p class="course-desc">{{ course.description }}</p>
                  <div class="course-meta">
                    <span class="course-duration">⏱️ {{ course.duration }}小时</span>
                    <span class="course-difficulty">{{ course.difficulty }}</span>
                  </div>
                  <button 
                    v-if="!userCompletedCourse(course.id)"
                    @click="enrollCourse(course.id)"
                    class="btn-enroll"
                  >
                    立即学习
                  </button>
                </div>
              </div>
            </div>
          </section>

          <!-- -------- 板块三：在线测试 & 成绩 -------- -->
          <section class="content-section">
            <div class="section-header">
              <h3 class="section-title">🧪 岗位能力测评</h3>
            </div>

            <div class="test-section">
              <div class="test-card" :class="{ 'test-completed': userTestScore }">
                <div class="test-header">
                  <h4>{{ job.title }} 岗位评测</h4>
                  <span v-if="userTestScore" class="test-score-badge">
                    {{ userTestScore }}分
                  </span>
                </div>
                <p class="test-desc">{{ job.test_description || '了解您是否具备此岗位的核心能力' }}</p>
                
                <div v-if="userTestScore" class="test-result">
                  <div class="score-bar">
                    <div class="score-fill" :style="{ width: userTestScore + '%' }"></div>
                  </div>
                  <p class="result-text">
                    <span v-if="userTestScore >= 80" class="result-good">
                      🎉 优秀！您的能力完全符合岗位要求
                    </span>
                    <span v-else-if="userTestScore >= 60" class="result-fair">
                      👍 良好，建议重点学习标记的弱项
                    </span>
                    <span v-else class="result-poor">
                      📝 需要加强，建议重新学习相关课程后再试
                    </span>
                  </p>
                </div>

                <button 
                  v-if="!userTestScore || retakeTest"
                  @click="startTest"
                  class="btn-primary"
                >
                  {{ userTestScore ? '重新测试' : '开始测评' }}
                </button>
              </div>
            </div>
          </section>

          <!-- -------- 板块四：项目经验 & 作品 -------- -->
          <section class="content-section">
            <div class="section-header">
              <h3 class="section-title">💼 相关项目经验</h3>
            </div>

            <div class="portfolio-container">
              <!-- 用户已上传的作品 -->
              <div v-if="userPortfolio.length > 0" class="portfolio-list">
                <div 
                  v-for="portfolio in userPortfolio" 
                  :key="portfolio.id"
                  class="portfolio-item"
                >
                  <div class="portfolio-thumbnail">
                    <img :src="portfolio.thumbnail" :alt="portfolio.title" />
                  </div>
                  <div class="portfolio-info">
                    <h4>{{ portfolio.title }}</h4>
                    <p class="portfolio-tech">
                      <span v-for="tech in portfolio.technologies" :key="tech" class="tech-tag">
                        {{ tech }}
                      </span>
                    </p>
                    <p class="portfolio-desc">{{ portfolio.description }}</p>
                  </div>
                  <div class="portfolio-actions">
                    <button @click="viewPortfolio(portfolio.id)" class="btn-view">
                      查看
                    </button>
                  </div>
                </div>
              </div>

              <!-- 空状态 -->
              <div v-else class="empty-portfolio">
                <p class="empty-icon">📁</p>
                <p class="empty-text">还没有上传作品？立即上传您的项目作品</p>
                <button @click="goToUploadPortfolio" class="btn-upload">
                  上传作品集
                </button>
              </div>
            </div>
          </section>

          <!-- -------- 板块五：真实工作场景 -------- -->
          <section class="content-section">
            <div class="section-header">
              <h3 class="section-title">🏢 工作环境与场景</h3>
            </div>

            <div class="workplace-carousel">
              <!-- 场景图片轮播 -->
              <div class="carousel-container">
                <div class="carousel-main">
                  <img 
                    :src="currentSceneImage" 
                    :alt="currentSceneTitle"
                    class="scene-image"
                  />
                  <div class="scene-info-overlay">
                    <h4>{{ currentSceneTitle }}</h4>
                    <p>{{ currentSceneDesc }}</p>
                  </div>
                </div>

                <!-- 缩略图导航 -->
                <div class="carousel-thumbnails">
                  <div 
                    v-for="(scene, index) in workplaceScenes" 
                    :key="index"
                    class="thumbnail"
                    :class="{ 'active': sceneIndex === index }"
                    @click="sceneIndex = index"
                  >
                    <img :src="scene.thumbnail" :alt="scene.title" />
                  </div>
                </div>
              </div>

              <!-- 场景视频 -->
              <div v-if="workplaceVideo" class="workplace-video">
                <video
                  :src="workplaceVideo"
                  controls
                  class="video-player"
                >
                  您的浏览器不支持视频播放
                </video>
              </div>
            </div>
          </section>

          <!-- -------- 板块六：企业信息 -------- -->
          <section class="content-section">
            <div class="section-header">
              <h3 class="section-title">🏭 企业信息</h3>
            </div>

            <div class="company-info-card">
              <div class="company-header">
                <img :src="company.logo" :alt="company.name" class="company-logo" />
                <div class="company-basic">
                  <h3>{{ company.name }}</h3>
                  <p class="company-type">
                    {{ company.industry }} | {{ company.scale }} | {{ company.founded_year }}年成立
                  </p>
                </div>
              </div>

              <div class="company-details">
                <div class="detail-row">
                  <span class="label">📍 工作地址</span>
                  <span class="value">{{ job.address }}</span>
                  <button @click="openMap" class="btn-map">导航</button>
                </div>
                <div class="detail-row">
                  <span class="label">📞 联系电话</span>
                  <span class="value">{{ job.contact_phone }}</span>
                  <button @click="callCompany" class="btn-call">呼叫</button>
                </div>
                <div class="detail-row">
                  <span class="label">✉️ 邮箱</span>
                  <span class="value">{{ job.contact_email }}</span>
                </div>
              </div>

              <div class="company-description">
                <h4>公司简介</h4>
                <p>{{ company.description }}</p>
              </div>

              <div class="company-benefits">
                <h4>企业福利</h4>
                <div class="benefits-grid">
                  <span v-for="benefit in company.benefits" :key="benefit" class="benefit-tag">
                    {{ benefit }}
                  </span>
                </div>
              </div>
            </div>
          </section>

          <!-- 底部空白区 -->
          <div class="panel-bottom-spacer"></div>
        </div>

        <!-- ==================== 底部固定提交按钮 ==================== -->
        <div class="panel-footer">
          <button 
            @click="submitApplication"
            class="btn-submit-primary"
            :disabled="isSubmitting"
          >
            <span v-if="!isSubmitting">📋 发送人才成型模块给HR</span>
            <span v-else>提交中...</span>
          </button>
          <p class="footer-tip">
            点击上方按钮，将您的完整画像（技能、经验、作品）发送给招聘官
          </p>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'
import { useUserStore } from '@/stores/userStore'
import { 
  fetchJobDetails, 
  calculateMatchRate,
  submitApplication as apiSubmitApplication,
  enrollCourse as apiEnrollCourse,
  fetchUserPortfolio,
  fetchWorkplaceScenes,
} from '@/api/client'

// ==================== 接口定义 ====================
interface JobRequirement {
  id: string
  name: string
  level?: string
  course_id?: string
}

interface Course {
  id: string
  name: string
  description: string
  cover_image: string
  duration: number
  difficulty: string
}

interface JobData {
  id: string
  title:
