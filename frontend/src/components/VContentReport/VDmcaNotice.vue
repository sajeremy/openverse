<template>
  <i18n
    class="dmca-notice"
    path="media-details.content-report.form.dmca.note"
    tag="p"
  >
    <template #form>
      <VLink
        :aria-label="$t('media-details.content-report.form.dmca.form')"
        :href="DMCA_FORM_URL"
        class="text-pink hover:underline"
        @click="$emit('click')"
        >{{ $t("media-details.content-report.form.dmca.form") }}</VLink
      >
    </template>
    <template #source>
      <VLink :href="foreignLandingUrl" class="text-pink hover:underline">{{
        provider
      }}</VLink>
    </template>
  </i18n>
</template>

<script>
import { defineComponent } from "vue"

import { DMCA_FORM_URL } from "~/constants/content-report"
import { defineEvent } from "~/types/emits"

import VLink from "~/components/VLink.vue"

export default defineComponent({
  name: "VDmcaNotice",
  components: { VLink },
  props: {
    /**
     * the foreign landing URL for the media item
     */
    foreignLandingUrl: {
      type: String,
      required: true,
    },
    /**
     * the name of the provider of the media item
     */
    provider: {
      type: String,
      required: true,
    },
  },
  emits: {
    click: defineEvent(),
  },
  setup() {
    return {
      DMCA_FORM_URL,
    }
  },
})
</script>
