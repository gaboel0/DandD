<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

interface ClassEntry {
  index: string
  name: string
  url: string
}

const BASE_URL = 'https://www.dnd5eapi.co'

const router = useRouter()
const allClasses = ref<ClassEntry[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const searchQuery = ref('')

const classMeta: Record<string, { color: string; icon: string; ptName: string; role: string }> = {
  barbarian: { color: '#ef4444', icon: '⚔', ptName: 'Bárbaro', role: 'Guerreiro Furioso' },
  bard: { color: '#a855f7', icon: '♪', ptName: 'Bardo', role: 'Artista Mágico' },
  cleric: { color: '#f59e0b', icon: '✦', ptName: 'Clérigo', role: 'Servo Divino' },
  druid: { color: '#22c55e', icon: '❧', ptName: 'Druida', role: 'Guardião da Natureza' },
  fighter: { color: '#94a3b8', icon: '🛡', ptName: 'Guerreiro', role: 'Mestre do Combate' },
  monk: { color: '#06b6d4', icon: '☯', ptName: 'Monge', role: 'Mestre das Artes Marciais' },
  paladin: { color: '#60a5fa', icon: '✙', ptName: 'Paladino', role: 'Cavaleiro Sagrado' },
  ranger: { color: '#4ade80', icon: '⟁', ptName: 'Patrulheiro', role: 'Caçador das Terras' },
  rogue: { color: '#818cf8', icon: '◈', ptName: 'Ladino', role: 'Especialista Furtivo' },
  sorcerer: { color: '#f97316', icon: '✺', ptName: 'Feiticeiro', role: 'Magia Inata' },
  warlock: { color: '#c084fc', icon: '☽', ptName: 'Bruxo', role: 'Pacto Arcano' },
  wizard: { color: '#38bdf8', icon: '✦', ptName: 'Mago', role: 'Estudioso da Magia' },
}

onMounted(async () => {
  try {
    const res = await fetch(`${BASE_URL}/api/2014/classes`)
    if (!res.ok) throw new Error('HTTP error')
    const data: { count: number; results: ClassEntry[] } = await res.json()
    allClasses.value = data.results
  } catch {
    error.value = 'Não foi possível carregar os pergaminhos de classe. Verifique sua conexão.'
  } finally {
    loading.value = false
  }
})

const filteredClasses = computed(() => {
  const q = searchQuery.value.toLowerCase().trim()
  if (!q) return allClasses.value
  return allClasses.value.filter(
    (c) => c.name.toLowerCase().includes(q) || classMeta[c.index]?.ptName.toLowerCase().includes(q),
  )
})

function navigate(index: string) {
  router.push({ name: 'class-detail', params: { index } })
}

function meta(index: string) {
  return classMeta[index] ?? { color: '#c9a84c', icon: '✦', ptName: index, role: '' }
}
</script>

<template>
  <div class="page-bg bg-[#0a0a0e] min-h-screen">
    <!-- HEADER -->
    <header
      class="text-center px-6 pt-14 pb-10 border-b border-amber-500/[0.14] bg-gradient-to-b from-amber-500/[0.05] to-transparent"
    >
      <div class="max-w-[1280px] mx-auto">
        <div class="text-[#c9a84c] text-base tracking-[0.6rem] mb-2 opacity-80">⚔ ✦ ⚔</div>
        <p class="font-cinzel text-amber-500/55 text-[0.72rem] tracking-[0.35rem] uppercase my-2">
          Grimório do Aventureiro
        </p>
        <h1
          class="font-cinzel-dec text-[clamp(2.8rem,8vw,5.5rem)] font-bold text-[#c9a84c] m-0 leading-none [text-shadow:0_0_60px_rgba(201,168,76,0.25),0_2px_6px_rgba(0,0,0,0.8)]"
        >
          CLASSES
        </h1>
        <h2
          class="font-cinzel text-[clamp(0.8rem,2vw,1.1rem)] font-normal text-[#e2d4b7]/55 mt-1 mb-4 tracking-[0.5rem] uppercase"
        >
          de D&D 5ª Edição
        </h2>
        <div class="text-[#c9a84c] text-base tracking-[0.6rem] opacity-80">⚔ ✦ ⚔</div>
        <p class="font-cinzel text-[0.75rem] text-amber-500/45 tracking-[0.08rem] mt-4">
          12 classes jogáveis do SRD 2014
        </p>
      </div>
    </header>

    <div class="max-w-[1280px] mx-auto px-6 pb-16">
      <!-- SEARCH -->
      <div class="my-10 flex justify-center">
        <div
          class="relative flex items-center w-full max-w-[480px] bg-white/[0.025] border border-amber-500/[0.22] rounded-[6px] transition-all duration-300 focus-within:border-amber-500/55 focus-within:[box-shadow:0_0_24px_rgba(201,168,76,0.07)]"
        >
          <svg
            class="w-[18px] h-[18px] ml-4 mr-3 text-amber-500/55 shrink-0"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <circle cx="11" cy="11" r="8" />
            <path d="M21 21l-4.35-4.35" stroke-linecap="round" />
          </svg>
          <input
            v-model="searchQuery"
            type="text"
            class="flex-1 bg-transparent border-none outline-none py-[0.85rem] font-cinzel text-[0.9rem] text-[#e2d4b7] tracking-[0.04rem] placeholder:text-[#e2d4b7]/28"
            placeholder="Buscar classe..."
          />
          <button
            v-if="searchQuery"
            class="px-4 bg-transparent border-none text-[#e2d4b7]/35 cursor-pointer text-[0.85rem] transition-colors duration-200 hover:text-[#c9a84c]"
            @click="searchQuery = ''"
          >
            ✕
          </button>
        </div>
      </div>

      <!-- LOADING SKELETONS -->
      <div v-if="loading" class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-5">
        <div
          v-for="n in 12"
          :key="n"
          class="skeleton-shimmer h-[220px] border border-amber-500/[0.07] rounded-[10px]"
        />
      </div>

      <!-- ERROR -->
      <div v-else-if="error" class="text-center py-20 font-cinzel text-amber-500/45">
        <span class="text-[3rem] block mb-4">☠</span>
        <p>{{ error }}</p>
      </div>

      <!-- GRID -->
      <template v-else>
        <div
          v-if="filteredClasses.length"
          class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-5"
        >
          <article
            v-for="c in filteredClasses"
            :key="c.index"
            class="relative bg-[#111118] border border-white/[0.06] rounded-[10px] overflow-hidden cursor-pointer flex flex-col items-center pt-0 px-4 pb-5 transition-all duration-[250ms] ease-out hover:-translate-y-[6px] hover:scale-[1.015] group"
            :style="{ '--accent': meta(c.index).color }"
            @click="navigate(c.index)"
          >
            <!-- top accent bar -->
            <div
              class="w-full h-[3px] mb-6 opacity-70 transition-opacity duration-[250ms] group-hover:opacity-100"
              :style="{ background: meta(c.index).color }"
            />

            <!-- icon -->
            <div
              class="text-[2.4rem] leading-none mb-3 [color:var(--accent)]"
              :style="{
                filter: `drop-shadow(0 0 12px color-mix(in srgb, ${meta(c.index).color} 50%, transparent))`,
              }"
            >
              {{ meta(c.index).icon }}
            </div>

            <!-- names -->
            <div class="text-center">
              <h3 class="font-cinzel-dec text-base font-bold text-[#e2d4b7] m-0 mb-[0.1rem]">
                {{ meta(c.index).ptName }}
              </h3>
              <p
                class="font-cinzel text-[0.65rem] text-[#e2d4b7]/35 uppercase tracking-[0.15rem] m-0 mb-2"
              >
                {{ c.name }}
              </p>
              <p class="font-cinzel text-[0.7rem] text-amber-500/50 m-0 tracking-[0.04rem]">
                {{ meta(c.index).role }}
              </p>
            </div>

            <!-- hover overlay -->
            <div
              class="absolute inset-0 bg-[#0a0a0e]/80 flex items-center justify-center opacity-0 transition-opacity duration-[250ms] group-hover:opacity-100"
            >
              <span
                class="font-cinzel text-[0.8rem] font-bold tracking-[0.2rem] [color:var(--accent)] uppercase px-[1.3rem] py-[0.55rem] rounded"
                :style="{
                  border: `1px solid color-mix(in srgb, ${meta(c.index).color} 55%, transparent)`,
                  background: `color-mix(in srgb, ${meta(c.index).color} 8%, transparent)`,
                }"
              >
                Ver Detalhes
              </span>
            </div>
          </article>
        </div>

        <!-- EMPTY STATE -->
        <div v-else class="text-center py-20 font-cinzel text-amber-500/45">
          <span class="text-[3rem] block mb-4">🔍</span>
          <p>
            Nenhuma classe encontrada para "<em>{{ searchQuery }}</em
            >"
          </p>
        </div>
      </template>
    </div>

    <footer
      class="text-center py-8 border-t border-amber-500/[0.09] font-cinzel text-[0.68rem] text-amber-500/[0.28] tracking-[0.15rem]"
    >
      <p>D&D API · Classes Jogáveis · Material Acadêmico</p>
    </footer>
  </div>
</template>
