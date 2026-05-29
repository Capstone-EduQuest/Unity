<script setup lang="ts">
import PageHeader from '../components/PageHeader.vue'
</script>

<template>
  <div class="min-h-screen bg-[#FFF2EF]">
    <PageHeader
      title="공지사항"
      subtitle="EduQuest의 최신 소식과 업데이트 내용을 확인해 보세요."
      back-link="/"
    />

    <main class="mx-auto max-w-6xl px-4 py-8 sm:px-6 lg:px-8">
      <section class="luxe-panel p-10 text-center">
        <p class="text-2xl font-black text-[#1A2A4F]">등록된 공지사항이 없습니다.</p>
        <p class="mt-3 text-sm leading-6 text-slate-600">
          새로운 공지가 등록되면 이곳에서 확인할 수 있습니다.
        </p>
      </section>
    </main>
  </div>
</template>
