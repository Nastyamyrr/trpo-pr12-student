<script>
import itemsData from './data/items.json'
import filtersData from './data/filters.json'
import uiStatesData from './data/ui_states.json'
import FilterPanel from './components/FilterPanel.vue'
import ItemList from './components/ItemList.vue'

export default {
  name: 'App',
  components: {
    FilterPanel,
    ItemList
  },
  data() {
    return {
      items: [],
      filters: filtersData,
      currentFilter: 'all',
      searchQuery: '',
      newItemTitle: '',
      formError: '',
      loading: true,
      error: '',
      demoStates: uiStatesData,
      showStateTools: false
    }
  },
  computed: {
    visibleItems() {
      const query = this.searchQuery.trim().toLowerCase()

      return this.items.filter((item) => {
        const matchesFilter = this.currentFilter === 'all'
          || item.status === this.currentFilter
        const matchesSearch = item.title.toLowerCase().includes(query)
        return matchesFilter && matchesSearch
      })
    },
    totalCount() {
      return this.items.length
    },
    completedCount() {
      return this.items.filter((item) => item.status === 'done').length
    },
    activeCount() {
      return this.items.filter((item) => item.status === 'doing').length
    }
  },
  mounted() {
    this.loadItems()
  },
  methods: {
    loadItems() {
      this.loading = true
      this.error = ''

      // Dataset подключён как локальные JSON-модули и читается при инициализации.
      try {
        this.items = itemsData.map((item) => ({ ...item }))
        this.loading = false
      } catch (error) {
        this.error = 'Не удалось загрузить список задач.'
        this.loading = false
      }
    },
    addItem() {
      const title = this.newItemTitle.trim()

      if (!title) {
        this.formError = 'Введите название задачи.'
        return
      }

      this.items.unshift({
        id: Date.now(),
        title,
        status: 'new',
        priority: 1
      })
      this.newItemTitle = ''
      this.formError = ''
      this.currentFilter = 'all'
      this.searchQuery = ''
    },
    removeItem(id) {
      this.items = this.items.filter((item) => item.id !== id)
    },
    selectItem(id) {
      const selected = this.items.find((item) => item.id === id)
      if (selected) {
        selected.status = selected.status === 'done' ? 'doing' : 'done'
      }
    },
    changeItemStatus(payload) {
      const item = this.items.find((entry) => entry.id === payload.id)
      if (item) {
        item.status = payload.status
      }
    },
    setDemoState(state) {
      this.error = ''
      this.loading = false

      if (state === 'loading') {
        this.loading = true
      } else if (state === 'error') {
        const errorState = this.demoStates.find((entry) => entry.state === 'error')
        this.error = errorState ? errorState.message : 'Тестовая ошибка'
      } else if (state === 'empty') {
        this.items = []
      } else if (state === 'success') {
        this.items = itemsData.map((item) => ({ ...item }))
      }
    },
    resetDemo() {
      this.loadItems()
      this.searchQuery = ''
      this.currentFilter = 'all'
    }
  }
}
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <div class="topbar-inner">
        <div class="brand">Практическая работа 12</div>
        <span class="course-label">ТРПО</span>
      </div>
    </header>

    <main class="container">
      <section class="hero">
        <span class="eyebrow">Vue 3 · Options API</span>
        <h1>Менеджер учебных задач</h1>
        <p class="muted">
          Каталог задач с фильтрацией, компонентами, проверкой формы и интерфейсными состояниями.
          Данные загружаются из dataset.
        </p>
        <button class="primary-button" type="button" @click="showStateTools = !showStateTools">
          {{ showStateTools ? 'Скрыть проверку состояний' : 'Проверить состояния' }}
        </button>
      </section>

      <section class="stats-grid" aria-label="Сводка задач">
        <article class="stat-card">
          <span class="stat-label">Всего задач</span>
          <strong>{{ totalCount }}</strong>
          <span class="stat-note">Записей в списке</span>
        </article>
        <article class="stat-card">
          <span class="stat-label">В работе</span>
          <strong>{{ activeCount }}</strong>
          <span class="stat-note">Требуют внимания</span>
        </article>
        <article class="stat-card">
          <span class="stat-label">Готово</span>
          <strong>{{ completedCount }}</strong>
          <span class="stat-note">Завершённые задачи</span>
        </article>
      </section>

      <section v-if="showStateTools" class="state-tools panel">
        <div>
          <h2>Проверка интерфейсных состояний</h2>
          <p class="muted">Сценарии взяты из файла ui_states.json.</p>
        </div>
        <div class="state-buttons">
          <button class="small-button" type="button" @click="setDemoState('loading')">Loading</button>
          <button class="small-button" type="button" @click="setDemoState('error')">Error</button>
          <button class="small-button" type="button" @click="setDemoState('empty')">Empty</button>
          <button class="small-button" type="button" @click="setDemoState('success')">Success</button>
          <button class="small-button secondary" type="button" @click="resetDemo">Сбросить</button>
        </div>
      </section>

      <section class="panel create-panel">
        <div>
          <h2>Новая задача</h2>
          <p class="muted">Добавьте запись в список. Название обязательно.</p>
        </div>
        <form class="create-form" @submit.prevent="addItem">
          <div class="field-wrap">
            <input
              v-model="newItemTitle"
              type="text"
              placeholder="Например, подготовить отчёт"
              aria-label="Название новой задачи"
              @input="formError = ''"
            />
            <span v-if="formError" class="field-error">{{ formError }}</span>
          </div>
          <button class="primary-button" type="submit">Добавить задачу</button>
        </form>
      </section>

      <FilterPanel
        :filters="filters"
        :current-filter="currentFilter"
        :search-query="searchQuery"
        @filter-change="currentFilter = $event"
        @search-change="searchQuery = $event"
      />

      <section class="list-heading">
        <div>
          <h2>Список задач</h2>
          <p class="muted">Показано: {{ visibleItems.length }} из {{ totalCount }}</p>
        </div>
        <span class="result-chip">{{ visibleItems.length }} записей</span>
      </section>

      <div v-if="loading" class="state-card">
        <span class="loader"></span>
        <strong>Загрузка задач...</strong>
        <p class="muted">Подготавливаем данные для отображения.</p>
      </div>

      <div v-else-if="error" class="state-card error-state">
        <strong>Не удалось отобразить задачи</strong>
        <p>{{ error }}</p>
        <button class="small-button" type="button" @click="resetDemo">Повторить</button>
      </div>

      <div v-else-if="visibleItems.length === 0" class="state-card empty-state">
        <div class="empty-icon">○</div>
        <strong>{{ totalCount === 0 ? 'Список пока пуст' : 'Ничего не найдено' }}</strong>
        <p class="muted">
          {{ totalCount === 0 ? 'Добавьте первую задачу с помощью формы выше.' : 'Измените фильтр или поисковый запрос.' }}
        </p>
        <button v-if="totalCount > 0" class="small-button" type="button" @click="searchQuery = ''; currentFilter = 'all'">
          Сбросить фильтры
        </button>
      </div>

      <ItemList
        v-else
        :items="visibleItems"
        @remove="removeItem"
        @select="selectItem"
        @status-change="changeItemStatus"
      />
    </main>

    <footer class="footer">Практическая работа №12 · Vue 3 Options API</footer>
  </div>
</template>
