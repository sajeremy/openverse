<template>
  <div>
    <!-- eslint-disable vuejs-accessibility/click-events-have-key-events -->
    <div
      ref="triggerContainerRef"
      class="flex w-min items-stretch whitespace-nowrap"
      @click="onTriggerClick"
    >
      <!-- eslint-enable vuejs-accessibility/click-events-have-key-events -->
      <!--
        @slot The trigger, should be a button 99.99% of the time. If you need custom event handling on the trigger button, ensure bubbling is not prevented or else the popover will not open
          @binding {object} a11yProps
          @binding {boolean} visible
      -->
      <slot
        name="trigger"
        :a11y-props="triggerA11yProps"
        :visible="visibleRef"
      />
    </div>
    <VPopoverContent
      v-if="triggerRef"
      :z-index="zIndex"
      :width="width"
      :visible="visibleRef"
      :trigger-element="triggerRef"
      :placement="placement"
      :strategy="strategy"
      :clippable="clippable"
      :hide-on-esc="hideOnEsc"
      :hide-on-click-outside="hideOnClickOutside"
      :auto-focus-on-show="autoFocusOnShow"
      :auto-focus-on-hide="autoFocusOnHide"
      :trap-focus="trapFocus"
      :hide="close"
      :aria-label="label"
      :aria-labelledby="labelledBy"
    >
      <!--
        @slot The content of the popover
          @binding {function} close
      -->
      <slot name="default" :close="close" />
    </VPopoverContent>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, PropType, SetupContext } from "vue"

import { zIndexValidator } from "~/constants/z-indices"

import { useDialogControl } from "~/composables/use-dialog-control"

import VPopoverContent from "~/components/VPopover/VPopoverContent.vue"

import type { Placement, PositioningStrategy } from "@popperjs/core"

export default defineComponent({
  name: "VPopover",
  components: { VPopoverContent },
  /**
   * NB: Most of these technically default to `undefined` so that the underlying `VPopoverContent`
   * default for each of them can take over.
   */
  props: {
    /**
     * Whether the popover should hide when the <kbd>Escape</kbd> key is pressed.
     *
     * @default true
     */
    hideOnEsc: { type: Boolean, default: undefined },
    /**
     * Whether the popover should hide when a click happens outside the popover content,
     * excluding the trigger. When the trigger is clicked and the popover is open, nothing
     * will happen.
     *
     * @default true
     */
    hideOnClickOutside: { type: Boolean, default: undefined },
    /**
     * Whether the popover content should automatically receive focus when the popover
     * opens.
     *
     * @default true
     */
    autoFocusOnShow: { type: Boolean, default: undefined },
    /**
     * Whether the trigger should automatically receive focus when the popover closes.
     *
     * @default true
     */
    autoFocusOnHide: { type: Boolean, default: undefined },
    /**
     * The placement of the popover relative to the trigger. Should be one of the options
     * for `placement` passed to popper.js.
     *
     * @see https://popper.js.org/docs/v2/constructors/#options
     *
     * @default 'bottom'
     */
    placement: {
      type: String as PropType<Placement>,
    },
    /**
     * The positioning strategy of the popover. If your reference element is in a fixed container
     * use the fixed strategy; otherwise use the default, absolute strategy.
     *
     * @see https://popper.js.org/docs/v2/constructors/#strategy
     *
     * @default 'absolute'
     */
    strategy: {
      type: String as PropType<PositioningStrategy>,
    },
    /**
     * The label of the popover content. Must be provided if `labelledBy` is empty.
     *
     * @default undefined
     */
    label: { type: String },
    /**
     * The id of the element labelling the popover content. Must be provided if `label` is empty.
     *
     * @default undefined
     */
    labelledBy: { type: String },
    /**
     * the z-index to apply to the popover content
     */
    zIndex: {
      type: [Number, String],
      default: "popover", // named z-index
      validator: zIndexValidator,
    },
    /**
     * Whether the popover height should be clipped and made scrollable
     * if the window height is too small.
     *
     * @default false
     */
    clippable: { type: Boolean, default: false },
    /**
     * Whether the popover should trap focus. This means that when the popover is open,
     * the user cannot tab out of the popover content. This is useful for ensuring that
     * the popover content is accessible.
     * @default true
     */
    trapFocus: { type: Boolean, default: undefined },
    /**
     * Optional Tailwind class for fixed width popover content.
     *
     * @default undefined
     */
    width: { type: String },
  },
  emits: [
    /**
     * Fires when the popover opens, regardless of reason. There are no extra parameters.
     */
    "open",
    /**
     * Fires when the popover closes, regardless of reason. There are no extra parameters.
     */
    "close",
  ],
  setup(_, { emit }) {
    const visibleRef = ref(false)
    const triggerContainerRef = ref<HTMLElement | null>(null)

    const triggerRef = computed(() =>
      triggerContainerRef.value?.firstChild
        ? (triggerContainerRef.value.firstChild as HTMLElement)
        : undefined
    )

    const { close, open, onTriggerClick, triggerA11yProps } = useDialogControl({
      visibleRef,
      emit: emit as SetupContext["emit"],
    })

    return {
      open,
      close,
      visibleRef,
      triggerContainerRef,
      triggerRef,
      onTriggerClick,
      triggerA11yProps,
    }
  },
})
</script>
