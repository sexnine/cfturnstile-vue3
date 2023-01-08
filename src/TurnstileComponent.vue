<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue'

export default defineComponent({
  name: "TurnstileComponent",
  props: {
    sitekey: {
      type: String,
      required: true
    },
    options: {
      type: Object,
      required: false
    }
  },
  emits: {
    verify: (response: string) => {
        if (response !== null && response !== "") return true
        else return false
    },
    expire: null,
    fail: null
  },
  setup(props, context) {
    const turnstileBox = ref(null)
    onMounted(() => {
      console.debug("Turnstile component mounted.")

      if (window.turnstile === undefined) {
        console.debug("Turnstile doesn't exist on window.")

        const script = document.createElement('script')
        script.src = 'https://challenges.cloudflare.com/turnstile/v0/api.js?onload=onloadTurnstileCallback'
        script.async = true
        script.defer = true
        document.head.appendChild(script)

        window.onloadTurnstileCallback = () => {
          renderTurnstile()
        }

        return;
      }

      renderTurnstile()
    })

    function renderTurnstile() {
      console.debug("Turnstile rendering...")

      window.turnstile?.render("#turnstile-box", {
        sitekey: props.sitekey,
        callback: (response: string) => context.emit('verify', response),
        'expired-callback': context.emit('expire'),
        'error-callback': context.emit('fail'),
        ...props.options
      })
    }
  }

})
</script>

<template>
  <div ref="turnstileBox" id="turnstile-box"></div>
</template>

<style scoped>

</style>
