<script lang="ts">
// X の公式スクリプト（widgets.js）は、1ページに投稿がいくつあっても1回だけ読み込む
let loading: Promise<void> | null = null

function loadWidgets(): Promise<void> {
  const w = window as any
  if (w.twttr?.widgets) return Promise.resolve()
  if (!loading) {
    loading = new Promise<void>((resolve, reject) => {
      const s = document.createElement('script')
      s.src = 'https://platform.twitter.com/widgets.js'
      s.async = true
      s.charset = 'utf-8'
      s.onload = () => resolve()
      s.onerror = () => {
        loading = null
        reject(new Error('widgets.js を読み込めませんでした'))
      }
      document.head.appendChild(s)
    })
  }
  return loading
}
</script>

<script setup lang="ts">
// 記事の中で <XPost url="https://x.com/ユーザー名/status/数字" /> と書くと、X（旧 Twitter）の投稿をカードで表示する。
// ページを開いたときに X の公式スクリプトを読み込んでカードにする。
// 読み込めなかったときは、投稿へのリンクだけが表示される。
import { computed, onMounted, ref } from 'vue'
import { useData } from 'vitepress'

const props = defineProps<{ url: string }>()
const { isDark } = useData()
const box = ref<HTMLElement | null>(null)

// 公式スクリプトは twitter.com の URL を前提にしているので、x.com の URL は書き換えて渡す
const href = computed(() => props.url.replace(/^https?:\/\/(www\.)?(x|twitter)\.com\//, 'https://twitter.com/'))

onMounted(async () => {
  try {
    await loadWidgets()
    ;(window as any).twttr?.widgets?.load(box.value ?? undefined)
  } catch {
    // 読み込めなかったときは、投稿へのリンクのまま表示しておく
  }
})
</script>

<template>
  <div ref="box" class="x-post">
    <blockquote class="twitter-tweet" data-lang="ja" data-dnt="true" :data-theme="isDark ? 'dark' : 'light'">
      <a :href="href">X の投稿を見る</a>
    </blockquote>
  </div>
</template>

<style scoped>
.x-post {
  margin: 24px 0;
}

/* 公式スクリプトが読み込まれる前（読み込めなかったとき）は、ふつうのリンクとして表示する */
.x-post .twitter-tweet {
  margin: 0;
  padding: 12px 16px;
  border: 1px solid var(--vp-c-divider);
  border-radius: 8px;
  background: none;
  color: var(--vp-c-text-2);
  font-size: 14px;
}

.x-post :deep(.twitter-tweet-rendered) {
  margin: 0 auto !important;
}
</style>
