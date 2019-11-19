<template>
  <div name="Datatable">
    <slot name="top"></slot>
    <div v-if="$slots.default || HeaderSettings" class="clearfix" style="margin-bottom: 10px">
      <header-settings v-if="HeaderSettings" class="pull-right"
        :columns="columns" :support-backup="supportBackup">
      </header-settings>
      <slot />
    </div>

    <tbl v-bind="$props" />
    
    <div v-if="Pagination" class="row mt-md-3 position-relative" >
      <slot name="bottom-left"></slot>
      <div class="col-12 justify-items-center">
        <pagination class="justify-items-center" :total="total" :query="query" />
      </div>

      <!-- <div class="col-md-7 order-md-2 order-1 mb-md-0 mb-3">
        <slot name="bottom-right"></slot> -->
        <!-- 未用到自選分頁，暫時隱藏 -->
        <!-- <strong>
          {{ $i18nForDatatable('Total') }} {{ total }} {{ $i18nForDatatable(',') }}
        </strong>
        <page-size-select :query="query" :page-size-options="pageSizeOptions"></page-size-select> -->
      <!-- </div> -->
    </div>
  </div>
</template>
<script>
import HeaderSettings from './HeaderSettings/index.vue'
import Tbl from './Table/index.vue'
import Pagination from './Pagination.vue'
import PageSizeSelect from './PageSizeSelect.vue'
import props from './_mixins/props'

export default {
  name: 'Datatable',
  mixins: [props],
  components: { HeaderSettings, Tbl, Pagination, PageSizeSelect },
  created () {
    // init query (make all the properties observable by using `$set`)
    const q = { limit: 10, offset: 0, sort: '', order: '', ...this.query }
    Object.keys(q).forEach(key => { this.$set(this.query, key, q[key]) })
  },
  watch: {
    data: {
      handler (data) {
        const { supportNested } = this
        // support nested components feature with extra magic
        if (supportNested) {
          const MAGIC_FIELD = '__nested__'
          data.forEach(item => {
            if (!item[MAGIC_FIELD]) {
              this.$set(item, MAGIC_FIELD, {
                comp: undefined, // current nested component
                visible: false,
                $toggle (comp, visible) {
                  switch (arguments.length) {
                    case 0:
                      this.visible = !this.visible
                      break
                    case 1:
                      switch (typeof comp) {
                        case 'boolean':
                          this.visible = comp
                          break
                        case 'string':
                        case 'object':
                          this.comp = comp
                          this.visible = !this.visible
                          break
                      }
                      break
                    case 2:
                      this.comp = comp
                      this.visible = visible
                      break
                  }
                }
              })
              if (supportNested === 'accordion') {
                this.$watch(
                  () => item[MAGIC_FIELD],
                  nested => {
                    // only one nested component can be expanded at the same time
                    if (data.filter(item => item[MAGIC_FIELD].visible).length < 2) return

                    data.forEach(item => {
                      if (item[MAGIC_FIELD].visible && item[MAGIC_FIELD] !== nested) {
                        item[MAGIC_FIELD].visible = false
                      }
                    })
                  },
                  { deep: true }
                )
              }
              Object.defineProperty(item, MAGIC_FIELD, { enumerable: false })
            }
          })
        }
      },
      immediate: true
    }
  }
}
</script>
<style>
/* transition effect: fade */
.fade-enter-active, .fade-leave-active {
  transition: opacity .2s;
}
.fade-enter, .fade-leave-active {
  opacity: 0;
}
</style>
