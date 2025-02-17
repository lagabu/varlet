<template>
  <div class="var-index-bar" ref="barEl">
    <slot />
    <ul class="var-index-bar__anchor-list" :style="{ zIndex: zIndex + 2 }">
      <li
        v-for="anchorName in anchorNameList"
        :key="anchorName"
        class="var-index-bar__anchor-item"
        :class="{ 'var-index-bar__anchor-item--active': active === anchorName }"
        :style="{ color: active === anchorName && highlightColor ?  highlightColor : '' }"
        @click="anchorClick(anchorName)"
      >
        {{ anchorName }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { computed, ComputedRef, defineComponent, nextTick, ref, Ref, watch, onMounted, onBeforeUnmount } from 'vue'
import { isPlainObject } from '../utils/shared'
import { getParentScroller, requestAnimationFrame } from '../utils/elements'
import { IndexBarProvider, useIndexAnchors } from './provide'
import { IndexAnchorProvider } from '../index-anchor/provide'
import { props } from './props'

export default defineComponent({
  name: 'VarIndexBar',
  props,
  setup(props) {
    const { length, indexAnchors, bindIndexAnchors } = useIndexAnchors()

    const scrollEl: Ref<HTMLElement | null> = ref(null)
    const scroller: Ref<HTMLElement | Window | null> = ref(null)
    const barEl: Ref<HTMLDivElement | null> = ref(null)
    const anchorNameList: Ref<Array<number | string>> = ref([])
    const active: Ref<number | string | undefined> = ref()

    const sticky: ComputedRef<boolean> = computed(() => props.sticky)
    const stickyOffsetTop: ComputedRef<number> = computed(() => props.stickyOffsetTop)
    const zIndex: ComputedRef<number | string> = computed(() => props.zIndex)

    const indexBarProvider: IndexBarProvider = {
      active,
      sticky,
      stickyOffsetTop,
      zIndex,
    }

    bindIndexAnchors(indexBarProvider)

    const emitEvent = (anchor: IndexAnchorProvider | number | string) => {
      const anchorName = isPlainObject(anchor) ? anchor.name.value : anchor
      if (anchorName === active.value) return

      active.value = anchorName
      props.onChange?.(anchorName)
    }

    const handleScroll = () => {
      const { scrollTop, scrollHeight } = scrollEl.value as HTMLElement
      indexAnchors.forEach((anchor: IndexAnchorProvider, index: number) => {
        const anchorTop = anchor.ownTop.value
        const top = scrollTop - anchorTop + stickyOffsetTop.value

        const distance = index === indexAnchors.length - 1
          ? scrollHeight
          : indexAnchors[index + 1].ownTop.value - anchor.ownTop.value

        if (top >= 0 && top <= distance) emitEvent(anchor)
      })
    }

    const anchorClick = (anchorName: string | number, manualCall?: boolean) => {
      if (manualCall) props.onClick?.(anchorName)
      if (anchorName === active.value) return
      const indexAnchor = indexAnchors.find(({ name }: IndexAnchorProvider) => anchorName === name.value)
      if (!indexAnchor) return
      const top = indexAnchor.ownTop.value
      const { scrollLeft } = scrollEl.value as HTMLDivElement
      ;(scrollEl.value as HTMLElement).scrollTo(scrollLeft, top)
      emitEvent(anchorName)
    }

    // expose
    const scrollTo = (index: number | string) => {
      requestAnimationFrame(() => anchorClick(index, true))
    }

    watch(
      () => length.value,
      () =>
        nextTick(() => {
          indexAnchors.forEach(({ name, setOwnTop }) => {
            if (name.value) anchorNameList.value.push(name.value)
            setOwnTop()
          })
        })
    )

    onMounted(() => {
      scroller.value = getParentScroller(barEl.value as HTMLDivElement)
      scrollEl.value =
        scroller.value === window
          ? (scroller.value as Window).document.documentElement
          : (scroller.value as HTMLElement)
      scroller.value?.addEventListener('scroll', handleScroll)
    })

    onBeforeUnmount(() => {
      scroller.value?.removeEventListener('scroll', handleScroll)
    })

    return {
      barEl,
      active,
      zIndex,
      anchorNameList,
      scrollTo,
      anchorClick,
    }
  },
})
</script>

<style lang="less">
@import './indexBar';
</style>
