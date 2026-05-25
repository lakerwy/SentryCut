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
        <button
          class="btn btn-primary"
          :disabled="selectedIds.length === 0"
          @click="startIndex"
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M12 2L2 7l10 5 10-5-10-5z"/>
            <path d="M2 17l10 5 10-5"/>
            <path d="M2 12l10 5 10-5"/>
          </svg>
          建立索引 ({{ selectedIds.length }})
        </button>
        <label class="btn btn-accent" style="cursor: pointer;">
          <input type="file" accept=".mp4,.mov,video/mp4,video/quicktime" multiple hidden @change="handleFiles">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
            <polyline points="17,8 12,3 7,8"/>
            <line x1="12" y1="3" x2="12" y2="15"/>
          </svg>
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

    <!-- Task Status -->
    <div v-if="taskStore.currentTask?.type === 'index'" class="card" style="margin-bottom: var(--space-5); padding: var(--space-4);">
      <div style="display: flex; align-items: center; gap: var(--space-4);">
        <span :class="['text-mono', taskStore.currentTask.status === 'completed' ? 'text-accent' : 'text-muted']">
          {{ taskStore.currentTask.status }}
        </span>
        <div style="flex: 1; height: 6px; background: var(--border); border-radius: 3px; overflow: hidden;">
          <div
            :style="{ width: taskStore.currentTask.progress + '%', height: '100%', background: 'var(--accent)', transition: 'width 0.3s' }"
          ></div>
        </div>
        <span class="text-muted" style="font-size: var(--text-sm);">{{ taskStore.currentTask.current_step }}</span>
      </div>
    </div>

    <!-- Video Player Modal -->
    <div v-if="playingMaterial" class="video-modal" @click.self="closePlayer">
      <div class="video-modal-content">
        <div class="video-modal-header">
          <span class="video-modal-title">{{ playingMaterial.original_filename }}</span>
          <button class="video-modal-close" @click="closePlayer">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="18" y1="6" x2="6" y2="18"/>
              <line x1="6" y1="6" x2="18" y2="18"/>
            </svg>
          </button>
        </div>
        <div class="video-modal-body">
          <video
            :src="`/static/materials/${playingMaterial.stored_filename}`"
            controls
            autoplay
          />
        </div>
      </div>
    </div>

    <!-- Asset Grid -->
    <div v-if="materials.length > 0" class="asset-grid">
      <div
        v-for="material in materials"
        :key="material.id"
        class="asset-card"
        :class="{ selected: isSelected(material.id) }"
        @click="toggleSelection(material.id)"
      >
        <div class="asset-thumbnail" @click.stop="openPlayer(material)">
          <div class="asset-checkbox" @click.stop>
            <input
              type="checkbox"
              :checked="isSelected(material.id)"
              @change="toggleSelection(material.id)"
            >
          </div>
          <div class="asset-type">视频</div>
          <div v-if="material.duration" class="asset-duration">{{ formatDuration(material.duration) }}</div>
          <button class="asset-play" @click.stop="openPlayer(material)">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor" style="margin-left: 2px;">
              <polygon points="5,3 19,12 5,21"/>
            </svg>
          </button>
          <button
            class="asset-delete"
            :disabled="material.status === 'indexing' || deletingId === material.id"
            @click.stop="handleDelete(material)"
          >
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="3,6 5,6 21,6"/>
              <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/>
            </svg>
          </button>
        </div>
        <div class="asset-info">
          <div class="asset-title">{{ material.original_filename }}</div>
          <div class="asset-meta">
            <span v-if="material.width && material.height">{{ material.width }}×{{ material.height }}</span>
            <span v-if="material.width && material.height" class="asset-meta-dot"></span>
            <span>{{ formatFileSize(material.file_size) }}</span>
            <span class="asset-meta-dot"></span>
            <span :class="{ 'text-accent': material.status === 'indexed', 'text-muted': material.status !== 'indexed' }">
              {{ getStatusLabel(material.status) }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { ElMessage } from "element-plus";
import { deleteMaterial, indexMaterials, listMaterials, uploadMaterial } from "@/api/material";
import type { Material } from "@/api/types";
import { useTaskStore } from "@/stores/task";

const materials = ref<Material[]>([]);
const loading = ref(false);
const selectedIds = ref<string[]>([]);
const deletingId = ref("");
const playingMaterial = ref<Material | null>(null);
const taskStore = useTaskStore();

function openPlayer(material: Material) {
  playingMaterial.value = material;
}

function closePlayer() {
  playingMaterial.value = null;
}

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

async function handleDelete(material: Material) {
  if (!confirm(`确定删除「${material.original_filename}」吗？已建立的索引片段也会一起清理。`)) {
    return;
  }

  deletingId.value = material.id;
  try {
    const result = await deleteMaterial(material.id);
    selectedIds.value = selectedIds.value.filter((id) => id !== material.id);
    ElMessage.success(`已删除素材，清理 ${result.removed_chunks} 个索引片段`);
    await loadMaterials();
  } catch (error) {
    ElMessage.error(error instanceof Error ? error.message : "删除失败");
  } finally {
    deletingId.value = "";
  }
}

async function startIndex() {
  if (selectedIds.value.length === 0) {
    ElMessage.warning("请先选择要索引的素材");
    return;
  }

  try {
    const task = await indexMaterials(selectedIds.value);
    taskStore.startPolling(task.task_id, async (done) => {
      await loadMaterials();
      if (done.status === "completed") ElMessage.success("索引完成");
      if (done.status === "failed") ElMessage.error(done.error_message || "索引失败");
    });
    ElMessage.info("已开始建立索引");
  } catch (error) {
    ElMessage.error(error instanceof Error ? error.message : "索引失败");
  }
}

function toggleSelection(id: string) {
  const idx = selectedIds.value.indexOf(id);
  if (idx === -1) {
    selectedIds.value.push(id);
  } else {
    selectedIds.value.splice(idx, 1);
  }
}

function isSelected(id: string): boolean {
  return selectedIds.value.includes(id);
}

onMounted(loadMaterials);
</script>
