<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { useRoute } from 'vue-router'
import PageHeader from '../components/PageHeader.vue'
import { bookmarkAPI, type BookmarkItem } from '../api/bookmark'
import { useAuthStore } from '../store/auth'

const route = useRoute()
const auth = useAuthStore()
const bookmarks = ref<BookmarkItem[]>([])
const error = ref('')
const isLoading = ref(true)

onMounted(async () => {
  await auth.restoreAuth(route.path)
  if (!auth.state.user) {
    error.value = '로그인이 필요한 기능입니다.'
    isLoading.value = false
    return
  }

  try {
    const response = await bookmarkAPI.getBookmarkList(auth.state.user.uuid, {
      page: 1,
      size: 50,
      sort: 'created_at',
      is_asc: false,
    })
    bookmarks.value = response.results
  } catch (fetchError) {
    console.error(fetchError)
    error.value = '북마크를 불러오는데 실패했습니다.'
  } finally {
    isLoading.value = false
  }
})
</script>

<template>
  <div class="min-h-screen bg-[#FFF2EF]">
    <PageHeader
      title="북마크"
      subtitle="중요한 문제를 모아두고 빠르게 다시 확인해 보세요."
      back-link="/"
    />

    <main class="mx-auto max-w-6xl px-4 py-8 sm:px-6 lg:px-8">
      <div
        v-if="error"
        class="luxe-card mb-6 p-6 text-sm font-medium text-[#1A2A4F]"
      >
        {{ error }}
      </div>

      <div
        v-else-if="isLoading"
        class="luxe-panel p-10 text-center"
      >
        <div class="mx-auto mb-4 h-14 w-14 animate-spin rounded-full border-2 border-[#FFDBB6] border-b-[#1A2A4F]" />
        <p class="font-bold text-[#1A2A4F]">북마크를 불러오는 중입니다.</p>
      </div>

      <div
        v-else-if="bookmarks.length === 0"
        class="luxe-panel p-10 text-center"
      >
        <p class="text-2xl font-black text-[#1A2A4F]">저장된 북마크가 없습니다.</p>
        <p class="mt-3 text-sm leading-6 text-slate-600">
          다시 풀어보고 싶은 문제를 북마크하면 이곳에서 한눈에 확인할 수 있어요.
        </p>
      </div>

      <section v-else class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-3">
        <article
          v-for="item in bookmarks"
          :key="item.problem_uuid ?? `${item.stage}-${item.number}`"
          class="luxe-card flex flex-col p-6 transition duration-300 hover:translate-y-[-2px]"
        >
          <div class="flex items-start justify-between gap-4">
            <h2 class="text-2xl font-black text-[#1A2A4F]">{{ item.stage ?? 'Stage -' }}</h2>
            <span class="luxe-pill shrink-0 px-3 py-1 text-xs font-medium uppercase tracking-[0.14em] text-[#1A2A4F]">
              {{ item.type ?? '문제' }}
            </span>
          </div>

          <div class="mt-5 rounded-[24px] border border-[#1A2A4F]/10 bg-[#FFF8F4] p-4">
            <p class="text-sm font-medium text-[#1A2A4F]/55">문제 번호</p>
            <p class="mt-2 text-3xl font-black text-[#1A2A4F]">{{ item.number ?? '-' }}</p>
          </div>

          <div class="mt-4 flex items-center justify-between rounded-[20px] border border-[#1A2A4F]/10 bg-[#FFF6EC] p-4">
            <span class="text-sm font-medium text-[#1A2A4F]">다시 볼 문제</span>
            <span class="text-sm font-bold text-[#1A2A4F]/55">Saved</span>
          </div>
        </article>
      </section>
    </main>
  </div>
</template>
