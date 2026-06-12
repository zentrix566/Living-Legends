<script setup>
import { computed, onMounted, ref } from 'vue'

const legends = ref([])
const isLoading = ref(true)
const loadError = ref('')
const searchText = ref('')
const statusFilter = ref('all')

const today = new Date()

onMounted(async () => {
  try {
    const response = await fetch('/data/living-legends.json')
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`)
    }
    legends.value = await response.json()
  } catch (error) {
    loadError.value = `人物文档读取失败：${error.message}`
  } finally {
    isLoading.value = false
  }
})

function calculateAge(birthDate, endDate = today) {
  const birth = new Date(`${birthDate}T00:00:00`)
  const end = endDate instanceof Date ? endDate : new Date(`${endDate}T00:00:00`)
  let age = end.getFullYear() - birth.getFullYear()
  const hasHadBirthday =
    end.getMonth() > birth.getMonth() ||
    (end.getMonth() === birth.getMonth() && end.getDate() >= birth.getDate())

  return hasHadBirthday ? age : age - 1
}

function formatDate(date) {
  return new Intl.DateTimeFormat('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
  }).format(new Date(`${date}T00:00:00`))
}

function enrichLegend(legend) {
  const isDeceased = Boolean(legend.deathDate)
  const age = calculateAge(legend.birthDate, legend.deathDate || today)

  return {
    ...legend,
    isDeceased,
    age,
    ageLabel: isDeceased ? `享年 ${age} 岁` : `${age} 岁`,
    statusLabel: isDeceased ? '已故' : '在世',
  }
}

const enrichedLegends = computed(() => legends.value.map(enrichLegend))

const filteredLegends = computed(() => {
  const keyword = searchText.value.trim().toLowerCase()

  return enrichedLegends.value.filter((legend) => {
    const matchesStatus =
      statusFilter.value === 'all' ||
      (statusFilter.value === 'living' && !legend.isDeceased) ||
      (statusFilter.value === 'deceased' && legend.isDeceased)

    const matchesKeyword =
      !keyword ||
      [
        legend.name,
        legend.romanizedName,
        legend.country,
        legend.event,
        legend.majorDeed,
      ]
        .filter(Boolean)
        .some((value) => value.toLowerCase().includes(keyword))

    return matchesStatus && matchesKeyword
  })
})

const totalLiving = computed(
  () => enrichedLegends.value.filter((legend) => !legend.isDeceased).length,
)

const totalDeceased = computed(
  () => enrichedLegends.value.filter((legend) => legend.isDeceased).length,
)
</script>

<template>
  <main class="shell">
    <section class="masthead">
      <div>
        <p class="eyebrow">Living Legends Archive</p>
        <h1>在世传奇</h1>
        <p class="intro">
          记录重大历史事件中的幸存者与关键当事人；如果未来人物去世，在文档中补充去世日期即可自动改变状态。
        </p>
      </div>

      <div class="stats-panel" aria-label="人物统计">
        <div>
          <span>{{ legends.length }}</span>
          <small>总人数</small>
        </div>
        <div>
          <span>{{ totalLiving }}</span>
          <small>在世</small>
        </div>
        <div>
          <span>{{ totalDeceased }}</span>
          <small>已故</small>
        </div>
      </div>
    </section>

    <section class="toolbar" aria-label="筛选人物">
      <label class="search-box">
        <span>搜索</span>
        <input
          v-model="searchText"
          type="search"
          placeholder="姓名、国家、事件"
          autocomplete="off"
        />
      </label>

      <div class="segmented" role="group" aria-label="生存状态">
        <button
          :class="{ active: statusFilter === 'all' }"
          type="button"
          @click="statusFilter = 'all'"
        >
          全部
        </button>
        <button
          :class="{ active: statusFilter === 'living' }"
          type="button"
          @click="statusFilter = 'living'"
        >
          在世
        </button>
        <button
          :class="{ active: statusFilter === 'deceased' }"
          type="button"
          @click="statusFilter = 'deceased'"
        >
          已故
        </button>
      </div>
    </section>

    <p v-if="isLoading" class="state-message">正在读取人物文档...</p>
    <p v-else-if="loadError" class="state-message error">{{ loadError }}</p>

    <section v-else class="legend-grid" aria-live="polite">
      <article
        v-for="legend in filteredLegends"
        :key="legend.id"
        class="legend-card"
        :class="{ deceased: legend.isDeceased }"
      >
        <div class="legend-heading">
          <div>
            <h2>{{ legend.name }}</h2>
            <p>{{ legend.romanizedName }}</p>
          </div>
          <span class="status-pill">
            <i aria-hidden="true"></i>
            {{ legend.statusLabel }}
          </span>
        </div>

        <dl class="facts">
          <div>
            <dt>国家</dt>
            <dd>{{ legend.country }}</dd>
          </div>
          <div>
            <dt>出生</dt>
            <dd>{{ formatDate(legend.birthDate) }}</dd>
          </div>
          <div>
            <dt>年龄</dt>
            <dd>{{ legend.ageLabel }}</dd>
          </div>
          <div>
            <dt>事件</dt>
            <dd>{{ legend.event }}</dd>
          </div>
        </dl>

        <p class="deed">{{ legend.majorDeed }}</p>

        <a class="source-link" :href="legend.sourceUrl" target="_blank" rel="noreferrer">
          来源
        </a>
      </article>

      <p v-if="filteredLegends.length === 0" class="state-message empty">
        没有匹配的人物。
      </p>
    </section>
  </main>
</template>
