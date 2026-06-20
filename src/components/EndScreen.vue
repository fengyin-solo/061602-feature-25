<script setup lang="ts">
import { computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useGameState } from '@/composables/useGameState'
import {
  DESTINATION_NAMES, DESTINATION_EMOJI, DESTINATION_DESCRIPTIONS,
  DESTINATION_GRADIENT, DESTINATION_PERSONALITY_MATCH,
} from '@/utils/constants'
import { PERSONALITY_NAMES, PERSONALITY_EMOJI, STAGE_EMOJI } from '@/utils/constants'
import type { ReleaseDestination } from '@/types/game'

const router = useRouter()
const { state, restartGame, returnToStart } = useGameState()

onMounted(() => {
  if (state.phase !== 'ended' && !state.score) {
    router.push('/')
  }
})

const score = computed(() => state.score)

const starArray = computed(() => {
  const s = score.value?.stars ?? 1
  return Array.from({ length: 5 }, (_, i) => i < s)
})

const releasedBirds = computed(() =>
  state.birds.filter(b => b.isReleased && !b.isDead)
)

const keptBirds = computed(() =>
  state.birds.filter(b => !b.isReleased && !b.isDead && b.stage === 'adult')
)

const deadBirds = computed(() =>
  state.birds.filter(b => b.isDead)
)

const endReasonText = computed(() => {
  switch (state.endReason) {
    case 'release': return '🕊️ 放飞了你的鸟儿们'
    case 'keep': return '🏡 选择与鸟儿们相伴终老'
    case 'allDead': return '💔 所有的小鸟都离开了...'
    default: return '🎮 游戏结束'
  }
})

const destinationStats = computed(() => {
  const stats: { dest: ReleaseDestination; count: number; birds: typeof releasedBirds.value }[] = []
  const destMap = new Map<ReleaseDestination, typeof releasedBirds.value>()

  releasedBirds.value.forEach(b => {
    const dest = b.releaseDestination || 'unknown'
    if (!destMap.has(dest)) destMap.set(dest, [])
    destMap.get(dest)!.push(b)
  })

  destMap.forEach((birds, dest) => {
    stats.push({ dest, count: birds.length, birds })
  })

  return stats.sort((a, b) => b.count - a.count)
})

const personalityMatchCount = computed(() => {
  if (!state.releaseDestination) return 0
  const matches = DESTINATION_PERSONALITY_MATCH[state.releaseDestination] || []
  return releasedBirds.value.filter(b => matches.includes(b.personality)).length
})

const handleRestart = () => {
  restartGame()
  router.push('/game')
}

const handleHome = () => {
  returnToStart()
  router.push('/')
}
</script>

<template>
  <div class="w-full h-full bg-gradient-to-br from-indigo-900 via-purple-900 to-rose-900 flex items-center justify-center p-6 overflow-auto">
    <div class="absolute inset-0 overflow-hidden pointer-events-none">
      <div class="absolute top-10 left-[10%] text-5xl opacity-20 animate-float">✨</div>
      <div class="absolute top-1/4 right-[15%] text-4xl opacity-20 animate-float" style="animation-delay: 0.5s">🌟</div>
      <div class="absolute bottom-1/4 left-[20%] text-5xl opacity-20 animate-float" style="animation-delay: 1s">🐦</div>
      <div class="absolute bottom-20 right-[10%] text-4xl opacity-20 animate-float" style="animation-delay: 1.5s">🌿</div>
    </div>

    <div v-if="score" class="relative z-10 max-w-2xl w-full">
      <div class="text-center mb-6 animate-pop-in">
        <div class="text-7xl mb-4">{{ score.stars >= 4 ? '🏆' : score.stars >= 3 ? '🎉' : '🌱' }}</div>
        <h1 class="font-display text-5xl text-white mb-2 text-stroke">游戏结束</h1>
        <div class="text-white/70 text-lg mb-2">{{ endReasonText }}</div>
        <div v-if="state.endReason === 'release' && state.releaseDestination && state.releaseDestination !== 'unknown'" class="text-white/60 text-sm">
          放飞去向：{{ DESTINATION_EMOJI[state.releaseDestination] }} {{ DESTINATION_NAMES[state.releaseDestination] }}
        </div>
        <div class="font-display text-3xl text-amber-300 mb-6 mt-2">{{ score.rank }}</div>
      </div>

      <div class="flex justify-center gap-2 mb-8 animate-pop-in" style="animation-delay: 0.1s">
        <div
          v-for="(filled, idx) in starArray"
          :key="idx"
          class="text-5xl transition-all"
          :class="filled ? 'animate-bounce-slow' : 'opacity-20 grayscale'"
          :style="{ animationDelay: `${idx * 0.1}s` }"
        >
          {{ filled ? '⭐' : '☆' }}
        </div>
      </div>

      <div class="glass rounded-3xl p-8 card-shadow animate-pop-in" style="animation-delay: 0.2s">
        <div class="text-center mb-8">
          <div class="text-white/60 text-sm mb-2">综合得分</div>
          <div class="font-display text-7xl text-yellow-300 text-stroke mb-1">
            {{ score.totalScore }}
            <span class="text-3xl text-white/60">/100</span>
          </div>
        </div>

        <div :class="['grid gap-3 mb-8', state.endReason === 'release' ? 'grid-cols-2 sm:grid-cols-3' : 'grid-cols-2']">
          <div class="bg-white/5 rounded-2xl p-4 text-center border border-white/10">
            <div class="text-3xl mb-2">💚</div>
            <div class="text-white/60 text-xs mb-1">成活率</div>
            <div class="font-bold text-2xl text-green-400">{{ score.survivalRate }}%</div>
          </div>

          <div class="bg-white/5 rounded-2xl p-4 text-center border border-white/10">
            <div class="text-3xl mb-2">❤️</div>
            <div class="text-white/60 text-xs mb-1">平均健康</div>
            <div class="font-bold text-2xl text-red-400">{{ score.avgHealth }}</div>
          </div>

          <div class="bg-white/5 rounded-2xl p-4 text-center border border-white/10">
            <div class="text-3xl mb-2">💝</div>
            <div class="text-white/60 text-xs mb-1">繁殖加成</div>
            <div class="font-bold text-2xl text-pink-400">+{{ score.breedingBonus }}</div>
          </div>

          <div class="bg-white/5 rounded-2xl p-4 text-center border border-white/10">
            <div class="text-3xl mb-2">🌟</div>
            <div class="text-white/60 text-xs mb-1">照料加成</div>
            <div class="font-bold text-2xl text-amber-400">+{{ score.personalityBonus }}</div>
          </div>

          <template v-if="state.endReason === 'release'">
            <div class="bg-white/5 rounded-2xl p-4 text-center border border-white/10">
              <div class="text-3xl mb-2">🕊️</div>
              <div class="text-white/60 text-xs mb-1">放飞加成</div>
              <div class="font-bold text-2xl text-sky-400">+{{ score.releaseBonus }}</div>
            </div>

            <div class="bg-white/5 rounded-2xl p-4 text-center border border-white/10">
              <div class="text-3xl mb-2">🗺️</div>
              <div class="text-white/60 text-xs mb-1">去向加成</div>
              <div class="font-bold text-2xl text-emerald-400">+{{ score.destinationBonus }}</div>
            </div>
          </template>
        </div>

        <div v-if="state.endReason === 'release' && state.releaseDestination && state.releaseDestination !== 'unknown'"
          class="bg-gradient-to-r from-sky-500/10 to-emerald-500/10 rounded-2xl p-5 mb-6 border border-sky-400/20">
          <div class="text-center">
            <div class="text-white/80 mb-3 flex items-center justify-center gap-2">
              <span>{{ DESTINATION_EMOJI[state.releaseDestination] }}</span>
              <span class="font-medium">放飞去向：{{ DESTINATION_NAMES[state.releaseDestination] }}</span>
            </div>
            <div class="text-white/60 text-sm mb-3">{{ DESTINATION_DESCRIPTIONS[state.releaseDestination] }}</div>
            <div class="flex flex-wrap justify-center gap-4 text-sm">
              <span class="text-white/70">
                <span class="text-sky-300">🕊️ 放飞数量：</span>{{ state.totalReleased }} 只
              </span>
              <span v-if="personalityMatchCount > 0" class="text-white/70">
                <span class="text-yellow-300">✨ 性格适配：</span>{{ personalityMatchCount }} 只
              </span>
            </div>
          </div>
        </div>

        <div class="bg-gradient-to-r from-amber-500/10 to-rose-500/10 rounded-2xl p-5 mb-6 border border-amber-400/20">
          <div class="text-center">
            <div class="text-white/80 mb-3 flex items-center justify-center gap-2">
              <span>📊</span>
              <span class="font-medium">本次总结</span>
            </div>
            <div class="text-white/60 text-sm flex flex-wrap justify-center gap-x-6 gap-y-1">
              <span>📅 第 {{ state.day }} 天</span>
              <span>🥚 孵化 {{ state.totalHatched }} 只</span>
              <span>💔 离世 {{ state.totalDied }} 只</span>
              <span>💝 繁殖 {{ state.breedingCount }} 窝</span>
              <span v-if="state.totalReleased > 0">🕊️ 放飞 {{ state.totalReleased }} 只</span>
              <span>🐦 巢中 {{ state.birds.filter(b => !b.isDead && !b.isReleased).length }} 只</span>
            </div>
          </div>
        </div>

        <div v-if="state.endReason === 'release' && destinationStats.length > 0" class="mb-6">
          <div class="text-white/60 text-sm text-center mb-3">🗺️ 放飞去向统计</div>
          <div class="space-y-2">
            <div
              v-for="stat in destinationStats"
              :key="stat.dest"
              class="bg-white/5 rounded-xl p-3 border border-white/10"
            >
              <div class="flex items-center justify-between mb-2">
                <div class="flex items-center gap-2">
                  <span class="text-xl">{{ DESTINATION_EMOJI[stat.dest] }}</span>
                  <span class="font-medium text-white">{{ DESTINATION_NAMES[stat.dest] }}</span>
                </div>
                <span class="text-sm text-white/60">{{ stat.count }} 只</span>
              </div>
              <div class="flex flex-wrap gap-1.5">
                <span
                  v-for="bird in stat.birds"
                  :key="bird.id"
                  class="bg-white/10 px-2 py-0.5 rounded-lg text-xs text-white/80 flex items-center gap-1"
                >
                  <span>🐦</span>
                  <span>{{ bird.name }}</span>
                  <span class="text-white/50">{{ PERSONALITY_EMOJI[bird.personality] }}</span>
                </span>
              </div>
            </div>
          </div>
        </div>

        <div v-if="releasedBirds.length > 0 || keptBirds.length > 0" class="mb-6">
          <div class="text-white/60 text-sm text-center mb-3">💐 那些陪伴过你的鸟儿们</div>

          <div v-if="state.endReason === 'release' && releasedBirds.length > 0" class="mb-3">
            <div class="text-sky-300/80 text-xs mb-2 text-center">🕊️ 已放飞</div>
            <div class="flex flex-wrap justify-center gap-2">
              <div
                v-for="bird in releasedBirds"
                :key="bird.id"
                class="bg-white/10 px-3 py-1.5 rounded-xl text-sm text-white/90 flex items-center gap-1.5"
              >
                <span>🐦</span>
                <span class="font-medium">{{ bird.name }}</span>
                <span v-if="bird.releaseDestination" class="text-xs text-sky-300/70">
                  {{ DESTINATION_EMOJI[bird.releaseDestination] }}
                </span>
              </div>
            </div>
          </div>

          <div v-if="keptBirds.length > 0">
            <div class="text-amber-300/80 text-xs mb-2 text-center">
              {{ state.endReason === 'keep' ? '🏡 留在巢中相伴' : '🏡 留在巢中' }}
            </div>
            <div class="flex flex-wrap justify-center gap-2">
              <div
                v-for="bird in keptBirds"
                :key="bird.id"
                class="bg-white/10 px-3 py-1.5 rounded-xl text-sm text-white/90 flex items-center gap-1.5"
              >
                <span>🐦</span>
                <span class="font-medium">{{ bird.name }}</span>
              </div>
            </div>
          </div>
        </div>

        <div v-if="deadBirds.length > 0" class="mb-6">
          <div class="text-red-300/60 text-xs mb-2 text-center">💔 已离世</div>
          <div class="flex flex-wrap justify-center gap-2">
            <div
              v-for="bird in deadBirds"
              :key="bird.id"
              class="bg-red-500/10 px-3 py-1.5 rounded-xl text-sm text-red-300/70 flex items-center gap-1.5 border border-red-400/10"
            >
              <span>🪦</span>
              <span class="font-medium">{{ bird.name }}</span>
              <span class="text-xs opacity-60">{{ STAGE_EMOJI[bird.stage] }}</span>
            </div>
          </div>
        </div>

        <div v-if="state.eventLog.length > 0" class="mb-6">
          <div class="text-white/60 text-sm text-center mb-3">📝 事件回顾</div>
          <div class="max-h-48 overflow-y-auto space-y-1.5 scrollbar-hide pr-1">
            <div
              v-for="log in state.eventLog.slice(0, 20)"
              :key="log.id"
              :class="[
                'px-3 py-1.5 rounded-lg text-xs flex items-start gap-2',
                log.type === 'success' ? 'bg-green-500/10 text-green-200/80' :
                log.type === 'warning' ? 'bg-amber-500/10 text-amber-200/80' :
                log.type === 'danger' ? 'bg-red-500/10 text-red-200/80' :
                'bg-white/5 text-white/60',
              ]"
            >
              <span class="opacity-50 shrink-0 text-[10px] mt-0.5">
                {{ new Date(log.timestamp).toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' }) }}
              </span>
              <span>{{ log.message }}</span>
            </div>
          </div>
        </div>

        <div class="flex flex-col sm:flex-row gap-3 justify-center">
          <button
            class="px-8 py-4 bg-gradient-to-r from-green-500 to-emerald-600 text-white rounded-2xl
                   font-bold text-lg btn-3d hover:from-green-400 hover:to-emerald-500 flex items-center justify-center gap-2"
            @click="handleRestart"
          >
            <span class="text-xl">🪺</span>
            再来一窝！
          </button>
          <button
            class="px-8 py-4 bg-gradient-to-r from-slate-600 to-slate-700 text-white rounded-2xl
                   font-bold text-lg btn-3d hover:from-slate-500 hover:to-slate-600 flex items-center justify-center gap-2"
            @click="handleHome"
          >
            <span class="text-xl">🏠</span>
            返回主页
          </button>
        </div>
      </div>

      <div class="text-center mt-6 text-white/40 text-sm">
        🌸 感谢游玩！希望你和小鸟们都收获了快乐~
      </div>
    </div>
  </div>
</template>
