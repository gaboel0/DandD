<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

interface Proficiency {
  index: string
  name: string
  url: string
}

interface SavingThrow {
  index: string
  name: string
  url: string
}

interface Subclass {
  index: string
  name: string
  url: string
}

interface SpellcastingInfo {
  name: string
  desc: string[]
}

interface Spellcasting {
  level: number
  spellcasting_ability: { index: string; name: string }
  info: SpellcastingInfo[]
}

interface ProficiencyChoice {
  desc: string
  choose: number
  type: string
  from: { options: Array<{ item: Proficiency }> }
}

interface MultiClassingPrerequisite {
  ability_score: { index: string; name: string }
  minimum_score: number
}

interface MultiClassing {
  prerequisites?: MultiClassingPrerequisite[]
  prerequisite_options?: {
    choose: number
    from: { options: Array<{ item: { index: string; name: string }; minimum_score: number }> }
  }
  proficiencies: Proficiency[]
}

interface DClass {
  index: string
  name: string
  hit_die: number
  proficiency_choices: ProficiencyChoice[]
  proficiencies: Proficiency[]
  saving_throws: SavingThrow[]
  subclasses: Subclass[]
  spellcasting?: Spellcasting
  multi_classing: MultiClassing
  url: string
}

interface ClassMeta {
  color: string
  icon: string
  ptName: string
  role: string
}

const BASE_URL = 'https://www.dnd5eapi.co'

const route = useRoute()
const router = useRouter()
const dclass = ref<DClass | null>(null)
const loading = ref(true)
const error = ref<string | null>(null)

const classIndex = route.params.index as string

const classMeta: Record<string, ClassMeta> = {
  barbarian: { color: '#ef4444', icon: '⚔', ptName: 'Bárbaro', role: 'Guerreiro Furioso' },
  bard: { color: '#a855f7', icon: '♪', ptName: 'Bardo', role: 'Artista Mágico' },
  cleric: { color: '#f59e0b', icon: '✦', ptName: 'Clérigo', role: 'Servo Divino' },
  druid: { color: '#22c55e', icon: '❧', ptName: 'Druida', role: 'Guardião da Natureza' },
  fighter: { color: '#94a3b8', icon: '🛡', ptName: 'Guerreiro', role: 'Mestre do Combate' },
  monk: { color: '#06b6d4', icon: '☯', ptName: 'Monge', role: 'Mestre das Artes Marciais' },
  paladin: { color: '#60a5fa', icon: '✙', ptName: 'Paladino', role: 'Cavaleiro Sagrado' },
  ranger: { color: '#4ade80', icon: '⟁', ptName: 'Patrulheiro', role: 'Caçador das Terras Ermas' },
  rogue: { color: '#818cf8', icon: '◈', ptName: 'Ladino', role: 'Especialista Furtivo' },
  sorcerer: { color: '#f97316', icon: '✺', ptName: 'Feiticeiro', role: 'Magia Inata' },
  warlock: { color: '#c084fc', icon: '☽', ptName: 'Bruxo', role: 'Pacto Arcano' },
  wizard: { color: '#38bdf8', icon: '✦', ptName: 'Mago', role: 'Estudioso da Magia' },
}

const profTranslations: Record<string, string> = {
  'All armor': 'Todas as armaduras',
  'Light Armor': 'Armadura Leve',
  'Medium Armor': 'Armadura Média',
  'Heavy Armor': 'Armadura Pesada',
  Shields: 'Escudos',
  'Simple Weapons': 'Armas Simples',
  'Martial Weapons': 'Armas Marciais',
  'Simple Melee Weapons': 'Armas de Combate Simples',
  Shortswords: 'Espadas Curtas',
  Longswords: 'Espadas Longas',
  Rapiers: 'Rapieiras',
  Handcrossbows: 'Bestas de Mão',
  Longbows: 'Arcos Longos',
  Shortbows: 'Arcos Curtos',
  "Thieves' Tools": 'Ferramentas de Ladrão',
  'Herbalism Kit': 'Kit de Herbalismo',
  'Disguise Kit': 'Kit de Disfarce',
  "Poisoner's Kit": 'Kit de Venenos',
  'Forgery Kit': 'Kit de Falsificação',
  "Alchemist's Supplies": 'Suprimentos de Alquimista',
  "Brewer's Supplies": 'Suprimentos de Cervejeiro',
  "Smith's Tools": 'Ferramentas de Ferreiro',
  "Tinker's Tools": 'Ferramentas de Artífice',
  "Mason's Tools": 'Ferramentas de Pedreiro',
  "Carpenter's Tools": 'Ferramentas de Carpinteiro',
  "Woodcarver's Tools": 'Ferramentas de Entalhador',
  "Leatherworker's Tools": 'Ferramentas de Correeiro',
  "Weaver's Tools": 'Ferramentas de Tecelão',
  "Jeweler's Tools": 'Ferramentas de Joalheiro',
  "Glassblower's Tools": 'Ferramentas de Vidreiro',
  "Calligrapher's Supplies": 'Suprimentos de Caligrafia',
  "Painter's Supplies": 'Suprimentos de Pintor',
  "Cook's Utensils": 'Utensílios de Cozinheiro',
  "Cobbler's Tools": 'Ferramentas de Sapateiro',
  "Cartographer's Tools": 'Ferramentas de Cartógrafo',
  "Potter's Tools": 'Ferramentas de Oleiro',
  "One type of artisan's tools": 'Um tipo de ferramentas de artesão',
  'One type of musical instrument': 'Um instrumento musical',
  'Musical instrument': 'Instrumento musical',
}

const subclassTranslations: Record<string, string> = {
  Berserker: 'Berserker',
  'Totem Warrior': 'Guerreiro Totêmico',
  'College of Lore': 'Colégio do Saber',
  'College of Valor': 'Colégio do Valor',
  Life: 'Domínio da Vida',
  Light: 'Domínio da Luz',
  Knowledge: 'Domínio do Conhecimento',
  Nature: 'Domínio da Natureza',
  Tempest: 'Domínio da Tempestade',
  Trickery: 'Domínio da Trapaça',
  War: 'Domínio da Guerra',
  Land: 'Círculo da Terra',
  Moon: 'Círculo da Lua',
  Champion: 'Campeão',
  'Battle Master': 'Mestre de Batalha',
  'Eldritch Knight': 'Cavaleiro Místico',
  'Open Hand': 'Mão Aberta',
  Shadow: 'Sombra',
  'Four Elements': 'Quatro Elementos',
  'Oath of Devotion': 'Juramento de Devoção',
  'Oath of the Ancients': 'Juramento dos Antigos',
  'Oath of Vengeance': 'Juramento de Vingança',
  Hunter: 'Caçador',
  'Beast Master': 'Mestre das Bestas',
  Thief: 'Ladrão',
  Assassin: 'Assassino',
  'Arcane Trickster': 'Trapaceiro Arcano',
  'Draconic Bloodline': 'Linhagem Dracônica',
  'Wild Magic': 'Magia Selvagem',
  'The Fiend': 'O Demônio',
  'The Great Old One': 'O Grande Ancião',
  'The Archfey': 'A Archfada',
  'School of Abjuration': 'Escola de Abjuração',
  'School of Conjuration': 'Escola de Conjuração',
  'School of Divination': 'Escola de Adivinhação',
  'School of Enchantment': 'Escola de Encantamento',
  'School of Evocation': 'Escola de Evocação',
  'School of Illusion': 'Escola de Ilusão',
  'School of Necromancy': 'Escola de Necromancia',
  'School of Transmutation': 'Escola de Transmutação',
}

const skillTranslations: Record<string, string> = {
  'Animal Handling': 'Trato com Animais',
  'Sleight of Hand': 'Prestidigitação',
  Acrobatics: 'Acrobacia',
  Arcana: 'Arcanismo',
  Athletics: 'Atletismo',
  Deception: 'Enganação',
  History: 'História',
  Insight: 'Intuição',
  Intimidation: 'Intimidação',
  Investigation: 'Investigação',
  Medicine: 'Medicina',
  Nature: 'Natureza',
  Perception: 'Percepção',
  Performance: 'Atuação',
  Persuasion: 'Persuasão',
  Religion: 'Religião',
  Stealth: 'Furtividade',
  Survival: 'Sobrevivência',
}

const spellInfoNamePT: Record<string, string> = {
  Cantrips: 'Truques',
  Spellbook: 'Grimório',
  'Preparing and Casting Spells': 'Preparar e Lançar Magias',
  'Spellcasting Ability': 'Habilidade de Conjuração',
  'Ritual Casting': 'Lançamento Ritual',
  'Spellcasting Focus': 'Foco de Conjuração',
  'Known Spells of 1st Level and Higher': 'Magias Conhecidas',
  'Spells Known of 1st Level and Higher': 'Magias Conhecidas',
  'Spell Slots': 'Espaços de Magia',
  'The Mystic Arcanum': 'O Arcanum Místico',
  'Eldritch Invocations': 'Invocações Místicas',
  Metamagic: 'Metamagia',
  'Wild Magic Surge': 'Surto de Magia Selvagem',
  'Tides of Chaos': 'Marés do Caos',
  'Font of Magic': 'Fonte de Magia',
}

const abilityPT: Record<string, string> = {
  str: 'Força',
  dex: 'Destreza',
  con: 'Constituição',
  int: 'Inteligência',
  wis: 'Sabedoria',
  cha: 'Carisma',
}

function translateProfName(name: string): string {
  return profTranslations[name] ?? name
}

function translateSubclass(name: string): string {
  return subclassTranslations[name] ?? name
}

function translateChoiceDesc(desc: string): string {
  let result = desc
  const sortedSkills = Object.entries(skillTranslations).sort((a, b) => b[0].length - a[0].length)
  for (const [en, pt] of sortedSkills) {
    result = result.replaceAll(en, pt)
  }
  result = result.replace(/^Choose\s+(\d+)\s+from/i, 'Escolha $1 entre')
  result = result.replace(/^Choose\s+any\s+(\d+)\s+skills?/i, 'Escolha $1 perícias quaisquer')
  result = result.replace(/^Choose\s+(\d+)/i, 'Escolha $1')
  result = result.replace(/\bfrom\b/gi, 'entre')
  result = result.replace(/\band\b/gi, 'e')
  result = result.replace(/\bany\b/gi, 'qualquer')
  result = result.replace(/\bskills?\b/gi, 'perícia(s)')
  return result
}

function translateSpellInfoName(name: string): string {
  return spellInfoNamePT[name] ?? name
}

onMounted(async () => {
  try {
    const res = await fetch(`${BASE_URL}/api/2014/classes/${classIndex}`)
    if (!res.ok) throw new Error('HTTP error')
    dclass.value = await res.json()
  } catch {
    error.value = 'Esta classe não pôde ser encontrada nos Arquivos Arcanos.'
  } finally {
    loading.value = false
  }
})

const meta = computed<ClassMeta>(
  () => classMeta[classIndex] ?? { color: '#c9a84c', icon: '✦', ptName: classIndex, role: '' },
)

const armorProfs = computed(
  () =>
    dclass.value?.proficiencies.filter(
      (p) => p.name.toLowerCase().includes('armor') || p.name.toLowerCase().includes('shield'),
    ) ?? [],
)

const weaponProfs = computed(
  () =>
    dclass.value?.proficiencies.filter(
      (p) => p.name.toLowerCase().includes('weapon') || p.name.toLowerCase().includes('weapons'),
    ) ?? [],
)

const toolProfs = computed(
  () =>
    dclass.value?.proficiencies.filter(
      (p) =>
        !p.name.toLowerCase().includes('armor') &&
        !p.name.toLowerCase().includes('shield') &&
        !p.name.toLowerCase().includes('weapon') &&
        !p.name.toLowerCase().includes('saving'),
    ) ?? [],
)

const multiclassPrereqs = computed(() => {
  const mc = dclass.value?.multi_classing
  if (!mc) return []
  const list: Array<{ label: string; min: number }> = []
  if (mc.prerequisites) {
    for (const p of mc.prerequisites) {
      list.push({
        label: abilityPT[p.ability_score.index] ?? p.ability_score.name,
        min: p.minimum_score,
      })
    }
  }
  if (mc.prerequisite_options) {
    for (const opt of mc.prerequisite_options.from.options) {
      list.push({ label: abilityPT[opt.item.index] ?? opt.item.name, min: opt.minimum_score })
    }
  }
  return list
})

const multiclassProfs = computed(() => dclass.value?.multi_classing?.proficiencies ?? [])
</script>

<template>
  <div class="page-bg bg-[#0a0a0e] min-h-screen">
    <div class="max-w-[1100px] mx-auto px-6 pb-16">
      <!-- TOP BAR -->
      <div class="pt-7">
        <button
          class="font-cinzel text-[0.8rem] font-semibold tracking-[0.1rem] text-amber-500/70 bg-transparent border border-amber-500/20 rounded-[5px] px-5 py-[0.55rem] cursor-pointer transition-all duration-200 hover:text-[#c9a84c] hover:border-amber-500/55 hover:bg-amber-500/[0.06]"
          @click="router.push({ name: 'home' })"
        >
          ← Voltar às Classes
        </button>
      </div>

      <!-- LOADING -->
      <div v-if="loading" class="text-center py-24 font-cinzel text-amber-500/50">
        <div
          class="w-12 h-12 border-[3px] border-amber-500/15 rounded-full animate-spin mx-auto mb-6"
          :style="{ borderTopColor: meta.color }"
        />
        <p>Consultando os Grimórios...</p>
      </div>

      <!-- ERROR -->
      <div v-else-if="error || !dclass" class="text-center py-24 font-cinzel text-amber-500/50">
        <span class="text-[3.5rem] block mb-4">☠</span>
        <p>{{ error }}</p>
        <button
          class="font-cinzel text-[0.8rem] font-semibold tracking-[0.1rem] text-amber-500/70 bg-transparent border border-amber-500/20 rounded-[5px] px-5 py-[0.55rem] cursor-pointer transition-all duration-200 hover:text-[#c9a84c] hover:border-amber-500/55 hover:bg-amber-500/[0.06] mt-6"
          @click="router.push({ name: 'home' })"
        >
          ← Voltar
        </button>
      </div>

      <!-- CONTENT -->
      <template v-else>
        <div :style="{ '--accent': meta.color }">
          <!-- HERO HEADER -->
          <header
            class="flex items-start gap-8 py-10 pb-8 border-b border-amber-500/[0.12] max-sm:flex-col max-sm:items-center max-sm:text-center"
          >
            <div
              class="text-[4rem] leading-none shrink-0 [color:var(--accent)]"
              :style="{
                filter: `drop-shadow(0 0 20px color-mix(in srgb, ${meta.color} 40%, transparent))`,
              }"
            >
              {{ meta.icon }}
            </div>
            <div>
              <h1
                class="font-cinzel-dec text-[clamp(2rem,5vw,3rem)] font-bold [color:var(--accent)] m-0 mb-[0.15rem]"
                :style="{
                  textShadow: `0 0 40px color-mix(in srgb, ${meta.color} 25%, transparent)`,
                }"
              >
                {{ meta.ptName }}
              </h1>
              <p
                class="font-cinzel text-[0.75rem] text-[#e2d4b7]/30 uppercase tracking-[0.25rem] m-0 mb-[0.35rem]"
              >
                {{ dclass.name }}
              </p>
              <p class="font-cinzel text-[0.85rem] text-amber-500/60 m-0 mb-3 tracking-[0.06rem]">
                {{ meta.role }}
              </p>
            </div>
          </header>

          <!-- ACCENT LINE -->
          <div
            class="h-0.5 w-20 rounded-sm mt-2 mb-10 opacity-60"
            :style="{ background: meta.color }"
          />

          <!-- MAIN LAYOUT -->
          <div class="flex gap-8 items-start max-md:flex-col">
            <!-- LEFT COLUMN -->
            <aside class="shrink-0 w-[260px] flex flex-col gap-4 max-md:w-full">
              <!-- HIT DIE -->
              <div
                class="bg-[#111118] border rounded-[10px] p-6 text-center shadow-inner"
                :style="{ borderColor: `color-mix(in srgb, ${meta.color} 30%, transparent)` }"
              >
                <div
                  class="font-cinzel-dec text-[2.8rem] font-bold leading-none [color:var(--accent)]"
                  :style="{
                    textShadow: `0 0 30px color-mix(in srgb, ${meta.color} 40%, transparent)`,
                  }"
                >
                  d{{ dclass.hit_die }}
                </div>
                <p
                  class="font-cinzel text-[0.68rem] text-amber-500/50 uppercase tracking-[0.15rem] mt-[0.4rem] mb-[0.2rem]"
                >
                  Dado de Vida
                </p>
                <p class="font-cinzel text-[0.72rem] text-[#e2d4b7]/35 m-0">
                  {{ dclass.hit_die }} PV por nível
                </p>
              </div>

              <!-- SAVING THROWS -->
              <div
                class="bg-[#111118] border border-amber-500/[0.12] rounded-[10px] px-[1.1rem] py-4 transition-[border-color] duration-200 hover:[border-color:color-mix(in_srgb,var(--accent)_25%,transparent)]"
              >
                <h3
                  class="font-cinzel text-[0.65rem] font-bold text-amber-500/50 uppercase tracking-[0.18rem] m-0 mb-3"
                >
                  Salvaguardas
                </h3>
                <div class="flex flex-wrap gap-[0.4rem]">
                  <span
                    v-for="st in dclass.saving_throws"
                    :key="st.index"
                    class="inline-block font-cinzel text-[0.72rem] font-semibold [color:var(--accent)] px-[0.65rem] py-[0.2rem] rounded"
                    :style="{
                      background: `color-mix(in srgb, ${meta.color} 12%, transparent)`,
                      border: `1px solid color-mix(in srgb, ${meta.color} 30%, transparent)`,
                    }"
                  >
                    {{ abilityPT[st.index] ?? st.name }}
                  </span>
                </div>
              </div>

              <!-- SPELLCASTING -->
              <div
                v-if="dclass.spellcasting"
                class="bg-[#111118] border rounded-[10px] px-[1.1rem] py-4"
                :style="{ borderColor: `color-mix(in srgb, ${meta.color} 20%, transparent)` }"
              >
                <h3
                  class="font-cinzel text-[0.65rem] font-bold text-amber-500/50 uppercase tracking-[0.18rem] m-0 mb-3"
                >
                  Conjuração
                </h3>
                <p class="font-cinzel text-[0.78rem] text-[#e2d4b7]/60 m-0 mb-[0.35rem]">
                  Atributo:
                  <strong class="[color:var(--accent)]">
                    {{
                      abilityPT[dclass.spellcasting.spellcasting_ability.index] ??
                      dclass.spellcasting.spellcasting_ability.name
                    }}
                  </strong>
                </p>
                <p class="font-cinzel text-[0.7rem] text-[#e2d4b7]/35 m-0">
                  Disponível a partir do nível {{ dclass.spellcasting.level }}
                </p>
              </div>

              <!-- SUBCLASSES -->
              <div
                v-if="dclass.subclasses.length"
                class="bg-[#111118] border border-amber-500/[0.12] rounded-[10px] px-[1.1rem] py-4 transition-[border-color] duration-200 hover:[border-color:color-mix(in_srgb,var(--accent)_25%,transparent)]"
              >
                <h3
                  class="font-cinzel text-[0.65rem] font-bold text-amber-500/50 uppercase tracking-[0.18rem] m-0 mb-3"
                >
                  Subclasses
                </h3>
                <ul class="list-none p-0 m-0 flex flex-col gap-[0.45rem]">
                  <li
                    v-for="sub in dclass.subclasses"
                    :key="sub.index"
                    class="font-cinzel text-[0.75rem] text-[#e2d4b7]/60 flex items-center gap-2"
                  >
                    <span
                      class="w-[5px] h-[5px] rounded-full shrink-0 opacity-70"
                      :style="{ background: meta.color }"
                    />
                    {{ translateSubclass(sub.name) }}
                  </li>
                </ul>
              </div>

              <!-- MULTICLASSE -->
              <div
                v-if="multiclassPrereqs.length || multiclassProfs.length"
                class="bg-[#111118] border border-amber-500/[0.12] rounded-[10px] px-[1.1rem] py-4 transition-[border-color] duration-200 hover:[border-color:color-mix(in_srgb,var(--accent)_25%,transparent)]"
              >
                <h3
                  class="font-cinzel text-[0.65rem] font-bold text-amber-500/50 uppercase tracking-[0.18rem] m-0 mb-3"
                >
                  Multiclasse
                </h3>
                <div v-if="multiclassPrereqs.length">
                  <p
                    class="font-cinzel text-[0.65rem] text-amber-500/45 uppercase tracking-[0.1rem] m-0 mb-[0.45rem]"
                  >
                    Pré-requisitos de atributo:
                  </p>
                  <div class="flex flex-wrap gap-[0.4rem]">
                    <span
                      v-for="req in multiclassPrereqs"
                      :key="req.label"
                      class="inline-block font-cinzel text-[0.72rem] font-semibold [color:var(--accent)] px-[0.65rem] py-[0.2rem] rounded"
                      :style="{
                        background: `color-mix(in srgb, ${meta.color} 12%, transparent)`,
                        border: `1px solid color-mix(in srgb, ${meta.color} 30%, transparent)`,
                      }"
                    >
                      {{ req.label }} {{ req.min }}+
                    </span>
                  </div>
                </div>
                <div v-if="multiclassProfs.length" class="mt-[0.6rem]">
                  <p
                    class="font-cinzel text-[0.65rem] text-amber-500/45 uppercase tracking-[0.1rem] m-0 mb-[0.45rem]"
                  >
                    Proficiências ganhas:
                  </p>
                  <div class="flex flex-wrap gap-[0.4rem]">
                    <span
                      v-for="p in multiclassProfs"
                      :key="p.index"
                      class="font-cinzel text-[0.7rem] text-[#e2d4b7]/65 bg-white/[0.04] border border-white/[0.07] px-[0.65rem] py-[0.2rem] rounded"
                    >
                      {{ translateProfName(p.name) }}
                    </span>
                  </div>
                </div>
              </div>
            </aside>

            <!-- RIGHT COLUMN -->
            <div class="flex-1 min-w-0">
              <!-- PROFICIÊNCIAS -->
              <section class="mb-8">
                <h2
                  class="font-cinzel text-[0.72rem] font-bold text-amber-500/55 uppercase tracking-[0.25rem] m-0 mb-5 flex items-center gap-[0.6rem]"
                >
                  <span class="[color:var(--accent)] opacity-50">⚔</span>
                  Proficiências
                </h2>
                <div class="flex flex-col gap-4">
                  <div
                    v-if="armorProfs.length"
                    class="bg-[#111118] border border-amber-500/10 rounded-[8px] p-4"
                  >
                    <h4
                      class="font-cinzel text-[0.65rem] font-bold text-amber-500/45 uppercase tracking-[0.14rem] m-0 mb-[0.65rem]"
                    >
                      Armaduras & Escudos
                    </h4>
                    <div class="flex flex-wrap gap-[0.4rem]">
                      <span
                        v-for="p in armorProfs"
                        :key="p.index"
                        class="font-cinzel text-[0.7rem] text-[#e2d4b7]/65 bg-white/[0.04] border border-white/[0.07] px-[0.65rem] py-[0.2rem] rounded"
                      >
                        {{ translateProfName(p.name) }}
                      </span>
                    </div>
                  </div>
                  <div
                    v-if="weaponProfs.length"
                    class="bg-[#111118] border border-amber-500/10 rounded-[8px] p-4"
                  >
                    <h4
                      class="font-cinzel text-[0.65rem] font-bold text-amber-500/45 uppercase tracking-[0.14rem] m-0 mb-[0.65rem]"
                    >
                      Armas
                    </h4>
                    <div class="flex flex-wrap gap-[0.4rem]">
                      <span
                        v-for="p in weaponProfs"
                        :key="p.index"
                        class="font-cinzel text-[0.7rem] text-[#e2d4b7]/65 bg-white/[0.04] border border-white/[0.07] px-[0.65rem] py-[0.2rem] rounded"
                      >
                        {{ translateProfName(p.name) }}
                      </span>
                    </div>
                  </div>
                  <div
                    v-if="toolProfs.length"
                    class="bg-[#111118] border border-amber-500/10 rounded-[8px] p-4"
                  >
                    <h4
                      class="font-cinzel text-[0.65rem] font-bold text-amber-500/45 uppercase tracking-[0.14rem] m-0 mb-[0.65rem]"
                    >
                      Ferramentas & Outros
                    </h4>
                    <div class="flex flex-wrap gap-[0.4rem]">
                      <span
                        v-for="p in toolProfs"
                        :key="p.index"
                        class="font-cinzel text-[0.7rem] text-[#e2d4b7]/65 bg-white/[0.04] border border-white/[0.07] px-[0.65rem] py-[0.2rem] rounded"
                      >
                        {{ translateProfName(p.name) }}
                      </span>
                    </div>
                  </div>
                </div>
              </section>

              <!-- ESCOLHAS DE PERÍCIAS -->
              <section v-if="dclass.proficiency_choices.length" class="mb-8">
                <h2
                  class="font-cinzel text-[0.72rem] font-bold text-amber-500/55 uppercase tracking-[0.25rem] m-0 mb-5 flex items-center gap-[0.6rem]"
                >
                  <span class="[color:var(--accent)] opacity-50">⚔</span>
                  Escolhas de Perícias
                </h2>
                <div
                  v-for="(choice, i) in dclass.proficiency_choices"
                  :key="i"
                  class="bg-[#111118] border border-amber-500/10 border-l-[3px] rounded-r-[8px] px-4 py-[0.85rem] mb-3"
                  :style="{ borderLeftColor: `color-mix(in srgb, ${meta.color} 50%, transparent)` }"
                >
                  <p class="font-cinzel text-[0.78rem] text-[#e2d4b7]/60 leading-[1.7] m-0">
                    {{ translateChoiceDesc(choice.desc) }}
                  </p>
                </div>
              </section>

              <!-- REGRAS DE CONJURAÇÃO -->
              <section v-if="dclass.spellcasting?.info?.length" class="mb-8">
                <h2
                  class="font-cinzel text-[0.72rem] font-bold text-amber-500/55 uppercase tracking-[0.25rem] m-0 mb-5 flex items-center gap-[0.6rem]"
                >
                  <span class="[color:var(--accent)] opacity-50">⚔</span>
                  Regras de Conjuração
                </h2>
                <div
                  v-for="info in dclass.spellcasting.info"
                  :key="info.name"
                  class="bg-[#111118] border border-amber-500/10 border-l-[3px] rounded-r-[8px] px-4 py-[0.9rem] mb-3"
                  :style="{ borderLeftColor: `color-mix(in srgb, ${meta.color} 50%, transparent)` }"
                >
                  <h4
                    class="font-cinzel text-[0.75rem] font-bold [color:var(--accent)] m-0 mb-[0.45rem] tracking-[0.06rem]"
                  >
                    {{ translateSpellInfoName(info.name) }}
                  </h4>
                  <p
                    v-for="(line, li) in info.desc"
                    :key="li"
                    class="font-cinzel text-[0.75rem] text-[#e2d4b7]/[0.58] leading-[1.75] m-0 mb-[0.35rem] last:mb-0"
                  >
                    {{ line }}
                  </p>
                </div>
              </section>
            </div>
          </div>
        </div>
      </template>
    </div>

    <footer
      class="text-center py-8 border-t border-amber-500/[0.09] font-cinzel text-[0.68rem] text-amber-500/[0.28] tracking-[0.15rem]"
    >
      <p>D&D 5e SRD API · Classes Jogáveis · Material Acadêmico</p>
    </footer>
  </div>
</template>
