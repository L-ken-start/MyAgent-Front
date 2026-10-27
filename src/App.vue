<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'

const appearances = [
  { value: 'light', label: '浅色' },
  { value: 'dark', label: '深色' },
  { value: 'system', label: '跟随系统' },
]

const accents = [
  { value: 'moss', label: '苔绿' },
  { value: 'blue', label: '深海蓝' },
  { value: 'violet', label: '鸢尾紫' },
  { value: 'amber', label: '琥珀' },
  { value: 'rose', label: '绛红' },
]

const conversations = [
  { title: '设计 Agent 工作台界面', date: '刚刚', active: true },
  { title: '接入 DeepSeek API', date: '昨天' },
  { title: '梳理工具调用流程', date: '3 天' },
]

const appearance = ref(localStorage.getItem('myagent-appearance') || 'light')
const accent = ref(localStorage.getItem('myagent-accent') || 'blue')
const themeOpen = ref(false)
const mobileSidebarOpen = ref(false)
const activeTab = ref('chat')
const prompt = ref('')
const mainPromptInput = ref(null)
const conversationRoot = ref(null)
const mainSelection = ref('')
const sideChatOpen = ref(false)
const sideChatContext = ref('')
const sideChatPrompt = ref('')
const sideMessages = ref([])
const selectionToolbar = ref({ visible: false, text: '', x: 0, y: 0 })

const systemTheme = window.matchMedia('(prefers-color-scheme: dark)')

const resolvedTheme = computed(() => {
  if (appearance.value !== 'system') return appearance.value
  return systemTheme.matches ? 'dark' : 'light'
})

function applyTheme() {
  document.documentElement.dataset.theme = resolvedTheme.value
  document.documentElement.dataset.accent = accent.value
  document.documentElement.style.colorScheme = resolvedTheme.value
}

function handleSystemTheme() {
  if (appearance.value === 'system') applyTheme()
}

function chooseAppearance(value) {
  appearance.value = value
}

function chooseAccent(value) {
  accent.value = value
}

function submitPrompt() {
  if (!prompt.value.trim()) return
  prompt.value = ''
  mainSelection.value = ''
}

function hideSelectionToolbar() {
  selectionToolbar.value.visible = false
}

function clearBrowserSelection() {
  window.getSelection()?.removeAllRanges()
}

function captureSelection() {
  requestAnimationFrame(() => {
    const selection = window.getSelection()
    const text = selection?.toString().replace(/\s+/g, ' ').trim()

    if (!text || !selection.rangeCount || !conversationRoot.value?.contains(selection.anchorNode) || !conversationRoot.value?.contains(selection.focusNode)) {
      hideSelectionToolbar()
      return
    }

    const rect = selection.getRangeAt(0).getBoundingClientRect()
    const halfToolbar = 138
    selectionToolbar.value = {
      visible: true,
      text,
      x: Math.min(window.innerWidth - halfToolbar - 12, Math.max(halfToolbar + 12, rect.left + rect.width / 2)),
      y: Math.max(54, rect.top - 10),
    }
  })
}

async function addSelectionToConversation() {
  mainSelection.value = selectionToolbar.value.text
  hideSelectionToolbar()
  clearBrowserSelection()
  await nextTick()
  mainPromptInput.value?.focus()
}

function askSelectionInSideChat() {
  sideChatContext.value = selectionToolbar.value.text
  sideMessages.value = []
  sideChatPrompt.value = ''
  sideChatOpen.value = true
  hideSelectionToolbar()
  clearBrowserSelection()
}

function openSideChat() {
  sideChatOpen.value = !sideChatOpen.value
  hideSelectionToolbar()
}

function useSideSuggestion(text) {
  sideChatPrompt.value = text
}

function submitSidePrompt() {
  const question = sideChatPrompt.value.trim()
  if (!question) return

  const id = Date.now()
  sideMessages.value.push(
    { id, role: 'user', text: question },
    { id: id + 1, role: 'assistant', text: '侧边聊天界面已经就绪。接入 DeepSeek 后，MyAgent 会结合这段引用和当前工作区上下文回答这个问题。' },
  )
  sideChatPrompt.value = ''
}

function closeTransientUi() {
  if (selectionToolbar.value.visible) return hideSelectionToolbar()
  if (themeOpen.value) return (themeOpen.value = false)
  sideChatOpen.value = false
}

watch([appearance, accent], () => {
  localStorage.setItem('myagent-appearance', appearance.value)
  localStorage.setItem('myagent-accent', accent.value)
  applyTheme()
}, { immediate: true })

onMounted(() => {
  systemTheme.addEventListener('change', handleSystemTheme)
  document.addEventListener('mouseup', captureSelection)
  window.addEventListener('resize', hideSelectionToolbar)
})

onBeforeUnmount(() => {
  systemTheme.removeEventListener('change', handleSystemTheme)
  document.removeEventListener('mouseup', captureSelection)
  window.removeEventListener('resize', hideSelectionToolbar)
})
</script>

<template>
  <div class="app-shell" @keydown.esc="closeTransientUi">
    <svg class="svg-sprite" aria-hidden="true">
      <symbol id="icon-plus" viewBox="0 0 24 24"><path d="M12 5v14M5 12h14" /></symbol>
      <symbol id="icon-chat" viewBox="0 0 24 24"><path d="M20 15a4 4 0 0 1-4 4H8l-5 3V7a4 4 0 0 1 4-4h9a4 4 0 0 1 4 4v8Z" /></symbol>
      <symbol id="icon-tool" viewBox="0 0 24 24"><path d="m14.7 6.3 3-3a4 4 0 0 1-5 5l-7.4 7.4a2.1 2.1 0 0 0 3 3l7.4-7.4a4 4 0 0 1 5-5l-3 3" /></symbol>
      <symbol id="icon-clock" viewBox="0 0 24 24"><circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 2"/></symbol>
      <symbol id="icon-folder" viewBox="0 0 24 24"><path d="M3 6.5h6l2 2h10v9.5a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6.5Z" /></symbol>
      <symbol id="icon-search" viewBox="0 0 24 24"><circle cx="11" cy="11" r="7"/><path d="m20 20-4-4"/></symbol>
      <symbol id="icon-settings" viewBox="0 0 24 24"><circle cx="12" cy="12" r="3"/><path d="M19.4 15a1.7 1.7 0 0 0 .3 1.9l.1.1-2.8 2.8-.1-.1a1.7 1.7 0 0 0-1.9-.3 1.7 1.7 0 0 0-1 1.6v.2h-4V21a1.7 1.7 0 0 0-1-1.6 1.7 1.7 0 0 0-1.9.3l-.1.1L4.2 17l.1-.1a1.7 1.7 0 0 0 .3-1.9A1.7 1.7 0 0 0 3 14H2.8v-4H3a1.7 1.7 0 0 0 1.6-1 1.7 1.7 0 0 0-.3-1.9L4.2 7 7 4.2l.1.1A1.7 1.7 0 0 0 9 4.6a1.7 1.7 0 0 0 1-1.6v-.2h4V3a1.7 1.7 0 0 0 1 1.6 1.7 1.7 0 0 0 1.9-.3l.1-.1L19.8 7l-.1.1a1.7 1.7 0 0 0-.3 1.9 1.7 1.7 0 0 0 1.6 1h.2v4H21a1.7 1.7 0 0 0-1.6 1Z"/></symbol>
      <symbol id="icon-panel" viewBox="0 0 24 24"><rect x="3" y="4" width="18" height="16" rx="2"/><path d="M8 4v16"/></symbol>
      <symbol id="icon-palette" viewBox="0 0 24 24"><path d="M12 3a9 9 0 0 0 0 18h1.3a1.7 1.7 0 0 0 1.2-2.9 1.7 1.7 0 0 1 1.2-2.9H17A4 4 0 0 0 21 11c0-4.4-4-8-9-8Z"/><circle cx="7.5" cy="11" r=".7" fill="currentColor"/><circle cx="10" cy="7" r=".7" fill="currentColor"/><circle cx="15" cy="7.5" r=".7" fill="currentColor"/></symbol>
      <symbol id="icon-more" viewBox="0 0 24 24"><circle cx="5" cy="12" r="1" fill="currentColor"/><circle cx="12" cy="12" r="1" fill="currentColor"/><circle cx="19" cy="12" r="1" fill="currentColor"/></symbol>
      <symbol id="icon-check" viewBox="0 0 24 24"><path d="m5 12 4 4L19 6"/></symbol>
      <symbol id="icon-file" viewBox="0 0 24 24"><path d="M6 3h8l4 4v14H6V3Z"/><path d="M14 3v5h5"/></symbol>
      <symbol id="icon-chevron" viewBox="0 0 24 24"><path d="m8 10 4 4 4-4"/></symbol>
      <symbol id="icon-paperclip" viewBox="0 0 24 24"><path d="m8.5 12.5 5.8-5.8a3 3 0 1 1 4.2 4.2l-7.9 7.9a5 5 0 0 1-7.1-7.1l8.2-8.2"/></symbol>
      <symbol id="icon-arrow-up" viewBox="0 0 24 24"><path d="m6 11 6-6 6 6M12 5v14"/></symbol>
      <symbol id="icon-code" viewBox="0 0 24 24"><path d="m8 8-4 4 4 4M16 8l4 4-4 4M14 5l-4 14"/></symbol>
      <symbol id="icon-menu" viewBox="0 0 24 24"><path d="M4 7h16M4 12h16M4 17h16"/></symbol>
      <symbol id="icon-spark" viewBox="0 0 24 24"><path d="M12 3a7 7 0 0 0 7 7 7 7 0 0 0-7 7 7 7 0 0 0-7-7 7 7 0 0 0 7-7Z"/><path d="M19 16a3 3 0 0 0 3 3 3 3 0 0 0-3 3 3 3 0 0 0-3-3 3 3 0 0 0 3-3Z"/></symbol>
      <symbol id="icon-x" viewBox="0 0 24 24"><path d="m6 6 12 12M18 6 6 18"/></symbol>
      <symbol id="icon-quote" viewBox="0 0 24 24"><path d="M10 11H5a5 5 0 0 1 5-5v11H5v-3M21 11h-5a5 5 0 0 1 5-5v11h-5v-3"/></symbol>
      <symbol id="icon-copy" viewBox="0 0 24 24"><rect x="8" y="8" width="11" height="11" rx="2"/><path d="M16 8V5a2 2 0 0 0-2-2H5a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h3"/></symbol>
      <symbol id="icon-thumbs-up" viewBox="0 0 24 24"><path d="M7 10v10H4a2 2 0 0 1-2-2v-6a2 2 0 0 1 2-2h3ZM7 20h9.5a2 2 0 0 0 2-1.6l1.3-6A2 2 0 0 0 17.8 10H14l.6-3a3 3 0 0 0-3-3L7 10"/></symbol>
      <symbol id="icon-volume" viewBox="0 0 24 24"><path d="M11 5 6 9H3v6h3l5 4V5ZM15 9a5 5 0 0 1 0 6M18 6a9 9 0 0 1 0 12"/></symbol>
      <symbol id="icon-undo" viewBox="0 0 24 24"><path d="m9 7-4 4 4 4M5 11h8a6 6 0 0 1 6 6"/></symbol>
    </svg>

    <aside class="sidebar" :class="{ 'sidebar--open': mobileSidebarOpen }" aria-label="主导航">
      <div class="brand-row">
        <a class="brand" href="#" aria-label="MyAgent 首页">
          <span class="brand-mark"><span>M</span></span>
          <span>MyAgent</span>
        </a>
        <button class="icon-button sidebar-close" aria-label="关闭侧边栏" @click="mobileSidebarOpen = false">
          <svg><use href="#icon-panel" /></svg>
        </button>
      </div>

      <button class="new-chat-button">
        <svg><use href="#icon-plus" /></svg>
        新建会话
        <span>Ctrl N</span>
      </button>

      <nav class="primary-nav" aria-label="功能导航">
        <a href="#" class="nav-item"><svg><use href="#icon-tool" /></svg><span>工具</span><small>6</small></a>
        <a href="#" class="nav-item"><svg><use href="#icon-clock" /></svg><span>自动化任务</span><small>2</small></a>
      </nav>

      <div class="workspace-heading">
        <span>工作区</span>
        <div>
          <button class="icon-button" aria-label="搜索"><svg><use href="#icon-search" /></svg></button>
          <button class="icon-button" aria-label="工作区设置"><svg><use href="#icon-settings" /></svg></button>
        </div>
      </div>

      <div class="workspace">
        <button class="workspace-name">
          <svg><use href="#icon-folder" /></svg>
          <span>MyAgent</span>
          <svg class="chevron"><use href="#icon-chevron" /></svg>
        </button>
        <div class="conversation-list">
          <button v-for="item in conversations" :key="item.title" class="conversation-item" :class="{ active: item.active }">
            <span>{{ item.title }}</span>
            <small>{{ item.date }}</small>
          </button>
        </div>
      </div>

      <div class="sidebar-footer">
        <button class="user-menu">
          <span class="avatar">L</span>
          <span><strong>本地工作区</strong><small>所有更改保存在此设备</small></span>
          <svg><use href="#icon-more" /></svg>
        </button>
      </div>
    </aside>

    <button v-if="mobileSidebarOpen" class="sidebar-scrim" aria-label="关闭侧边栏" @click="mobileSidebarOpen = false" />

    <main class="main-panel">
      <header class="topbar">
        <div class="title-group">
          <button class="icon-button mobile-menu" aria-label="打开侧边栏" @click="mobileSidebarOpen = true"><svg><use href="#icon-menu" /></svg></button>
          <div>
            <div class="title-line">
              <h1>设计 Agent 工作台界面</h1>
              <span class="mode-label"><svg><use href="#icon-code" /></svg>工作区模式</span>
            </div>
            <p>D:\front\MyAgent</p>
          </div>
        </div>

        <div class="topbar-actions">
          <div class="theme-control">
            <button class="icon-button theme-button" :aria-expanded="themeOpen" aria-haspopup="dialog" aria-label="更改主题" @click="themeOpen = !themeOpen">
              <svg><use href="#icon-palette" /></svg>
              <span class="accent-dot" />
            </button>
            <div v-if="themeOpen" class="theme-popover" role="dialog" aria-label="主题设置">
              <div class="popover-header">
                <div><strong>界面主题</strong><span>设置会自动保存在本机</span></div>
                <button class="icon-button" aria-label="关闭主题设置" @click="themeOpen = false"><svg><use href="#icon-more" /></svg></button>
              </div>
              <span class="field-label">外观</span>
              <div class="appearance-options">
                <button v-for="item in appearances" :key="item.value" :class="{ selected: appearance === item.value }" @click="chooseAppearance(item.value)">
                  <span class="appearance-preview" :class="`appearance-preview--${item.value}`"><i /><i /></span>
                  <span>{{ item.label }}</span>
                  <svg v-if="appearance === item.value"><use href="#icon-check" /></svg>
                </button>
              </div>
              <span class="field-label">强调色</span>
              <div class="accent-options">
                <button v-for="item in accents" :key="item.value" :class="[`accent-${item.value}`, { selected: accent === item.value }]" :aria-label="item.label" :title="item.label" @click="chooseAccent(item.value)">
                  <span /><svg v-if="accent === item.value"><use href="#icon-check" /></svg>
                </button>
              </div>
            </div>
          </div>
          <button class="icon-button" aria-label="更多操作"><svg><use href="#icon-more" /></svg></button>
          <button class="icon-button desktop-panel" :class="{ active: sideChatOpen }" :aria-pressed="sideChatOpen" aria-label="切换侧边聊天" @click="openSideChat"><svg><use href="#icon-panel" /></svg></button>
        </div>
      </header>

      <div class="tabs" role="tablist" aria-label="任务视图">
        <button :class="{ active: activeTab === 'chat' }" role="tab" :aria-selected="activeTab === 'chat'" @click="activeTab = 'chat'">对话</button>
        <button :class="{ active: activeTab === 'trace' }" role="tab" :aria-selected="activeTab === 'trace'" @click="activeTab = 'trace'">轨迹 <span>4</span></button>
      </div>

      <section v-if="activeTab === 'chat'" ref="conversationRoot" class="conversation-view" aria-label="对话内容" @keyup="captureSelection" @scroll="hideSelectionToolbar">
        <div class="conversation-content">
          <article class="message message--user">
            <div class="user-bubble">中间消息区域的视觉效果也和 Codex 对齐。</div>
          </article>

          <article class="message message--assistant">
            <details class="execution-details">
              <summary><span>用时 12.8 秒</span><svg><use href="#icon-chevron" /></svg></summary>
              <div class="tool-trace" aria-label="已完成的操作">
                <div><span class="tool-status"><svg><use href="#icon-check" /></svg></span><span><strong>分析 Codex 消息层级</strong><small>用户胶囊 · 文档式回复 · 文件变更</small></span><time>0.8s</time></div>
                <div><span class="tool-status"><svg><use href="#icon-check" /></svg></span><span><strong>调整对话结构</strong><small>App.vue · styles.css</small></span><time>9.4s</time></div>
              </div>
            </details>

            <div class="assistant-copy">
              <p><strong>已经按 Codex 的消息层级重新整理好了。</strong></p>
              <p>现在中间对话区域会：</p>
              <ul>
                <li>把用户消息显示成右侧蓝色胶囊，减少不必要的身份信息。</li>
                <li>把执行过程收进“用时”折叠栏，让回答正文成为阅读主线。</li>
                <li>助手回复使用连续文档流，保持清晰的段落、列表和代码引用。</li>
                <li>文件变更集中在统一面板里，操作按钮降到次要层级。</li>
                <li>保留选中文本、添加引用和侧边聊天交互。</li>
              </ul>
              <p>输入框也改成了更接近 Codex 的中性填充状态，聚焦时不会再出现抢眼的蓝色外框。</p>
            </div>

            <div class="change-summary">
              <div class="change-summary__header">
                <span><span class="change-icon"><svg><use href="#icon-file" /></svg></span><strong>已编辑 4 个文件</strong><small><b>+126</b> <em>-38</em></small></span>
                <div><button><svg><use href="#icon-undo" /></svg>撤销</button><button class="review-button">查看变更</button></div>
              </div>
              <div><code>D:/front/MyAgent/src/App.vue</code><span><b>+64</b> <em>-21</em></span></div>
              <div><code>D:/front/MyAgent/src/styles.css</code><span><b>+48</b> <em>-15</em></span></div>
              <div><code>D:/front/MyAgent/DESIGN.md</code><span><b>+14</b> <em>-2</em></span></div>
              <button class="change-summary__more">再显示 1 个文件 <svg><use href="#icon-chevron" /></svg></button>
            </div>

            <div class="message-actions">
              <button aria-label="复制回复" title="复制"><svg><use href="#icon-copy" /></svg></button>
              <button aria-label="有帮助" title="有帮助"><svg><use href="#icon-thumbs-up" /></svg></button>
              <button aria-label="朗读回复" title="朗读"><svg><use href="#icon-volume" /></svg></button>
              <button aria-label="更多回复操作" title="更多"><svg><use href="#icon-more" /></svg></button>
            </div>
          </article>
        </div>
      </section>

      <section v-else class="trace-view" aria-label="执行轨迹">
        <div class="trace-header"><div><h2>执行轨迹</h2><p>查看本次任务的上下文读取与工具调用。</p></div><span>4 个步骤 · 12.8s</span></div>
        <ol class="trace-list">
          <li><span><svg><use href="#icon-check" /></svg></span><div><strong>读取项目说明</strong><p>确认产品定位、设计约束和技术栈。</p><code>PRODUCT.md · DESIGN.md</code></div><time>0.2s</time></li>
          <li><span><svg><use href="#icon-check" /></svg></span><div><strong>分析参考界面</strong><p>提取结构、密度和交互层级，不复制品牌资产。</p><code>image.inspect</code></div><time>0.8s</time></li>
          <li><span><svg><use href="#icon-check" /></svg></span><div><strong>生成 Vue 界面</strong><p>创建工作区导航、对话画布、输入区与主题系统。</p><code>src/App.vue · src/styles.css</code></div><time>9.6s</time></li>
          <li class="current"><span><i /></span><div><strong>等待视觉确认</strong><p>界面已准备好，可以继续调整布局与风格。</p></div><time>现在</time></li>
        </ol>
      </section>

      <div class="composer-wrap">
        <form class="composer" @submit.prevent="submitPrompt">
          <label class="sr-only" for="prompt">向 MyAgent 提问</label>
          <div v-if="mainSelection" class="composer-selection">
            <svg><use href="#icon-quote" /></svg>
            <span>{{ mainSelection }}</span>
            <button type="button" aria-label="移除引用" @click="mainSelection = ''"><svg><use href="#icon-x" /></svg></button>
          </div>
          <textarea id="prompt" ref="mainPromptInput" v-model="prompt" rows="1" placeholder="发送消息、描述任务，或使用 @ 引用文件…" @keydown.ctrl.enter.prevent="submitPrompt"></textarea>
          <div class="composer-toolbar">
            <div class="composer-tools">
              <button type="button" class="icon-button add-button" aria-label="添加附件"><svg><use href="#icon-plus" /></svg></button>
              <button type="button" class="context-mode"><svg><use href="#icon-code" /></svg>工作区内修改<svg class="chevron"><use href="#icon-chevron" /></svg></button>
            </div>
            <div class="composer-submit">
              <button type="button" class="model-select">DeepSeek-V3.2<svg><use href="#icon-chevron" /></svg></button>
              <button type="submit" class="send-button" :disabled="!prompt.trim()" aria-label="发送消息"><svg><use href="#icon-arrow-up" /></svg></button>
            </div>
          </div>
        </form>
        <div class="composer-status"><span><i class="online-dot" /> 本地工作区已连接</span><span>上下文 18%</span><span>Ctrl + Enter 发送</span></div>
      </div>
    </main>

    <Transition name="side-chat">
      <aside v-if="sideChatOpen" class="side-chat" aria-label="侧边聊天">
        <header class="side-chat__header">
          <div>
            <strong>侧边聊天</strong>
            <span v-if="sideChatContext">基于所选内容</span>
            <span v-else>独立上下文</span>
          </div>
          <button class="icon-button" aria-label="关闭侧边聊天" @click="sideChatOpen = false"><svg><use href="#icon-x" /></svg></button>
        </header>

        <div class="side-chat__body">
          <div v-if="sideChatContext" class="side-context">
            <div><svg><use href="#icon-quote" /></svg><span>所选内容</span></div>
            <p>{{ sideChatContext }}</p>
            <button aria-label="移除所选内容" @click="sideChatContext = ''"><svg><use href="#icon-x" /></svg></button>
          </div>

          <div v-if="!sideMessages.length" class="side-chat__empty">
            <span class="side-agent-mark"><svg><use href="#icon-spark" /></svg></span>
            <h2>{{ sideChatContext ? '想了解这段内容的什么？' : '在这里开启独立讨论' }}</h2>
            <p>{{ sideChatContext ? 'MyAgent 会把所选文字作为引用上下文，不打断主对话。' : '从主对话中选择文字，即可带着引用在这里继续提问。' }}</p>
            <div v-if="sideChatContext" class="side-suggestions">
              <button @click="useSideSuggestion('解释一下这段内容')">解释这段内容</button>
              <button @click="useSideSuggestion('这段内容有什么潜在问题？')">找出潜在问题</button>
              <button @click="useSideSuggestion('给出一个更清晰的改写版本')">给出改写建议</button>
            </div>
          </div>

          <div v-else class="side-messages" aria-live="polite">
            <article v-for="message in sideMessages" :key="message.id" :class="`side-message side-message--${message.role}`">
              <div>
                <span v-if="message.role === 'assistant'" class="side-agent-mark"><svg><use href="#icon-spark" /></svg></span>
                <span v-else class="avatar avatar--small">L</span>
                <strong>{{ message.role === 'assistant' ? 'MyAgent' : '你' }}</strong>
              </div>
              <p>{{ message.text }}</p>
            </article>
          </div>
        </div>

        <form class="side-composer" @submit.prevent="submitSidePrompt">
          <label class="sr-only" for="side-prompt">在侧边聊天中提问</label>
          <textarea id="side-prompt" v-model="sideChatPrompt" rows="2" placeholder="针对所选内容提问…" @keydown.enter.exact.prevent="submitSidePrompt"></textarea>
          <div>
            <span>{{ sideChatContext ? '已附加 1 段引用' : '未附加引用' }}</span>
            <button type="submit" class="send-button" :disabled="!sideChatPrompt.trim()" aria-label="发送侧边聊天消息"><svg><use href="#icon-arrow-up" /></svg></button>
          </div>
        </form>
      </aside>
    </Transition>

    <Teleport to="body">
      <div
        v-if="selectionToolbar.visible"
        class="selection-toolbar"
        role="toolbar"
        aria-label="所选文本操作"
        :style="{ left: `${selectionToolbar.x}px`, top: `${selectionToolbar.y}px` }"
        @mousedown.prevent
      >
        <button @click="addSelectionToConversation">添加到对话</button>
        <button @click="askSelectionInSideChat">在侧边聊天中提问</button>
      </div>
    </Teleport>
  </div>
</template>
