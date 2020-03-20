<template>
  <a class="admin-focus1 admin-link" href="#" @click.prevent="handleClick" name="HeadSort"  :class="cls" > {{title}}
      <i class="icon-sort-arrow-title"></i> 
  </a>
</template>
<script>
/**
 * Sorting arrows within <th>
 */
export default {
  name: 'HeadSort',
  props: {
    title: { type: String, required: false },
    field: { type: String, required: true },
    query: { type: Object, required: true }
  },
  data: () => ({
    order: ''
  }),
  computed: {
    cls () {
      const { order } = this
      return [
        { '': !order,
          'sorted-asc': order === 'asc',
          'sorted-desc': order === 'desc'
        }
      ]
    }
  },
  watch: {
    query: {
      handler ({ sort: field, order }) {
        this.order = field === this.field ? order : ''
      },
      deep: true,
      immediate: true
    }
  },
  methods: {
    handleClick () {
      const { query, order } = this
      query.sort = this.field
      query.order = this.order = order === 'desc' ? 'asc' : 'desc'
    }
  }
}
</script>
