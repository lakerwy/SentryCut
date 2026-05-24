<template>
  <div>
    <div class="page-header">
      <h2>创建视频</h2>
      <p>输入口播文案，生成分镜并匹配素材，一键合成带配音和字幕的 MP4。</p>
    </div>

    <div class="workspace-grid">
      <div class="card" style="padding: var(--space-5);">
        <h3 style="margin: 0 0 var(--space-4); font-size: var(--text-lg);">口播设置</h3>

        <div style="margin-bottom: var(--space-4);">
          <label style="display: block; font-size: var(--text-sm); font-weight: 500; margin-bottom: var(--space-2);">口播文案</label>
          <textarea
            v-model="script"
            class="input"
            style="width: 100%; min-height: 200px; resize: vertical;"
            placeholder="请输入完整口播文案"
          ></textarea>
        </div>

        <div style="margin-bottom: var(--space-4);">
          <label style="display: block; font-size: var(--text-sm); font-weight: 500; margin-bottom: var(--space-2);">视频比例</label>
          <div style="display: flex; gap: var(--space-2);">
            <button
              v-for="ratio in aspectOptions"
              :key="ratio"
              class="btn"
              :class="aspectRatio === ratio ? 'btn-primary' : 'btn-ghost'"
              @click="aspectRatio = ratio"
            >{{ ratio }}</button>
          </div>
        </div>

        <div style="margin-bottom: var(--space-4);">
          <label style="display: block; font-size: var(--text-sm); font-weight: 500; margin-bottom: var(--space-2);">配音音色</label>
          <select v-model="voice" class="input" style="width: 100%;">
            <option value="zh-CN-XiaoxiaoNeural">中文女声-晓晓</option>
            <option value="zh-CN-YunxiNeural">中文男声-云希</option>
            <option value="zh-CN-YunjianNeural">中文男声-云健</option>
            <option value="zh-CN-XiaoyiNeural">中文女声-晓伊</option>
          </select>
        </div>

        <div style="display: flex; gap: var(--space-2);">
          <button class="btn btn-primary" :disabled="planning" @click="handlePlan">
            {{ planning ? '处理中...' : '生成分镜' }}
          </button>
          <button class="btn btn-ghost" :disabled="matching || segments.length === 0" @click="handleMatch">
            匹配素材
          </button>
          <button class="btn btn-accent" :disabled="rendering" @click="handleRender">
            {{ rendering ? '生成中...' : '生成视频' }}
          </button>
        </div>

        <div v-if="taskStore.currentTask?.type === 'render'" style="margin-top: var(--space-4); display: flex; align-items: center; gap: var(--space-3);">
          <span class="text-muted" style="font-size: var(--text-sm);">{{ taskStore.currentTask.status }}</span>
          <div style="flex: 1; height: 4px; background: var(--border); border-radius: 2px; overflow: hidden;">
            <div style="height: 100%; background: var(--accent); transition: width 0.3s;" :style="{ width: taskStore.currentTask.progress + '%' }"></div>
          </div>
          <span class="text-muted" style="font-size: var(--text-xs);">{{ taskStore.currentTask.current_step }}</span>
        </div>
      </div>

      <div class="card" style="padding: var(--space-5);">
        <h3 style="margin: 0 0 var(--space-4); font-size: var(--text-lg);">分镜预览</h3>

        <div v-if="segments.length === 0" class="empty-state" style="min-height: 200px;">
          <p>输入文案后生成分镜，匹配结果会显示在这里。</p>
        </div>

        <div v-else style="display: flex; flex-direction: column; gap: var(--space-3);">
          <div v-for="segment in segments" :key="segment.index" style="border: 1px solid var(--border); border-radius: var(--radius-md); padding: var(--space-3); background: var(--bg);">
            <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: var(--space-2);">
              <strong style="font-size: var(--text-sm);">分镜 {{ segment.index }}</strong>
              <span v-if="segment.similarity_score != null" class="text-accent" style="font-size: var(--text-xs);">{{ segment.similarity_score.toFixed(2) }}</span>
            </div>
            <p style="font-size: var(--text-sm); margin: var(--space-1) 0;"><b>口播：</b>{{ segment.narration }}</p>
            <p style="font-size: var(--text-sm); margin: var(--space-1) 0; color: var(--muted);"><b>画面：</b>{{ segment.visual_query }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { useRouter } from "vue-router";
import { ElMessage } from "element-plus";
import { matchClips, planScript, renderVideo } from "@/api/video";
import type { AspectRatio, Segment, Task } from "@/api/types";
import { useTaskStore } from "@/stores/task";

const router = useRouter();
const taskStore = useTaskStore();
const script = ref("");
const voice = ref("zh-CN-XiaoxiaoNeural");
const aspectRatio = ref<AspectRatio>("9:16");
const aspectOptions = ["9:16", "16:9", "1:1"] as AspectRatio[];
const segments = ref<Segment[]>([]);
const planning = ref(false);
const matching = ref(false);
const rendering = ref(false);

function requireScript() {
  if (!script.value.trim()) {
    ElMessage.warning("请先输入口播文案");
    return false;
  }
  return true;
}

async function handlePlan() {
  if (!requireScript()) return;
  planning.value = true;
  try {
    segments.value = await planScript({
      script: script.value,
      aspect_ratio: aspectRatio.value,
      voice: voice.value,
    });
    ElMessage.success("分镜已生成");
  } catch (error) {
    ElMessage.error(error instanceof Error ? error.message : "生成分镜失败");
  } finally {
    planning.value = false;
  }
}

async function handleMatch() {
  if (segments.value.length === 0) return;
  matching.value = true;
  try {
    segments.value = await matchClips({ segments: segments.value, results: 1 });
    ElMessage.success("素材匹配完成");
  } catch (error) {
    ElMessage.error(error instanceof Error ? error.message : "匹配失败");
  } finally {
    matching.value = false;
  }
}

async function handleRender() {
  if (!requireScript()) return;
  rendering.value = true;
  try {
    const task = await renderVideo({
      script: script.value,
      voice: voice.value,
      aspect_ratio: aspectRatio.value,
      segments: segments.value,
    });
    taskStore.startPolling(task.task_id, (done: Task) => {
      rendering.value = false;
      if (done.status === "completed") {
        ElMessage.success("视频生成完成");
        void router.push(`/preview/${done.task_id}`);
      } else if (done.status === "failed") {
        ElMessage.error(done.error_message || "生成失败");
      }
    });
    void router.push(`/tasks/${task.task_id}`);
  } catch (error) {
    rendering.value = false;
    ElMessage.error(error instanceof Error ? error.message : "生成视频失败");
  }
}
</script>
