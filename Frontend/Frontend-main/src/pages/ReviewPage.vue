<script setup lang="ts">
import PageHeader from '../components/PageHeader.vue'

const reviewStages = [
  {
    id: 1,
    title: '변수와 자료형',
    summary: '기본 개념을 다시 정리하고, 헷갈리기 쉬운 선언 방식만 빠르게 복습해보세요.',
    date: '2026.05.01',
  },
  {
    id: 2,
    title: 'for 반복문',
    summary: '반복문의 실행 순서와 자주 틀리는 조건문 연결 부분을 확인할 수 있어요.',
    date: '2026.05.02',
  },
  {
    id: 3,
    title: '리스트와 인덱싱',
    summary: '인덱스 접근, 슬라이싱, 자주 나오는 실수 포인트를 묶어서 다시 볼 수 있어요.',
    date: '2026.05.03',
  },
]
</script>

<template>
  <div class="min-h-screen bg-[#FFF2EF]">
    <PageHeader
      title="복습하기"
      subtitle="지금까지 학습한 내용을 다시 훑고, 중요한 개념만 빠르게 복습해보세요."
      back-link="/home"
    />

    <main class="mx-auto max-w-5xl px-4 py-8 sm:px-6 lg:px-8">
      <section class="grid gap-4">
        <article
          v-for="stage in reviewStages"
          :key="stage.id"
          class="luxe-card flex flex-col gap-5 p-6 sm:flex-row sm:items-center sm:justify-between"
        >
          <div class="min-w-0">
            <div class="flex flex-wrap items-center gap-3">
              <p class="rounded-full border border-[#1A2A4F]/10 bg-[#FFF8F4] px-3 py-1 text-xs font-medium uppercase tracking-[0.14em] text-[#1A2A4F]">
                Review {{ stage.id }}
              </p>
              <p class="text-sm text-[#1A2A4F]/45">{{ stage.date }}</p>
            </div>
            <h2 class="mt-4 text-2xl font-black text-[#1A2A4F]">{{ stage.title }}</h2>
            <p class="mt-3 max-w-2xl text-sm leading-6 text-[#1A2A4F]/68">
              {{ stage.summary }}
            </p>
          </div>

          <button
            type="button"
            class="luxe-button-accent cursor-pointer shrink-0 rounded-full px-5 py-3 text-sm font-medium transition duration-300 hover:translate-y-[-1px]"
          >
            복습 시작
          </button>
        </article>
      </section>
    </main>
  </div>
</template>
