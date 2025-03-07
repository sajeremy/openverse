<template>
  <div
    class="flex flex-col gap-1 rounded-sm border bg-white"
    :class="bordered ? 'border-dark-charcoal-20 p-4 shadow-el-2' : 'border-tx'"
    data-testid="recent-searches"
  >
    <div
      class="flex flex-row items-center justify-between py-2"
      :class="{ 'pe-2': !bordered }"
    >
      <!-- Left margin to align with the text of recent searches. -->
      <span class="category mx-2 my-1">
        {{ $t("recent-searches.heading") }}
      </span>
      <VButton
        v-show="entries.length"
        variant="transparent-gray"
        class="label-bold"
        size="small"
        :aria-label="$t('recent-searches.clear.label').toString()"
        @click="handleClear"
      >
        {{ $t("recent-searches.clear.text") }}
      </VButton>
    </div>

    <ul
      v-if="entries.length"
      id="recent-searches-list"
      role="listbox"
      :aria-label="$t('recent-searches.heading').toString()"
    >
      <!-- eslint-disable vuejs-accessibility/interactive-supports-focus Combobox descendants only have visual focus. -->
      <!-- eslint-disable vuejs-accessibility/click-events-have-key-events Key events handled by input field of combobox. -->
      <li
        v-for="(entry, idx) in entries"
        :id="`option-${idx}`"
        :key="idx"
        role="option"
        class="description-regular my-1 rounded-sm border-1.5 p-2 hover:bg-dark-charcoal-10"
        :class="idx === selectedIdx ? 'border-pink' : 'border-tx'"
        :aria-selected="idx === selectedIdx"
        @click="handleClick(idx)"
      >
        {{ entry }}
      </li>
      <!-- eslint-enable -->
    </ul>
    <span v-else class="description-regular mx-2 my-3">
      {{ $t("recent-searches.none") }}
    </span>

    <span class="caption-regular mx-2 my-3 text-dark-charcoal-70">
      {{ $t("recent-searches.disclaimer") }}
    </span>
  </div>
</template>

<script lang="ts">
import { defineComponent, type PropType } from "vue"

import { defineEvent } from "~/types/emits"

import VButton from "~/components/VButton.vue"

/**
 * List the recent searches of the user allowing them to go back to a previous
 * search. These searches are saved locally and never shared with the server.
 */
export default defineComponent({
  name: "VRecentSearches",
  components: { VButton },
  props: {
    /**
     * the list of saved past searches
     */
    entries: {
      type: Array as PropType<string[]>,
      default: () => [],
    },
    /**
     * the index of the currently selected entry
     */
    selectedIdx: {
      type: Number,
    },
    /**
     * the desktop popover is bordered, and the mobile element is not
     */
    bordered: {
      type: Boolean,
      default: true,
    },
  },
  emits: {
    select: defineEvent<[number]>(),
    clear: defineEvent(),
  },
  setup(_, { emit }) {
    const handleClick = (idx: number) => {
      emit("select", idx)
    }
    const handleClear = () => {
      emit("clear")
    }

    return {
      handleClick,
      handleClear,
    }
  },
})
</script>
