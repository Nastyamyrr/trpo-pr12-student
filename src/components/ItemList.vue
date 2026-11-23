<script>
import ItemCard from './ItemCard.vue'

export default {
  name: 'ItemList',
  components: {
    ItemCard
  },
  props: {
    items: {
      type: Array,
      required: true
    }
  },
  emits: ['remove', 'select', 'status-change']
}
</script>

<template>
  <section class="items-grid" aria-label="Задачи">
    <ItemCard
      v-for="item in items"
      :key="item.id"
      :item="item"
      @remove="$emit('remove', $event)"
      @select="$emit('select', $event)"
      @status-change="$emit('status-change', $event)"
    />
  </section>
</template>
