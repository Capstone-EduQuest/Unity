<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { useRouter } from 'vue-router'
import { userAPI } from '../api/auth'
import { hintAPI, problemAPI, submissionAPI, type HintResponse, type ProblemDetail } from '../api/learning'
import { useAuthStore } from '../store/auth'

const props = defineProps<{
  problemId?: string | null
}>()

const router = useRouter()
const auth = useAuthStore()
const problem = ref<ProblemDetail | null>(null)
const code = ref('# Python 코드를 작성하세요.\nprint("Hello, World!")')
const blockAnswer = ref<number[]>([])
const hintMessage = ref('')
const usedHintLevels = ref<Set<number>>(new Set())
const submissionResult = ref<{ success: boolean; message: string } | null>(null)
const isLoading = ref(false)

const parsedBlock = computed(() => {
  const rawBlock = problem.value?.block
  if (!rawBlock) {
    return null
  }

  if (typeof rawBlock === 'string') {
    try {
      return JSON.parse(rawBlock) as { answer?: number[]; blocks?: Array<{ code: string }> }
    } catch {
      return null
    }
  }

  return rawBlock
})

const hasBlocks = computed(() => Boolean(parsedBlock.value?.blocks?.length))
const availableHints = computed<HintResponse[]>(() =>
  [...(problem.value?.hints ?? [])].sort((left, right) => left.level - right.level),
)

const refreshCurrentUserCoin = async () => {
  const user = auth.state.user
  const userUuid = user?.uuid
  if (!userUuid) {
    return
  }

  try {
    const profile = await userAPI.getProfile(userUuid)
    user.balance = profile.wallet?.balance ?? profile.point ?? user.balance
  } catch (error) {
    console.warn('failed to refresh user coin after hint:', error)
  }
}

const wait = (ms: number) => new Promise((resolve) => window.setTimeout(resolve, ms))

const pollEvaluationResult = async (submissionUuid: string) => {
  const maxAttempts = 20

  for (let attempt = 0; attempt < maxAttempts; attempt += 1) {
    const evaluation = await submissionAPI.getEvaluation(submissionUuid)

    if (evaluation.status === 'completed') {
      return Boolean(evaluation.result)
    }

    await wait(700)
  }

  throw new Error('채점 결과를 가져오지 못했습니다.')
}

const fetchProblem = async () => {
  if (!props.problemId) {
    return
  }

  try {
    problem.value = await problemAPI.getProblem(props.problemId)
    blockAnswer.value = []
    hintMessage.value = ''
    usedHintLevels.value = new Set()
    submissionResult.value = null
  } catch (error) {
    console.error('failed to fetch problem:', error)
    alert('문제를 불러오지 못했습니다.')
  }
}

const handleSubmit = async () => {
  if (!problem.value) {
    return
  }

  const answer = hasBlocks.value && blockAnswer.value.length > 0 ? blockAnswer.value : code.value
  if (typeof answer === 'string' && !answer.trim()) {
    return
  }

  isLoading.value = true
  submissionResult.value = null

  try {
    const response = await submissionAPI.submitProblem(problem.value.uuid, answer)
    const isCorrect = await pollEvaluationResult(response.uuid)

    submissionResult.value = {
      success: isCorrect,
      message: isCorrect ? '정답입니다.' : '오답입니다. 다시 도전해 보세요.',
    }

    if (isCorrect) {
      setTimeout(() => {
        router.push('/')
      }, 2000)
    }
  } catch (error: any) {
    submissionResult.value = {
      success: false,
      message: error.response?.data?.details ?? error.response?.data?.message ?? '제출에 실패했습니다.',
    }
  } finally {
    isLoading.value = false
  }
}

const handleHint = async (level: number) => {
  if (!problem.value) {
    return
  }

  try {
    const hintMeta = availableHints.value.find((hint) => hint.level === level)
    const wasAlreadyUsedInThisView = usedHintLevels.value.has(level)
    const response = await hintAPI.getHint(problem.value.uuid, level)
    const nextUsedLevels = new Set(usedHintLevels.value)
    nextUsedLevels.add(level)
    usedHintLevels.value = nextUsedLevels

    const cost = response.point ?? hintMeta?.point ?? 0
    const prefix = wasAlreadyUsedInThisView
      ? '이미 확인한 힌트입니다.'
      : `힌트 사용 완료: ${cost}코인이 차감되었습니다.`
    hintMessage.value = `${prefix}\n${response.content ?? ''}`.trim()
    await refreshCurrentUserCoin()

    if (typeof window !== 'undefined') {
      window.dispatchEvent(new CustomEvent('eduquest:coin-updated'))
    }
  } catch (error: any) {
    hintMessage.value = resolveHintErrorMessage(error)
  }
}

const resolveHintErrorMessage = (error: any) => {
  const data = error?.response?.data
  const details = data?.details
  const detailCode = typeof details === 'object' && details !== null ? details.code : ''
  const serverMessage = data?.message ?? data?.error ?? (typeof details === 'string' ? details : '')

  if (
    data?.code === 'INSUFFICIENT_BALANCE' ||
    detailCode === 'INSUFFICIENT_BALANCE' ||
    String(serverMessage).includes('코인이 부족')
  ) {
    return '코인이 부족합니다.'
  }

  if (
    data?.code === 'HINT_NOT_FOUND' ||
    detailCode === 'HINT_NOT_FOUND' ||
    String(serverMessage).includes('해당 단계의 힌트가 없습니다') ||
    String(serverMessage).includes('힌트')
  ) {
    return String(serverMessage).includes('해당 단계의 힌트가 없습니다')
      ? '해당 단계의 힌트가 없습니다.'
      : serverMessage || '해당 단계의 힌트가 없습니다.'
  }

  return serverMessage || '힌트를 불러오지 못했습니다.'
}

const appendBlock = (index: number) => {
  blockAnswer.value = [...blockAnswer.value, index]
}

watch(() => props.problemId, fetchProblem, { immediate: true })
onMounted(fetchProblem)
</script>

<template>
  <div
    v-if="!problem"
    class="rounded-2xl border-4 border-dashed border-gray-600 bg-gray-800 p-10 text-center text-white"
  >
    <h2 class="mb-4 text-3xl font-bold">문제 로딩 중...</h2>
    <div class="mx-auto h-12 w-12 animate-spin rounded-full border-b-2 border-blue-400" />
  </div>

  <div
    v-else
    class="flex max-h-[90vh] flex-col overflow-hidden rounded-2xl border-4 border-gray-600 bg-gray-800 p-8 text-white"
  >
    <div class="mb-6 shrink-0">
      <h2 class="mb-2 text-2xl font-bold text-blue-400">문제 {{ problem.number }}</h2>
      <p class="mb-4 whitespace-pre-line text-gray-300">{{ problem.summary }}</p>

      <div v-if="problem.example" class="mb-4 rounded-lg bg-gray-700 p-4">
        <h3 class="mb-2 font-bold text-green-400">예제 입력</h3>
        <pre class="text-sm text-gray-200">{{ problem.example }}</pre>
      </div>

      <div v-if="problem.expectedOutput" class="rounded-lg bg-gray-700 p-4">
        <h3 class="mb-2 font-bold text-green-400">예제 출력</h3>
        <pre class="text-sm text-gray-200">{{ problem.expectedOutput }}</pre>
      </div>
    </div>

    <div class="flex min-h-0 flex-1 flex-col">
      <div v-if="hasBlocks" class="mb-4 rounded-xl bg-gray-900 p-4">
        <p class="mb-3 text-sm text-gray-300">
          블록 문제인 경우 버튼을 눌러 정답 순서를 만들어 보세요.
        </p>
        <div class="flex flex-wrap gap-2">
          <button
            v-for="(block, index) in parsedBlock?.blocks"
            :key="`${index}-${block.code}`"
            type="button"
            class="rounded-lg bg-gray-700 px-3 py-2 text-sm hover:bg-gray-600"
            @click="appendBlock(index + 1)"
          >
            {{ index + 1 }}. {{ block.code }}
          </button>
        </div>
        <p class="mt-3 text-sm text-blue-300">선택 순서: {{ blockAnswer.join(', ') || '아직 없음' }}</p>
      </div>

      <textarea
        v-model="code"
        class="w-full flex-1 resize-none rounded-lg bg-gray-900 p-4 font-mono text-sm text-green-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
        placeholder="Python 코드를 작성하세요."
      />

      <div v-if="availableHints.length" class="mt-4 flex flex-wrap gap-2">
        <button
          v-for="hint in availableHints"
          :key="hint.level"
          type="button"
          class="rounded-lg border border-blue-400 px-3 py-2 text-sm text-blue-300 hover:bg-blue-400 hover:text-white"
          @click="handleHint(hint.level)"
        >
          힌트 {{ hint.level }} (-{{ hint.point }} 코인)
        </button>
      </div>

      <div v-if="hintMessage" class="mt-4 whitespace-pre-line rounded-lg bg-blue-950 p-4 text-sm text-blue-100">
        {{ hintMessage }}
      </div>

      <div
        v-if="submissionResult"
        class="mt-4 rounded-lg p-4"
        :class="submissionResult.success ? 'bg-green-800' : 'bg-red-800'"
      >
        <h3 class="font-bold">{{ submissionResult.success ? '제출 성공' : '제출 실패' }}</h3>
        <p class="mt-2 text-sm">{{ submissionResult.message }}</p>
      </div>

      <div class="mt-4 flex gap-4">
        <button
          type="button"
          disabled
          class="flex-1 cursor-not-allowed rounded-lg bg-blue-400 px-6 py-3 font-bold text-white"
        >
          코드 실행 (미지원)
        </button>
        <button
          type="button"
          :disabled="isLoading"
          class="flex-1 rounded-lg bg-green-600 px-6 py-3 font-bold text-white transition-colors hover:bg-green-500 disabled:bg-gray-600"
          @click="handleSubmit"
        >
          {{ isLoading ? '제출 중...' : '정답 제출' }}
        </button>
      </div>
    </div>
  </div>
</template>
