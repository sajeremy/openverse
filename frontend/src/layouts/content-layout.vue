<template>
  <div
    class="app flex grid min-h-[100dvh] min-h-screen grid-rows-[auto,1fr,auto] bg-white"
    :class="[
      isDesktopLayout ? 'desktop' : 'mobile',
      breakpoint,
      { 'has-sidebar': isSidebarVisible },
      isSidebarVisible
        ? 'grid-cols-[1fr_var(--filter-sidebar-width)]'
        : 'grid-cols-1',
    ]"
  >
    <div class="header-el sticky top-0 z-40 block bg-white">
      <VTeleportTarget name="skip-to-content" :force-destroy="true" />
      <VBanners />
      <template v-if="isSearchHeader">
        <VHeaderDesktop v-if="isDesktopLayout" class="h-20 bg-white" />
        <VHeaderMobile v-else class="h-20 bg-white" />
      </template>
      <VHeaderInternal
        v-else
        class="h-20 bg-white"
        :class="{ 'border-b-dark-charcoal-20': isHeaderScrolled }"
      />
    </div>

    <aside
      v-if="isSidebarVisible"
      class="sidebar fixed end-0 z-10 mt-[80px] h-[calc(100dvh-80px)] h-[calc(100vh-80px)] overflow-y-auto border-s border-dark-charcoal-20 bg-dark-charcoal-06"
    >
      <VSearchGridFilter class="px-10 pb-10 pt-8" @close="closeSidebar" />
    </aside>

    <div class="main-page flex h-full w-full min-w-0 flex-col justify-between">
      <Nuxt />
      <VFooter
        :mode="isSearchHeader ? 'content' : 'internal'"
        class="border-t border-dark-charcoal-20 bg-white"
      />
    </div>

    <VModalTarget class="modal" />
    <VGlobalAudioSection />
  </div>
</template>
<script lang="ts">
import { computed, defineComponent, onMounted, provide, ref, watch } from "vue"
import { useContext } from "@nuxtjs/composition-api"
import { PortalTarget as VTeleportTarget } from "portal-vue"

import { useWindowScroll } from "~/composables/use-window-scroll"
import { useMatchSearchRoutes } from "~/composables/use-match-routes"
import { useLayout } from "~/composables/use-layout"

import { useUiStore } from "~/stores/ui"
import { useSearchStore } from "~/stores/search"
import { useFeatureFlagStore } from "~/stores/feature-flag"

import { IsHeaderScrolledKey, IsSidebarVisibleKey } from "~/types/provides"

import VBanners from "~/components/VBanner/VBanners.vue"
import VFooter from "~/components/VFooter/VFooter.vue"
import VModalTarget from "~/components/VModal/VModalTarget.vue"
import VGlobalAudioSection from "~/components/VGlobalAudioSection/VGlobalAudioSection.vue"
import VSearchGridFilter from "~/components/VFilters/VSearchGridFilter.vue"

/**
 * This is the ContentLayout: the search page, the single result page,
 * and the content pages.
 * It has white background and is scrollable. It can also have a sidebar.
 */
export default defineComponent({
  name: "ContentLayout",
  components: {
    VBanners,
    VHeaderDesktop: () => import("~/components/VHeader/VHeaderDesktop.vue"),
    VHeaderInternal: () => import("~/components/VHeader/VHeaderInternal.vue"),
    VHeaderMobile: () =>
      import("~/components/VHeader/VHeaderMobile/VHeaderMobile.vue"),
    VFooter,
    VModalTarget,
    VTeleportTarget,
    VGlobalAudioSection,
    VSearchGridFilter,
  },
  setup() {
    const { app } = useContext()
    const uiStore = useUiStore()
    const searchStore = useSearchStore()

    const featureStore = useFeatureFlagStore()
    onMounted(() => {
      featureStore.initFromSession()
    })

    const { updateBreakpoint } = useLayout()

    /**
     * Update the breakpoint value in the cookie on mounted.
     * The Pinia state might become different from the cookie state if, for example, the cookies were saved when the screen was `sm`,
     * and then a page is opened on SSR on a `lg` screen.
     */
    onMounted(() => {
      updateBreakpoint()
    })

    const { matches: isSearchRoute } = useMatchSearchRoutes()

    const nuxtError = computed(() => app.nuxt.err)

    const isSearchHeader = computed(
      () => !nuxtError.value && isSearchRoute.value
    )

    const isDesktopLayout = computed(() => uiStore.isDesktopLayout)
    const breakpoint = computed(() => uiStore.breakpoint)

    /**
     * Filters sidebar is visible only on desktop layouts
     * on search result pages for supported search types.
     */
    const isSidebarVisible = computed(
      () =>
        isSearchRoute.value &&
        searchStore.searchTypeIsSupported &&
        uiStore.isFilterVisible &&
        isDesktopLayout.value
    )

    const closeSidebar = () => {
      uiStore.setFiltersState(false)
    }

    const isHeaderScrolled = ref(false)
    const { isScrolled: isMainContentScrolled, y: scrollY } = useWindowScroll()
    watch([isMainContentScrolled], ([isMainContentScrolled]) => {
      isHeaderScrolled.value = isMainContentScrolled
    })
    const showScrollButton = computed(() => scrollY.value > 70)

    provide("showScrollButton", showScrollButton)
    provide(IsHeaderScrolledKey, isHeaderScrolled)
    provide(IsSidebarVisibleKey, isSidebarVisible)

    return {
      isHeaderScrolled,
      isDesktopLayout,
      isSidebarVisible,
      isSearchRoute,
      isSearchHeader,
      breakpoint,

      closeSidebar,
    }
  },
  head() {
    return this.$nuxtI18nHead({
      addSeoAttributes: true,
      addDirAttribute: true,
    })
  },
})
</script>

<style scoped>
.has-sidebar .sidebar {
  width: var(--filter-sidebar-width);
}
.app {
  grid-template-areas: "header" "main" "global-audio";
}
.header-el {
  grid-area: header;
}
.main-page {
  grid-area: main;
}
.sidebar {
  grid-area: sidebar;
}
.has-sidebar.app {
  grid-template-areas: "header header" "main sidebar";
}
</style>
