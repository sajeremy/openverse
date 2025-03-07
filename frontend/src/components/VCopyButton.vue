<template>
  <VButton
    :id="id"
    type="button"
    variant="filled-dark"
    size="small"
    class="label-bold flex-shrink-0"
    :data-clipboard-target="el"
  >
    <span v-if="!success">
      {{ $t("media-details.reuse.copy-license.copy-text") }}
    </span>
    <span v-else>
      {{ $t("media-details.reuse.copy-license.copied") }}
    </span>
  </VButton>
</template>

<script lang="ts">
import Clipboard from "clipboard"

import { defineComponent, onBeforeUnmount, onMounted, ref } from "vue"

import VButton from "~/components/VButton.vue"

export default defineComponent({
  name: "VCopyButton",
  components: { VButton },
  props: {
    el: {
      type: String,
      required: true,
    },
    id: {
      type: String,
      required: true,
    },
  },
  emits: ["copied", "copy-failed"],
  setup(props, { emit }) {
    const clipboard = ref<Clipboard | null>(null)
    const success = ref(false)

    function setFocusOnButton(): void {
      const button = document.getElementById(props.id)
      if (button) {
        button.focus()
      }
    }

    const onCopySuccess = (e: Clipboard.Event) => {
      success.value = true
      emit("copied")

      setTimeout(() => {
        success.value = false
      }, 2000)

      e.clearSelection()

      /* Set the focus back on the button */
      setFocusOnButton()
    }
    const onCopyError = (e: Clipboard.Event) => {
      emit("copy-failed")
      e.clearSelection()

      /* Restore focus on the button */
      setFocusOnButton()
    }

    onMounted(() => {
      clipboard.value = new Clipboard(`#${props.id}`)
      clipboard.value.on("success", onCopySuccess)
      clipboard.value.on("error", onCopyError)
    })

    onBeforeUnmount(() => clipboard.value?.destroy())

    return {
      clipboard,
      success,
    }
  },
})
</script>
