<template>
  <div>
    <div class="list-block block-space">
      <div class="container mx-auto">
        <div v-if="getVariant == 'faq'" class="list--faq">
          <div class="flex flex-col md:flex-row">
            <div class="md:w-1/4 md:pr-4">
              <slot name="subtitle"/>
            </div>
            <div class="md:w-3/4 bg-gray-100 p-6 rounded-md">
              <ul class="list__items">
                <li class="list__item mb-6" v-for="item in items" :key="item.id">
                  <h3 class="text-2xl ">{{ item.heading }}</h3>
                  <div class="markup__html opacity-70 prose max-w-none" v-html="item.content"></div>
                </li>
              </ul>
            </div>
          </div>
        </div>
        <div v-else-if="getVariant == 'accordion'" class="list--accordion">
          <slot name="subtitle"/>
          <div class="list__items">
            <div class="list__item mb-2 rounded-md border border-gray-100" :class="{'is-active': item.active}" v-for="(item, index) in items" :key="item.id">
              <dt class="list__title">
                <button @click="toggle(index)" class="accordion-item-trigger block relative hover:opacity-70 text-left w-full px-5 py-2">
                  <h3 class="text-lg ">{{ item.heading }}</h3>
                  <font-awesome-icon :icon="['fas', 'chevron-circle-down']" :class="{'rotate-180 ': item.active}" class="list__icon transform transition duration-300 ease-in-out absolute right-3 top-4" />
                </button>
              </dt>
              <transition
                name="accordion-item"
                @enter="startTransition"
                @after-enter="endTransition"
                @before-leave="startTransition"
                @after-leave="endTransition">
                <dd v-if="item.active" class="accordion-item-details bg-gray-100 px-5 py-2">
                  <div class="markup__html opacity-70 pt-2 prose max-w-none" v-html="item.content"></div>
                </dd>
              </transition>
            </div>
          </div>
        </div>
        <div v-else class="list--default">
          <slot name="subtitle"/>
          <ul v-bind:class="'list__items'">
            <li class="list__item mb-6" v-for="item in items" :key="item.id">
              <h3 class="text-2xl ">{{ item.heading }}</h3>
              <div class="markup__html prose max-w-none" v-html="item.content"></div>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { DruxtEntityMixin } from 'druxt-entity'
import { mapActions } from 'vuex'

export default {
  mixins: [DruxtEntityMixin],
  async fetch() {
    const util = require('util')
    for (const delta in this.fields.items.data.data) { // get fields.items.data.data... from Detected Vue tool --> data
      const item = this.fields.items.data.data[delta]
      const result = await this.getEntity({ id: item.id, type: item.type })
      if (!this.items) this.items = []
      this.items[delta] = {
        id: result.id,
        props: false,
        active: false,
        heading: result.attributes._heading, // get result.attributes._heading... from Detected Vue tool --> attributes
        content: result.attributes._markup.value // get result.attributes._markup.value... from Detected Vue tool --> attributes
      }
    }
  },
  data: () => ({
    items: false,
    showContent: false
  }),
  methods: {
    ...mapActions({
      getEntity: 'druxtRouter/getEntity'
    }),
    toggle(index) {
      this.items[index].active = !this.items[index].active
    },
    
    startTransition(el) {
      el.style.height = el.scrollHeight + 'px'
    },
    
    endTransition(el) {
      el.style.height = ''
    }
  },
  computed: {
    getVariant() {
      let variant = this.entity.attributes.behavior_settings.lecapi_variants.variant
      return variant
    }
  }
}
</script>
<style lang="scss" scoped>
  * {
    box-sizing: border-box;
  }
</style>