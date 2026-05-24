<template>
  <div>
    <div class="page-header">
      <h2>视频预览</h2>
      <p>{{ taskId }}</p>
    </div>

    <div class="card" style="padding: var(--space-5);">
      <div v-if="task?.status !== 'completed'" class="empty-state" style="min-height: 300px;">
        <div>
          <p style="margin-bottom: var(--space-3);">视频还没有生成完成。</p>
          <button class="btn btn-primary" @click="$router.push(`/tasks/${taskId}`)">查看任务</button>
        </div>
      </div>

      <template v-else>
        <video
          :src="task.output_url || ''"
          controls
          style="width: 100%; max-width: 960px; margin: 0 auto; display: block; border-radius: var(--radius-md); background: oklch(15% 0.02 240);"
        />
        <div style="display: flex; gap: var(--space-2); margin-top: var(--space-4); justify-content: center;">
          <a
            v-if="task.output_url"
            :href="task.output_url"
            download
            class="btn btn-primary"
          >
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
              <polyline points="7,10 12,15 17,10"/>
              <line x1="12" y1="15" x2="12" y2="3"/>
            </svg>
            下载 MP4
          </a>
          <a
            v-if="task.subtitle_url"
            :href="task.subtitle_url"
            download
            class="btn btn-ghost"
          >
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
              <polyline points="14,2 14,8 20,8"/>
              <line x1="16" y1="13" x2="8" y2="13"/>
              <line x1="16" y1="17" x2="8" y2="17"/>
            </svg>
            下载字幕
          </a>
        </div>
      </template>
    </div>

    <div v-if="segments.length" class="card" style="margin-top: var(--space-5); padding: var(--space-5);">
      <h3 style="margin: 0 0 var(--space-4); font-size: var(--text-lg);">使用的素材片段</h3>
      <div style="display: flex; flex-direction: column; gap: var(--space-3);">
        <div v-for="segment in segments" :key="segment.index" style="border: 1px solid var(--border); border-radius: var(--radius-md); padding: var(--space-3); background: var(--bg);">
          <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: var(--space-2);">
            <strong style="font-size: var(--text-sm);">分镜 {{ segment.index }}</strong>
            <span v-if="segment.similarity_score != null" class="text-accent" style="font-size: var(--text-xs);">{{ segment.similarity_score.toFixed(2) }}</span>
          </div>
          <p style="font-size: var(--text-sm); margin: var(--space-1) 0;"><b>口播：</b>{{ segment.narration }}</p>
          <p style="font-size: var(--text-sm); margin: var(--space-1) 0; color: var(--muted);"><b>素材：</b>{{ segment.source_file }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted } from "vue";
import { useTaskStore } from "@/stores/task";

const props = defineProps<{ taskId: string }>();
const taskStore = useTaskStore();
const task = computed(() => taskStore.currentTask);
const segments = computed(() => task.value?.result_json?.segments ?? []);

async function loadTask() {
  await taskStore.load(props.taskId);
}

onMounted(loadTask);
</script>
