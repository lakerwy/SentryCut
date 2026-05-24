<template>
  <div>
    <div class="page-header">
      <h2>素材库</h2>
      <p>管理和浏览您的视频素材</p>
    </div>

    <div class="filter-bar">
      <div class="filter-tabs">
        <div class="filter-tab active">全部</div>
        <div class="filter-tab">视频</div>
        <div class="filter-tab">最近使用</div>
      </div>
      <div class="filter-actions">
        <button class="btn btn-ghost">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polygon points="22,3 2,3 10,12.46 10,19 14,21 14,12.46 22,3"/>
          </svg>
          筛选
        </button>
        <button class="btn btn-ghost">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <line x1="8" y1="6" x2="21" y2="6"/>
            <line x1="8" y1="12" x2="21" y2="12"/>
            <line x1="8" y1="18" x2="21" y2="18"/>
            <line x1="3" y1="6" x2="3.01" y2="6"/>
            <line x1="3" y1="12" x2="3.01" y2="12"/>
            <line x1="3" y1="18" x2="3.01" y2="18"/>
          </svg>
          排序
        </button>
        <label class="btn btn-accent" style="cursor: pointer;">
          <input type="file" accept=".mp4,.mov,video/mp4,video/quicktime" multiple hidden @change="handleFiles">
          上传素材
        </label>
      </div>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="empty-state">
      <p>加载中...</p>
    </div>

    <!-- Empty State -->
    <div v-else-if="materials.length === 0" class="empty-state">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
        <rect x="3" y="3" width="18" height="18" rx="2"/>
        <path d="M3 9h18M9 21V9"/>
      </svg>
      <h3>暂无素材</h3>
      <p>上传您的第一个视频素材开始使用</p>
    </div>

    <!-- Asset Grid -->
    <div v-else class="asset-grid">
      <div v-for="material in materials" :key="material.id" class="asset-card">
        <div class="asset-thumbnail">
          <div class="asset-type">视频</div>
          <div v-if="material.duration" class="asset-duration">{{ formatDuration(material.duration) }}</div>
        </div>
        <div class="asset-info">
          <div class="asset-title">{{ material.original_filename }}</div>
          <div class="asset-meta">
            <span v-if="material.width && material.height">{{ material.width }}×{{ material.height }}</span>
            <span v-if="material.width && material.height" class="asset-meta-dot"></span>
            <span>{{ formatFileSize(material.file_size) }}</span>
            <span class="asset-meta-dot"></span>
            <span>{{ getStatusLabel(material.status) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { ElMessage } from "element-plus";
import { listMaterials, uploadMaterial } from "@/api/material";
import type { Material } from "@/api/types";

const materials = ref<Material[]>([]);
const loading = ref(false);

function formatDuration(seconds: number): string {
  const m = Math.floor(seconds / 60);
  const s = Math.round(seconds % 60);
  return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
}

function formatFileSize(bytes: number | null): string {
  if (bytes == null) return '-';
  if (bytes >= 1024 * 1024 * 1024) {
    return (bytes / (1024 * 1024 * 1024)).toFixed(1) + ' GB';
  }
  if (bytes >= 1024 * 1024) {
    return (bytes / (1024 * 1024)).toFixed(0) + ' MB';
  }
  return (bytes / 1024).toFixed(0) + ' KB';
}

function getStatusLabel(status: string): string {
  const labels: Record<string, string> = {
    pending: '待处理',
    indexing: '索引中',
    indexed: '已索引',
    failed: '失败'
  };
  return labels[status] || status;
}

async function loadMaterials() {
  loading.value = true;
  try {
    materials.value = await listMaterials();
  } catch (error) {
    ElMessage.error(error instanceof Error ? error.message : "加载失败");
  } finally {
    loading.value = false;
  }
}

async function handleFiles(event: Event) {
  const target = event.target as HTMLInputElement;
  const files = target.files;
  if (!files || files.length === 0) return;

  for (const file of Array.from(files)) {
    try {
      await uploadMaterial(file);
      ElMessage.success(`${file.name} 上传成功`);
    } catch (error) {
      ElMessage.error(error instanceof Error ? error.message : "上传失败");
    }
  }

  target.value = '';
  await loadMaterials();
}

onMounted(loadMaterials);
</script>
