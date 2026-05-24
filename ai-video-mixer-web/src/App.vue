<template>
  <div class="app">
    <!-- Sidebar -->
    <aside class="sidebar">
      <div class="sidebar-brand">
        <h1>
          <span class="brand-icon">◐</span>
          <span>视频混剪助手</span>
        </h1>
      </div>

      <nav class="sidebar-nav">
        <div class="nav-section">
          <div class="nav-section-title">工作区</div>
          <RouterLink to="/materials" class="nav-item" :class="{ active: $route.path === '/materials' }">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <rect x="3" y="3" width="18" height="18" rx="2"/>
              <path d="M3 9h18M9 21V9"/>
            </svg>
            <span>素材库</span>
          </RouterLink>
          <RouterLink to="/create" class="nav-item" :class="{ active: $route.path === '/create' }">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 19l7-7 3 3-7 7-3-3z"/>
              <path d="M18 13l-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"/>
              <path d="M2 2l7.586 7.586"/>
              <circle cx="11" cy="11" r="2"/>
            </svg>
            <span>创建视频</span>
          </RouterLink>
        </div>
      </nav>
    </aside>

    <!-- Main Content -->
    <main class="main">
      <!-- Header -->
      <header class="header">
        <div class="header-left">
          <div class="breadcrumb">
            <a href="#">工作台</a>
            <span class="breadcrumb-separator">/</span>
            <span>{{ currentPageTitle }}</span>
          </div>
        </div>
        <div class="header-right">
          <div class="search-box">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <circle cx="11" cy="11" r="8"/>
              <path d="m21 21-4.35-4.35"/>
            </svg>
            <input type="text" class="input" placeholder="搜索素材、项目...">
          </div>
          <button class="btn btn-primary">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="12" y1="5" x2="12" y2="19"/>
              <line x1="5" y1="12" x2="19" y2="12"/>
            </svg>
            新建项目
          </button>
        </div>
      </header>

      <!-- Content Area -->
      <div class="content">
        <RouterView />
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();

const currentPageTitle = computed(() => {
  if (route.path === '/materials') return '素材库';
  if (route.path === '/create') return '创建视频';
  if (route.path.startsWith('/tasks/')) return '任务详情';
  if (route.path.startsWith('/preview/')) return '视频预览';
  return '素材库';
});
</script>
