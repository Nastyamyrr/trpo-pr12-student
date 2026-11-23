<script>
export default {
  name: 'FilterPanel',
  props: {
    filters: {
      type: Array,
      required: true
    },
    currentFilter: {
      type: String,
      required: true
    },
    searchQuery: {
      type: String,
      default: ''
    }
  },
  emits: ['filter-change', 'search-change']
}
</script>

<template>
  <section class="filter-panel panel">
    <div class="filter-top">
      <div>
        <h2>Фильтры</h2>
        <p class="muted">Выберите статус или найдите задачу по названию.</p>
      </div>
      <input
        class="search-input"
        type="search"
        :value="searchQuery"
        placeholder="Поиск задачи..."
        aria-label="Поиск задачи"
        @input="$emit('search-change', $event.target.value)"
      />
    </div>
    <div class="filter-buttons">
      <button
        v-for="filter in filters"
        :key="filter.id"
        type="button"
        class="filter-button"
        :class="{ active: currentFilter === filter.id }"
        @click="$emit('filter-change', filter.id)"
      >
        {{ filter.title }}
      </button>
    </div>
  </section>
</template>
