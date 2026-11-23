<script>
export default {
  name: 'ItemCard',
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  emits: ['remove', 'select', 'status-change'],
  computed: {
    statusTitle() {
      const names = {
        new: 'Новая',
        doing: 'В работе',
        done: 'Готово'
      }
      return names[this.item.status] || 'Неизвестно'
    },
    priorityTitle() {
      const names = {
        1: 'Низкий',
        2: 'Средний',
        3: 'Высокий'
      }
      return names[this.item.priority] || 'Не указана'
    }
  },
  methods: {
    toggleStatus() {
      const nextStatus = this.item.status === 'done' ? 'doing' : 'done'
      this.$emit('status-change', {
        id: this.item.id,
        status: nextStatus
      })
    }
  }
}
</script>

<template>
  <article class="item-card">
    <div class="item-card-top">
      <span class="item-id">Задача #{{ item.id }}</span>
      <span class="status-badge" :class="`status-${item.status}`">{{ statusTitle }}</span>
    </div>
    <h3>{{ item.title }}</h3>
    <div class="item-meta">
      <span>Приоритет: <strong>{{ priorityTitle }}</strong></span>
    </div>
    <div class="item-actions">
      <button class="small-button" type="button" @click="toggleStatus">
        {{ item.status === 'done' ? 'Вернуть в работу' : 'Завершить' }}
      </button>
      <button class="text-button danger" type="button" @click="$emit('remove', item.id)">
        Удалить
      </button>
    </div>
  </article>
</template>
