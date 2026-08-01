<template>
  <transition name="panel-slide-left">
    <div v-if="true" class="left-panel">
      <!-- 顶部匹配状态 -->
      <div class="match-header">
        <button class="close-btn" @click="$emit('close')">✕</button>
        <h2>岗位匹配度</h2>
      </div>

      <div class="match-status">
        <div class="match-score">
          <div class="score-circle">
            <span class="percentage">{{ matchRate }}%</span>
          </div>
          <p class="status-text">{{ matchStatus }}</p>
        </div>
      </div>

      <!-- 滚动内容区 -->
      <div class="panel-content">
        <!-- 1. 技能与证书 -->
        <section class="section">
          <h3>📚 技能与证书</h3>
          <div class="skill-grid">
            <div 
              v-for="skill in job.requirements.skills || []" 
              :key="skill"
              class="skill-badge"
              :class="{ 'has-skill': userProfile.skills.includes(skill) }"
            >
