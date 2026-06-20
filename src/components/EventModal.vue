<script setup lang="ts">
import { ref, watch, computed } from 'vue'
import type { GameState } from '@/types/game'
import type { ReleaseDestination } from '@/types/game'
import {
  RELEASE_DESTINATIONS, DESTINATION_NAMES, DESTINATION_EMOJI,
  DESTINATION_DESCRIPTIONS, DESTINATION_GRADIENT, DESTINATION_PERSONALITY_MATCH,
} from '@/utils/constants'
import { PERSONALITY_NAMES, PERSONALITY_EMOJI } from '@/utils/constants'

const props = defineProps<{
  state: GameState
  allAdults: boolean
}>()

const emit = defineEmits<{
  (e: 'release', destination: ReleaseDestination): void
  (e: 'breed'): void
}>()

const lastLogCount = ref(0)
const showTip = ref(false)
const tipMessage = ref('')
const showDestinationPicker = ref(false)
const selectedDestination = ref<ReleaseDestination | null>(null)

const adultBirds = computed(() =>
  props.state.birds.filter(b => b.stage === 'adult' && !b.isDead && !b.isReleased)
)

const personalityMatchText = (dest: ReleaseDestination): string => {
  const personalities = DESTINATION_PERSONALITY_MATCH[dest] || []
  if (personalities.length === 0) return ''
  return personalities.map(p => `${PERSONALITY_EMOJI[p]} ${PERSONALITY_NAMES[p]}`).join('、')
}

const openDestinationPicker = () => {
  selectedDestination.value = null
  showDestinationPicker.value = true
}

const confirmRelease = () => {
  if (selectedDestination.value) {
    emit('release', selectedDestination.value)
    showDestinationPicker.value = false
  }
}

watch(
  () => props.state.eventLog.length,
  (newLen) => {
    if (newLen > lastLogCount.value && props.state.eventLog[0]) {
      const last = props.state.eventLog[0]
      if (last.type === 'warning' || last.type === 'danger' || last.type === 'success') {
        tipMessage.value = last.message
        showTip.value = true
        setTimeout(() => { showTip.value = false }, 2500)
      }
    }
    lastLogCount.value = newLen
  },
  { immediate: true }
)
</script>

<template>
  <div class="relative w-full h-full flex flex-col gap-3 p-4">
    <div
      v-if="showTip"
      class="absolute top-16 left-1/2 -translate-x-1/2 z-50 animate-pop-in"
    >
      <div
        :class="[
          'px-5 py-2.5 rounded-2xl font-medium text-white shadow-2xl',
          state.eventLog[0]?.type === 'success' ? 'bg-gradient-to-r from-green-500 to-emerald-500' :
          state.eventLog[0]?.type === 'warning' ? 'bg-gradient-to-r from-amber-500 to-orange-500' :
          state.eventLog[0]?.type === 'danger' ? 'bg-gradient-to-r from-red-500 to-rose-500' :
          'bg-gradient-to-r from-blue-500 to-indigo-500',
        ]"
      >
        {{ tipMessage }}
      </div>
    </div>

    <Teleport to="body">
      <div
        v-if="showDestinationPicker"
        class="fixed inset-0 z-[100] flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm animate-fade-in"
        @click.self="showDestinationPicker = false"
      >
        <div class="bg-gradient-to-br from-indigo-900 via-purple-900 to-rose-900 rounded-3xl p-6 max-w-2xl w-full card-shadow border border-white/20 animate-pop-in max-h-[90vh] overflow-y-auto">
          <div class="text-center mb-5">
            <div class="text-4xl mb-2">🕊️</div>
            <div class="font-display text-3xl text-yellow-300 text-stroke mb-1">选择放飞去向</div>
            <div class="text-white/60 text-sm">为你的鸟儿们挑选一个适合的家吧~</div>
          </div>

          <div v-if="adultBirds.length > 0" class="mb-5 bg-white/5 rounded-2xl p-4 border border-white/10">
            <div class="text-white/70 text-xs mb-2">即将放飞的鸟儿：</div>
            <div class="flex flex-wrap gap-2">
              <span
                v-for="bird in adultBirds"
                :key="bird.id"
                class="bg-white/10 px-2.5 py-1 rounded-lg text-xs text-white/90 flex items-center gap-1"
              >
                <span>🐦</span>
                <span class="font-medium">{{ bird.name }}</span>
                <span class="text-white/50">{{ PERSONALITY_EMOJI[bird.personality] }}</span>
              </span>
            </div>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 mb-5">
            <button
              v-for="dest in RELEASE_DESTINATIONS"
              :key="dest"
              :class="[
                'relative p-4 rounded-2xl text-left transition-all border-2',
                selectedDestination === dest
                  ? 'border-yellow-400 scale-[1.02] ring-2 ring-yellow-400/40'
                  : 'border-white/10 hover:border-white/30 hover:scale-[1.01]',
              ]"
              @click="selectedDestination = dest"
            >
              <div :class="['absolute inset-0 rounded-2xl bg-gradient-to-br opacity-30', DESTINATION_GRADIENT[dest]]" />
              <div class="relative z-10">
                <div class="flex items-center gap-2 mb-2">
                  <span class="text-3xl">{{ DESTINATION_EMOJI[dest] }}</span>
                  <span class="font-bold text-lg text-white">{{ DESTINATION_NAMES[dest] }}</span>
                </div>
                <div class="text-white/60 text-xs mb-2">{{ DESTINATION_DESCRIPTIONS[dest] }}</div>
                <div v-if="personalityMatchText(dest)" class="text-[11px] text-yellow-300/80">
                  ✨ 适配性格：{{ personalityMatchText(dest) }}
                </div>
              </div>
              <div
                v-if="selectedDestination === dest"
                class="absolute top-2 right-2 w-6 h-6 rounded-full bg-yellow-400 flex items-center justify-center text-xs text-black font-bold"
              >
                ✓
              </div>
            </button>
          </div>

          <div class="flex gap-3 justify-center">
            <button
              class="px-6 py-3 bg-white/10 text-white rounded-2xl font-bold border border-white/20 hover:bg-white/20 transition-all"
              @click="showDestinationPicker = false"
            >
              取消
            </button>
            <button
              :disabled="!selectedDestination"
              :class="[
                'px-8 py-3 rounded-2xl font-bold flex items-center gap-2 transition-all',
                selectedDestination
                  ? 'bg-gradient-to-r from-sky-500 to-blue-600 text-white btn-3d hover:from-sky-400 hover:to-blue-500'
                  : 'bg-gray-500/50 text-white/50 cursor-not-allowed',
              ]"
              @click="confirmRelease"
            >
              <span class="text-xl">🕊️</span>
              确认放飞
            </button>
          </div>
        </div>
      </div>
    </Teleport>

    <div v-if="allAdults" class="animate-pop-in">
      <div class="glass rounded-2xl p-5 card-shadow border border-yellow-400/30 bg-gradient-to-br from-yellow-500/10 to-orange-500/10">
        <div class="text-center mb-4">
          <div class="text-3xl mb-2">🎉</div>
          <div class="font-display text-2xl text-yellow-300 text-stroke">全部长成成鸟啦！</div>
          <div class="text-sm text-white/70 mt-1">请选择它们的未来...</div>
        </div>
        <div class="flex gap-3 justify-center flex-wrap">
          <button
            class="px-6 py-3 bg-gradient-to-r from-sky-500 to-blue-600 text-white rounded-2xl font-bold
                   btn-3d hover:from-sky-400 hover:to-blue-500 flex items-center gap-2"
            @click="openDestinationPicker"
          >
            <span class="text-xl">🕊️</span>
            放飞自由
          </button>
          <button
            v-if="state.breedingCount < state.maxBreedingRounds"
            class="px-6 py-3 bg-gradient-to-r from-pink-500 to-rose-600 text-white rounded-2xl font-bold
                   btn-3d hover:from-pink-400 hover:to-rose-500 flex items-center gap-2"
            @click="emit('breed')"
          >
            <span class="text-xl">💝</span>
            留下配对 ({{ state.breedingCount }}/{{ state.maxBreedingRounds }})
          </button>
          <button
            v-else
            class="px-6 py-3 bg-gradient-to-r from-amber-500 to-yellow-600 text-white rounded-2xl font-bold
                   btn-3d hover:from-amber-400 hover:to-yellow-500 flex items-center gap-2"
            @click="openDestinationPicker"
          >
            <span class="text-xl">🏡</span>
            放飞（结束）
          </button>
        </div>
      </div>
    </div>

    <div class="flex-1 min-h-0 space-y-2 overflow-y-auto scrollbar-hide pr-1">
      <div v-for="log in state.eventLog.slice(0, 15)" :key="log.id" class="text-xs">
        <div
          :class="[
            'px-3 py-2 rounded-xl flex items-start gap-2',
            log.type === 'success' ? 'bg-green-500/15 text-green-200' :
            log.type === 'warning' ? 'bg-amber-500/15 text-amber-200' :
            log.type === 'danger' ? 'bg-red-500/15 text-red-200' :
            'bg-white/10 text-white/80',
          ]"
        >
          <span class="opacity-60 shrink-0 text-[10px] mt-0.5">
            {{ new Date(log.timestamp).toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' }) }}
          </span>
          <span>{{ log.message }}</span>
        </div>
      </div>
      <div v-if="state.eventLog.length === 0" class="text-center text-white/40 text-sm py-8">
        游戏日志将在这里显示~
      </div>
    </div>
  </div>
</template>
