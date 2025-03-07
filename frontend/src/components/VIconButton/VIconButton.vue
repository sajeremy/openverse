<template>
  <VButton
    v-bind="buttonProps"
    :aria-label="label"
    size="disabled"
    class="icon-button flex flex-shrink-0 items-center justify-center"
    :class="[buttonSizeClasses, { 'border-1.5': !borderless }]"
    :type="type"
    v-on="$listeners"
  >
    <slot name="default" :icon-size="iconSize" />
    <VIcon
      v-if="iconProps"
      class="pointer-events-none"
      :size="iconSize"
      v-bind="iconProps"
    />
  </VButton>
</template>

<script lang="ts">
import { computed, defineComponent, PropType } from "vue"

import type { ButtonType } from "~/types/button"

import VIcon, { type IconProps } from "~/components/VIcon/VIcon.vue"
import VButton, { type ButtonProps } from "~/components/VButton.vue"

import type { TranslateResult } from "vue-i18n"

const SIZE_MAP = Object.freeze({
  tiny: { icon: 6, button: "w-6 h-6" },
  small: { icon: 6, button: "w-10 h-10" },
  medium: { icon: 6, button: "w-12 h-12" },
  large: { icon: 8, button: "w-14 h-14" },
  "extra-large": { icon: 12, button: "w-20 h-20" },
} as const)
type Size = keyof typeof SIZE_MAP

export default defineComponent({
  name: "VIconButton",
  components: { VIcon, VButton },
  props: {
    /**
     * the size of the button; The size affects both the size of the button
     * itself and the icon inside it.
     */
    size: {
      type: String as PropType<Size>,
      default: "medium",
      validator: (val: string) => Object.keys(SIZE_MAP).includes(val),
    },
    /**
     * props to pass down to the `VIcon` component nested inside the button; See
     * documentation on `VIcon`.
     */
    iconProps: {
      type: Object as PropType<IconProps>,
      required: false,
    },
    /**
     * props to pass down to the `VButton` component nested inside the button;
     * See documentation on `VButton`.
     */
    buttonProps: {
      type: Object as PropType<Omit<ButtonProps, "size">>,
      default: () => ({ variant: "transparent-tx" }),
    },
    /**
     * the label for the button; This is used for accessibility purposes.
     */
    label: {
      type: [String, Object] as PropType<string | TranslateResult>,
      required: true,
    },
    /**
     * whether the button should have a border or not.
     * The `focus-` custom TW classes don't always work well with border.
     */
    borderless: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, { attrs }) {
    const type = (attrs["type"] ?? "button") as ButtonType

    const buttonSizeClasses = computed(() => SIZE_MAP[props.size].button)
    const iconSize = computed(() => SIZE_MAP[props.size].icon)

    return {
      type,

      buttonSizeClasses,
      iconSize,
    }
  },
})
</script>
