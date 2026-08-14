<template>
  <div class="console">
    <!-- 导演工作台 -->
    <section class="hero">
      <div class="kicker">文旅漫剧 · 导演工作台</div>
      <h1 class="greeting">导演，欢迎回来</h1>
      <div class="motto">元途一启，万象随行</div>

      <div class="console-bar">
        <div class="cmd-input">
          <t-input
            v-model="promptText"
            size="large"
            placeholder="输入一句话，例如「用国风做一集王阳明的龙场悟道」"
            @enter="startCreate" />
          <div class="drop" title="拖入图片 / 老照片 / 文物图">
            <i-upload-picture class="drop-icon" />
            <span>拖入图片 / 老照片 / 文物图</span>
          </div>
        </div>
        <div class="cmd-modes">
          <div class="modes">
            <button :class="{ on: mode === 'agent' }" @click="mode = 'agent'">Agent 模式</button>
            <button :class="{ on: mode === 'precise' }" @click="mode = 'precise'">精准设置</button>
            <button :class="{ on: mode === 'template' }" @click="mode = 'template'">模板</button>
          </div>
          <span class="hint">Agent 托管出片 · 精准设置精修 · 沉淀为 Skill 复用</span>
          <div class="actions">
            <button class="save-skill" title="把当前项目沉淀为可复用 Skill">
              <i-cloud-upload class="ss-icon" />
              保存为 Skill
            </button>
            <t-button theme="primary" size="large" class="start" @click="startCreate">
              <template #icon><i-play class="start-icon" /></template>
              开始创作
            </t-button>
          </div>
        </div>
      </div>
    </section>

    <!-- 四类文旅模板 -->
    <section class="section">
      <div class="section-head">
        <h2>从一套文旅范式开始</h2>
      </div>
      <div class="tpl-grid">
        <div class="tpl-card" v-for="t in templates" :key="t.name" @click="useTemplate(t)">
          <div class="tpl-thumb" :style="{ background: t.gradient }">
            <span class="tag">{{ t.tag }}</span>
            <span class="mark">{{ t.mark }}</span>
          </div>
          <div class="tpl-body">
            <h3>{{ t.name }}</h3>
            <p>{{ t.desc }}</p>
          </div>
        </div>
      </div>
    </section>

    <!-- 最近创作 -->
    <section class="section">
      <div class="section-head">
        <h2>最近创作</h2>
        <a @click="router.push('/project')">进入项目列表 →</a>
      </div>
      <div class="proj-grid" v-if="allProject.length">
        <div class="proj-card" v-for="p in allProject.slice(0, 3)" :key="p.id" @click="openProject(p.id)">
          <div class="proj-thumb" :style="{ background: p.artStyle ? `linear-gradient(145deg,var(--td-brand-color),var(--td-brand-color-7))` : 'linear-gradient(145deg,#6B5B44,#3A3125)' }">
            <span class="badge">{{ p.projectType === 'novel' ? '小说改编' : '剧本创作' }}</span>
            <span class="pmark">{{ p.name.slice(0, 1) }}</span>
          </div>
          <div class="proj-body">
            <h3>{{ p.name }}</h3>
            <div class="proj-meta">
              <span class="chip" v-if="p.artStyle">{{ p.artStyle }}</span>
              <span class="chip" v-if="p.videoRatio">{{ p.videoRatio }}</span>
            </div>
            <div class="proj-time">更新于 {{ dayjs(p.updatedAt || p.createTime).format('YYYY/MM/DD') }}</div>
          </div>
        </div>
      </div>
      <div class="empty" v-else>还没有项目，用上面一句话或一套模板开始第一段文旅漫剧。</div>
    </section>

    <!-- 可复用 Skill -->
    <section class="section">
      <div class="section-head">
        <h2>可复用 Skill</h2>
        <a @click="openSkillManage">管理 Skill →</a>
      </div>
      <div class="skill-grid">
        <div class="skill-card" v-for="s in skills" :key="s.name">
          <span class="skill-seal" :style="{ background: s.color }">{{ s.seal }}</span>
          <div>
            <h3>{{ s.name }}</h3>
            <p>{{ s.desc }}</p>
          </div>
        </div>
      </div>
    </section>
  </div>

  <projectDialog v-model="dialogShow" :projectData="editProjectData" @add="addProjectFn" />
</template>

<script setup lang="ts">
import projectDialog from "@/views/project/components/projectDialog.vue";
import projectStore from "@/stores/project";
import dayjs from "dayjs";
import axios from "@/utils/axios";

const router = useRouter();
const { allProject, project } = storeToRefs(projectStore());

const promptText = ref("");
const mode = ref<"agent" | "precise" | "template">("agent");
const dialogShow = ref(false);
const editProjectData = ref<any>(null);

const templates = [
  { tag: "名人类", mark: "阳", name: "《阳明先生》", desc: "十二集生平漫剧，从出生到立德立功立言，研学与文旅双用。", gradient: "linear-gradient(145deg,#C1442F,#8F2F21)" },
  { tag: "文物类", mark: "河", name: "《七千年前的河姆渡》", desc: "遗址场景复原动画，让文物从展柜「活」成生活。", gradient: "linear-gradient(145deg,#4A8A74,#2A5C4F)" },
  { tag: "红色类", mark: "山", name: "《浙东小延安 · 四明山》", desc: "红色教育漫剧，宁波方言讲述在地温度。", gradient: "linear-gradient(145deg,#8A3B2C,#54241A)" },
  { tag: "非遗类", mark: "戏", name: "《姚剧与梁弄大糕》", desc: "非遗申报片 + 传播版，一次成系列多版本。", gradient: "linear-gradient(145deg,#C19A4B,#8A6320)" },
];

const skills = [
  { seal: "名", name: "国风 · 历史名人", desc: "生平漫剧范式 + 年龄阶段一致性 + 四句教口播。", color: "linear-gradient(145deg,#C1442F,#8F2F21)" },
  { seal: "物", name: "水墨 · 文物复原", desc: "遗址场景复原 + 文物「复活」叙事 + 博物馆解说。", color: "linear-gradient(145deg,#4A8A74,#2A5C4F)" },
  { seal: "红", name: "红色 · 宁波方言", desc: "红色教育叙事 + 方言旁白 + 民乐配乐，在地温度。", color: "linear-gradient(145deg,#C19A4B,#8A6320)" },
];

function getAllProject() {
  axios.post("/project/getProject").then(({ data }) => {
    allProject.value = data || [];
  });
}

function startCreate() {
  editProjectData.value = null;
  dialogShow.value = true;
}

function useTemplate(t: (typeof templates)[number]) {
  editProjectData.value = null;
  dialogShow.value = true;
  window.$message.info(`已选择模板「${t.name}」，请在弹窗中填写项目名并配置模型`);
}

function addProjectFn() {
  window.$message.success("项目已创建");
  getAllProject();
}

function openProject(id: string) {
  const item = allProject.value.find((p) => p.id === id);
  if (!item) return;
  project.value = item;
  router.push(item.projectType === "novel" ? "/novel" : "/script");
}

function openSkillManage() {
  router.push("/scriptAgent");
}

onMounted(getAllProject);
</script>

<style lang="scss" scoped>
$kai: "Kaiti SC", "STKaiti", "KaiTi", "楷体", "STSong", serif;

.console {
  max-width: 1180px;
  margin: 0 auto;
  padding: 20px 4px 48px;
}

.hero {
  padding-top: 24px;
}
.kicker {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 3px;
  color: var(--td-brand-color);
  &::before {
    content: "";
    width: 20px;
    height: 2px;
    background: linear-gradient(90deg, var(--td-brand-color), transparent);
  }
}
.greeting {
  font-family: $kai;
  font-size: 44px;
  font-weight: 700;
  margin: 14px 0 4px;
  letter-spacing: 2px;
  color: var(--td-text-color-primary);
}
.motto {
  font-family: $kai;
  font-size: 17px;
  letter-spacing: 4px;
  color: var(--td-text-color-secondary);
}

/* 命令台 */
.console-bar {
  margin-top: 28px;
  background: var(--td-bg-color-container);
  border: 1px solid var(--td-border-level-1-color);
  border-radius: 14px;
  box-shadow: 0 18px 50px rgba(60, 45, 25, 0.08);
  padding: 22px 24px;
}
.cmd-input {
  display: flex;
  align-items: center;
  gap: 12px;
  border: 1px solid var(--td-border-level-1-color);
  border-radius: 11px;
  padding: 6px 7px 6px 18px;
  background: var(--td-bg-color-page);
}
.cmd-input :deep(.t-input) {
  flex: 1;
  border: none;
  box-shadow: none;
  background: transparent;
}
.drop {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 9px 14px;
  border: 1px dashed var(--td-border-level-2-color);
  border-radius: 8px;
  font-size: 13px;
  color: var(--td-text-color-secondary);
  cursor: pointer;
  white-space: nowrap;
  &:hover {
    border-color: var(--td-brand-color);
    color: var(--td-brand-color);
  }
}
.drop-icon {
  font-size: 17px;
}

.cmd-modes {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-top: 16px;
  flex-wrap: wrap;
}
.modes {
  display: flex;
  gap: 4px;
  background: var(--td-bg-color-page);
  border-radius: 10px;
  padding: 4px;
  button {
    padding: 7px 15px;
    border: none;
    border-radius: 8px;
    background: transparent;
    cursor: pointer;
    font-size: 14px;
    font-weight: 600;
    color: var(--td-text-color-secondary);
    &.on {
      background: var(--td-bg-color-container);
      color: var(--td-brand-color);
      box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
    }
  }
}
.hint {
  font-size: 13px;
  color: var(--td-text-color-placeholder);
}
.actions {
  margin-left: auto;
  display: flex;
  gap: 10px;
  align-items: center;
}
.save-skill {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: var(--td-text-color-secondary);
  padding: 8px 13px;
  border: 1px solid var(--td-border-level-1-color);
  border-radius: 8px;
  cursor: pointer;
  background: transparent;
  &:hover {
    border-color: var(--td-brand-color);
    color: var(--td-brand-color);
  }
}
.ss-icon {
  font-size: 15px;
}
.start-icon {
  font-size: 16px;
}

/* section */
.section {
  margin-top: 40px;
}
.section-head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 16px;
  h2 {
    font-family: $kai;
    font-size: 22px;
    font-weight: 700;
    color: var(--td-text-color-primary);
    &::before {
      content: "◆";
      color: var(--td-brand-color);
      font-size: 12px;
      margin-right: 8px;
      vertical-align: 2px;
    }
  }
  a {
    font-size: 13px;
    color: var(--td-text-color-secondary);
    cursor: pointer;
    &:hover {
      color: var(--td-brand-color);
    }
  }
}

/* 模板 */
.tpl-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}
.tpl-card {
  background: var(--td-bg-color-container);
  border: 1px solid var(--td-border-level-1-color);
  border-radius: 13px;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.18s ease, box-shadow 0.18s ease;
  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 16px 40px rgba(90, 60, 30, 0.16);
  }
}
.tpl-thumb {
  height: 110px;
  position: relative;
  display: grid;
  place-items: center;
}
.tag {
  position: absolute;
  top: 11px;
  left: 11px;
  padding: 2px 9px;
  border-radius: 6px;
  font-size: 11px;
  font-weight: 600;
  background: rgba(255, 255, 255, 0.92);
  color: #333;
}
.mark {
  width: 42px;
  height: 42px;
  border: 1.5px solid rgba(255, 255, 255, 0.8);
  display: grid;
  place-items: center;
  font-family: $kai;
  font-size: 22px;
  color: #fff;
  font-weight: 700;
  border-radius: 6px;
}
.tpl-body {
  padding: 14px 16px 16px;
  h3 {
    font-family: $kai;
    font-size: 16px;
    font-weight: 700;
    color: var(--td-text-color-primary);
    margin-bottom: 5px;
  }
  p {
    font-size: 12.5px;
    color: var(--td-text-color-secondary);
    line-height: 1.6;
  }
}

/* 项目 */
.proj-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}
.proj-card {
  background: var(--td-bg-color-container);
  border: 1px solid var(--td-border-level-1-color);
  border-radius: 13px;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.18s ease, box-shadow 0.18s ease;
  &:hover {
    transform: translateY(-3px);
    box-shadow: 0 14px 34px rgba(60, 45, 25, 0.13);
  }
}
.proj-thumb {
  height: 120px;
  position: relative;
  display: grid;
  place-items: center;
}
.badge {
  position: absolute;
  top: 11px;
  left: 11px;
  padding: 2px 9px;
  border-radius: 6px;
  font-size: 11px;
  font-weight: 600;
  background: rgba(255, 255, 255, 0.92);
  color: #333;
}
.pmark {
  font-family: $kai;
  font-size: 32px;
  color: rgba(255, 255, 255, 0.88);
  font-weight: 700;
}
.proj-body {
  padding: 14px 16px 16px;
  h3 {
    font-family: $kai;
    font-size: 16px;
    font-weight: 700;
    color: var(--td-text-color-primary);
    margin-bottom: 7px;
  }
}
.proj-meta {
  display: flex;
  gap: 7px;
  flex-wrap: wrap;
  margin-bottom: 8px;
}
.chip {
  font-size: 11px;
  padding: 2px 8px;
  border-radius: 6px;
  background: var(--td-bg-color-page);
  color: var(--td-text-color-secondary);
}
.proj-time {
  font-size: 12px;
  color: var(--td-text-color-placeholder);
}
.empty {
  padding: 40px;
  text-align: center;
  color: var(--td-text-color-placeholder);
  font-size: 14px;
}

/* skill */
.skill-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}
.skill-card {
  background: var(--td-bg-color-container);
  border: 1px solid var(--td-border-level-1-color);
  border-radius: 13px;
  padding: 18px 20px;
  display: flex;
  gap: 14px;
  align-items: flex-start;
  cursor: pointer;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  &:hover {
    border-color: var(--td-brand-color);
    box-shadow: 0 10px 26px rgba(90, 60, 30, 0.12);
  }
  h3 {
    font-family: $kai;
    font-size: 15px;
    font-weight: 700;
    color: var(--td-text-color-primary);
    margin-bottom: 4px;
  }
  p {
    font-size: 12px;
    color: var(--td-text-color-secondary);
    line-height: 1.6;
  }
}
.skill-seal {
  width: 40px;
  height: 40px;
  flex: 0 0 40px;
  border-radius: 8px;
  display: grid;
  place-items: center;
  font-family: $kai;
  font-size: 18px;
  color: #fff;
  font-weight: 700;
}

@media (max-width: 1100px) {
  .tpl-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  .proj-grid,
  .skill-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
