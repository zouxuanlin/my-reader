<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import { novel } from '../data/novel.js'

// 当前章节索引
const currentChapterIndex = ref(0)

// 字体大小
const fontSize = ref(18)

// 背景主题：light, beige, sepia, dark
const theme = ref('light')

// 是否显示设置面板
const showSettings = ref(false)

// 是否显示章节列表
const showChapterList = ref(false)

// 阅读进度
const scrollProgress = ref(0)

// 获取当前章节
const currentChapter = computed(() => {
  return novel.chapters[currentChapterIndex.value]
})

// 主题类名
const themeClass = computed(() => {
  const themeMap = {
    light: 'bg-white text-gray-800',
    beige: 'bg-beige text-gray-800',
    sepia: 'bg-sepia text-gray-800',
    dark: 'bg-dark text-gray-200'
  }
  return themeMap[theme.value]
})

// 上一章
const prevChapter = () => {
  if (currentChapterIndex.value > 0) {
    currentChapterIndex.value--
    scrollToTop()
    saveProgress()
  }
}

// 下一章
const nextChapter = () => {
  if (currentChapterIndex.value < novel.chapters.length - 1) {
    currentChapterIndex.value++
    scrollToTop()
    saveProgress()
  }
}

// 滚动到顶部
const scrollToTop = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// 保存进度
const saveProgress = () => {
  const progress = {
    chapterIndex: currentChapterIndex.value,
    fontSize: fontSize.value,
    theme: theme.value,
    scrollTop: window.scrollY
  }
  localStorage.setItem('novelProgress', JSON.stringify(progress))
}

// 加载进度
const loadProgress = () => {
  const saved = localStorage.getItem('novelProgress')
  if (saved) {
    const progress = JSON.parse(saved)
    currentChapterIndex.value = progress.chapterIndex || 0
    fontSize.value = progress.fontSize || 18
    theme.value = progress.theme || 'light'
    setTimeout(() => {
      window.scrollTo({ top: progress.scrollTop || 0, behavior: 'smooth' })
    }, 100)
  }
}

// 监听滚动计算进度
const handleScroll = () => {
  const scrollTop = window.scrollY
  const scrollHeight = document.documentElement.scrollHeight - window.innerHeight
  scrollProgress.value = Math.round((scrollTop / scrollHeight) * 100) || 0
}

// 键盘导航
const handleKeydown = (e) => {
  if (e.key === 'ArrowLeft') {
    prevChapter()
  } else if (e.key === 'ArrowRight') {
    nextChapter()
  } else if (e.key === 'Escape') {
    showSettings.value = false
    showChapterList.value = false
  }
}

onMounted(() => {
  loadProgress()
  window.addEventListener('scroll', handleScroll)
  document.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  document.removeEventListener('keydown', handleKeydown)
  saveProgress()
})

// 监听变化自动保存
watch([currentChapterIndex, fontSize, theme], () => {
  saveProgress()
})
</script>

<template>
  <div class="min-h-screen" :class="themeClass">
    <!-- 顶部进度条 -->
    <div class="fixed top-0 left-0 w-full h-1 bg-gray-200 z-50">
      <div
        class="h-full bg-blue-500 transition-all duration-200"
        :style="{ width: scrollProgress + '%' }"
      ></div>
    </div>

    <!-- 顶部导航栏 -->
    <header class="fixed top-1 left-0 right-0 z-40 bg-opacity-95 backdrop-blur-sm shadow-sm">
      <div class="max-w-4xl mx-auto px-4 py-3 flex items-center justify-between">
        <div class="flex items-center space-x-4">
          <button
            @click="showChapterList = true"
            class="p-2 rounded-lg hover:bg-black/5 transition-colors"
            title="目录"
          >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
            </svg>
          </button>
          <h1 class="text-lg font-semibold truncate">{{ novel.title }}</h1>
        </div>

        <div class="flex items-center space-x-2">
          <button
            @click="showSettings = !showSettings"
            class="p-2 rounded-lg hover:bg-black/5 transition-colors"
            title="设置"
          >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/>
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
            </svg>
          </button>
        </div>
      </div>
    </header>

    <!-- 设置面板 -->
    <transition
      enter-active-class="transition ease-out duration-200"
      enter-from-class="opacity-0 translate-y-2"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition ease-in duration-150"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-2"
    >
      <div
        v-if="showSettings"
        class="fixed top-16 right-4 z-50 w-72 bg-white dark:bg-gray-800 rounded-xl shadow-2xl border border-gray-200 dark:border-gray-700 overflow-hidden"
      >
        <div class="p-4">
          <h3 class="text-sm font-semibold text-gray-500 dark:text-gray-400 mb-3">背景主题</h3>
          <div class="flex space-x-2 mb-4">
            <button
              @click="theme = 'light'"
              class="w-12 h-12 rounded-lg border-2 transition-all"
              :class="theme === 'light' ? 'border-blue-500' : 'border-gray-300'"
              style="background-color: white;"
            ></button>
            <button
              @click="theme = 'beige'"
              class="w-12 h-12 rounded-lg border-2 transition-all"
              :class="theme === 'beige' ? 'border-blue-500' : 'border-gray-300'"
              style="background-color: #f5f0e6;"
            ></button>
            <button
              @click="theme = 'sepia'"
              class="w-12 h-12 rounded-lg border-2 transition-all"
              :class="theme === 'sepia' ? 'border-blue-500' : 'border-gray-300'"
              style="background-color: #f4ecd8;"
            ></button>
            <button
              @click="theme = 'dark'"
              class="w-12 h-12 rounded-lg border-2 transition-all"
              :class="theme === 'dark' ? 'border-blue-500' : 'border-gray-600'"
              style="background-color: #1a1a1a;"
            ></button>
          </div>

          <h3 class="text-sm font-semibold text-gray-500 dark:text-gray-400 mb-3">字体大小</h3>
          <div class="flex items-center space-x-3">
            <span class="text-sm text-gray-500">A</span>
            <input
              type="range"
              min="14"
              max="28"
              v-model="fontSize"
              class="flex-1 h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
            >
            <span class="text-xl text-gray-500">A</span>
          </div>
          <div class="text-center mt-2">
            <span class="text-sm text-gray-500">{{ fontSize }}px</span>
          </div>
        </div>
      </div>
    </transition>

    <!-- 章节列表侧边栏 -->
    <transition
      enter-active-class="transition ease-out duration-200"
      enter-from-class="opacity-0 -translate-x-full"
      enter-to-class="opacity-100 translate-x-0"
      leave-active-class="transition ease-in duration-150"
      leave-from-class="opacity-100 translate-x-0"
      leave-to-class="opacity-0 -translate-x-full"
    >
      <div
        v-if="showChapterList"
        class="fixed inset-0 z-50 flex"
      >
        <div
          class="absolute inset-0 bg-black/50"
          @click="showChapterList = false"
        ></div>
        <div
          class="relative w-80 max-w-[85vw] h-full bg-white dark:bg-gray-900 shadow-2xl overflow-y-auto"
        >
          <div class="sticky top-0 bg-white dark:bg-gray-900 p-4 border-b dark:border-gray-700">
            <div class="flex items-center justify-between">
              <h2 class="text-lg font-semibold">目录</h2>
              <button
                @click="showChapterList = false"
                class="p-1 rounded hover:bg-gray-100 dark:hover:bg-gray-800"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
                </svg>
              </button>
            </div>
            <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">{{ novel.title }} - {{ novel.author }}</p>
          </div>
          <div class="p-2">
            <div
              v-for="(chapter, index) in novel.chapters"
              :key="chapter.id"
              @click="currentChapterIndex = index; showChapterList = false; scrollToTop()"
              class="p-3 rounded-lg cursor-pointer transition-colors"
              :class="currentChapterIndex === index ? 'bg-blue-50 dark:bg-blue-900/30 text-blue-600 dark:text-blue-400' : 'hover:bg-gray-50 dark:hover:bg-gray-800'"
            >
              <div class="text-sm font-medium truncate">{{ chapter.title }}</div>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- 主要内容 -->
    <main class="pt-20 pb-24">
      <article class="max-w-3xl mx-auto px-6">
        <header class="mb-12 text-center">
          <h2 class="text-2xl md:text-3xl font-bold mb-4">{{ currentChapter.title }}</h2>
          <div class="flex items-center justify-center text-sm text-gray-500">
            <span>{{ novel.author }}</span>
            <span class="mx-2">•</span>
            <span>{{ currentChapterIndex + 1 }} / {{ novel.chapters.length }}</span>
          </div>
        </header>

        <div
          class="novel-content"
          :style="{ fontSize: fontSize + 'px' }"
        >
          <p v-for="(paragraph, idx) in currentChapter.content.split('\n\n')" :key="idx">
            {{ paragraph }}
          </p>
        </div>

        <!-- 章节导航 -->
        <nav class="mt-16 flex justify-between items-center">
          <button
            @click="prevChapter"
            :disabled="currentChapterIndex === 0"
            class="flex items-center px-6 py-3 rounded-lg border transition-all disabled:opacity-50 disabled:cursor-not-allowed hover:shadow-md"
          >
            <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/>
            </svg>
            上一章
          </button>
          <button
            @click="nextChapter"
            :disabled="currentChapterIndex === novel.chapters.length - 1"
            class="flex items-center px-6 py-3 rounded-lg border transition-all disabled:opacity-50 disabled:cursor-not-allowed hover:shadow-md"
          >
            下一章
            <svg class="w-5 h-5 ml-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/>
            </svg>
          </button>
        </nav>
      </article>
    </main>

    <!-- 底部阅读进度 -->
    <footer class="fixed bottom-0 left-0 right-0 bg-white/90 dark:bg-gray-900/90 backdrop-blur-sm border-t dark:border-gray-700">
      <div class="max-w-4xl mx-auto px-4 py-3 flex items-center justify-between text-sm text-gray-500">
        <span>{{ scrollProgress }}%</span>
        <span>{{ currentChapter.title }}</span>
      </div>
    </footer>
  </div>
</template>

<style scoped>
input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
}

input[type="range"]::-moz-range-thumb {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: none;
}
</style>
