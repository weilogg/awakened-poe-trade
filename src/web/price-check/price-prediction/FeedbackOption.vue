<template>
  <ui-popover tag-name="div" trigger="click" boundary="#price-window">
    <template #target>
      <button class="bg-gray-700 px-2 opacity-25" :class="{ 'rounded-l': option === 'low', 'rounded-r': option === 'high' }"
        >{{ option }}</button>
    </template>
    <template #content>
      <form @submit.prevent="submit" class="w-64 p-2">
        <div>{{ text }}</div>
        <textarea v-if="option !== 'fair'"
          v-model="feedbackText"
          placeholder="Why do you think so? (Not required)"
          rows="5" class="w-full bg-gray-700 text-gray-100 p-1"></textarea>
        <button class="btn" type="submit">Send feedback</button>
      </form>
    </template>
  </ui-popover>
</template>

<script lang="ts">
import { computed, defineComponent, PropType, ref } from 'vue'
import { ParsedItem } from '@/parser'
import { sendFeedback } from './poeprices'

export default defineComponent({
  emits: ['sent'],
  props: {
    option: {
      type: String as PropType<'fair' | 'low' | 'high'>,
      required: true
    },
    prediction: {
      type: Object as PropType<{ min: number, max: number, currency: 'chaos' | 'exalt' }>,
      required: true
    },
    item: {
      type: Object as PropType<ParsedItem>,
      required: true
    }
  },
  setup (props, ctx) {
    const feedbackText = ref('')

    const text = computed(() => {
      if (props.option === 'low') {
        return 'Predicted price is too low.'
      } else if (props.option === 'high') {
        return 'Predicted price is too high.'
      } else {
        return 'Predicted price is fair.'
      }
    })
    
    function submit () {
      ctx.emit('sent')
      sendFeedback({
        text: feedbackText.value,
        option: props.option
      }, props.prediction, props.item)
    }

    return {
      feedbackText,
      text,
      submit
    }
  }
})
</script>
