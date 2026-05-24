<template>
  <div>
    <div class="page-header">
      <h2>任务详情</h2>
      <p>{{ taskId }}</p>
    </div>

    <div class="workspace-grid">
      <div class="card" style="padding: var(--space-5);">
        <h3 style="margin: 0 0 var(--space-4); font-size: var(--text-lg);">进度</h3>

        <div v-if="!task" class="empty-state" style="min-height: 120px;">
          <p>正在加载任务</p>
        </div>

        <template v-else>
          <div style="display: flex; gap: var(--space-2); margin-bottom: var(--space-4);">
            <span class="btn" style="cursor: default; padding: var(--space-1) var(--space-3); font-size: var(--text-xs;" :class="getStatusClass(task.status)">{{ task.type }}</span>
            <span class="btn" style="cursor: default; padding: var(--space-1) var(--space-3); font-size: var(--text-xs;" :class="getStatusClass(task.status)">{{ task.status }}</span>
          </div>

          <div style="height: 8px; background: var(--border); border-radius: 4px; overflow: hidden; margin-bottom: var(--space-2);">
            <div style="height: 100%; background: var(--accent); transition: width 0.3s;" :style="{ width: task.progress + '%' }"></div>
          </div>

          <p class="text-muted" style="font-size: var(--text-sm);">{{ task.current_step }}</p>

          <div v-if="task.error_message" style="margin-top: var(--space-3); padding: var(--space-3); background: oklch(55% 0.18 25 / 0.1); border-radius: var(--radius-md); color: var(--danger); font-size: var(--text-sm);">
            {{ task.error_message }}
          </div>
        </template>
      </div>

      <div class="card" style="padding: var(--space-5);">
        <h3 style="margin: 0 0 var(--space-4); font-size: var(--text-lg);">结果</h3>

        <div v-if="!task?.result_json" class="empty-state" style="min-height: 120px;">
          <p>任务完成后会显示输出结果。</p>
        </div>

        <template v-else>
          <div class="property-row">
            <span class="property-label">输出视频</span>
            <a v-if="task.output_url" :href="task.output_url" target="_blank" class="text-accent" style="font-size: var(--text-sm);">{{ task.output_url }}</a>
            <span v-else class="text-muted">无</span>
          </div>
          <div class="property-row">
            <span class="property-label">字幕文件</span>
            <a v-if="task.subtitle_url" :href="task.subtitle_url" target="_blank" class="text-accent" style="font-size: var(--text-sm);">{{ task.subtitle_url }}</a>
            <span v-else class="text-muted">无</span>
          </div>
        </template>
      </div>
    </div>

    <div v-if="segments.length" class="card" style="margin-top: var(--space-5); padding: var(--space-5);">
      <h3 style="margin: 0 0 var(--space-4); font-size: var(--text-lg);">分镜记录</h3>
      <div style="overflow-x: auto;">
        <table style="width: 100%; border-collapse: collapse; font-size: var(--text-sm);">
          <thead>
            <tr style="border-bottom: 1px solid var(--border);">
              <th style="padding: var(--space-2); text-align: left; color: var(--muted); font-weight: 500;">#</th>
              <th style="padding: var(--space-2); text-align: left; color: var(--muted); font-weight: 500;">口播</th>
              <th style="padding: var(--space-2); text-align: left; color: var(--muted); font-weight: 500;">画面检索</th>
              <th style="padding: var(--space-2); text-align: left; color: var(--muted); font-weight: 500;">时长</th>
              <th style="padding: var(--space-2); text-align: left; color: var(--muted); font-weight: 500;">相似度</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="segment in segments" :key="segment.index" style="border-bottom: 1px solid var(--border);">
              <td style="padding: var(--space-2);">{{ segment.index }}</td>
              <td style="padding: var(--space-2);">{{ segment.narration }}</td>
              <td style="padding: var(--space-2); color: var(--muted);">{{ segment.visual_query }}</td>
              <td style="padding: var(--space-2);">{{ segment.actual_duration ?? segment.estimated_duration }}s</td>
              <td style="padding: var(--space-2);">{{ segment.similarity_score == null ? '-' : segment.similarity_score.toFixed(2) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted } from "vue";
import { useTaskStore } from "@/stores/task";

const props = defineProps<{ taskId: string }>();
const taskStore = useTaskStore();
const task = computed(() => taskStore.currentTask);
const segments = computed(() => task.value?.result_json?.segments ?? []);

function getStatusClass(status: string) {
  if (status === "completed") return "btn-primary";
  if (status === "failed") return "btn-accent";
  return "btn-ghost";
}

async function loadTask() {
  await taskStore.load(props.taskId);
}

onMounted(() => {
  taskStore.startPolling(props.taskId);
});

onUnmounted(() => {
  taskStore.stopPolling();
});
</script>
