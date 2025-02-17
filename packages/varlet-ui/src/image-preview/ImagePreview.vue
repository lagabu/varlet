<template>
  <var-popup
    class="var-image-preview__popup"
    var-image-preview-cover
    transition="var-fade"
    :show="popupShow"
    :overlay="false"
    :close-on-click-overlay="false"
    :lock-scroll="lockScroll"
    :teleport="teleport"
    @open="onOpen"
    @close="onClose"
    @closed="onClosed"
    @opened="onOpened"
    @route-change="onRouteChange"
  >
    <var-swipe
      class="var-image-preview__swipe"
      var-image-preview-cover
      :touchable="canSwipe"
      :indicator="indicator && images.length > 1"
      :initial-index="initialIndex"
      :loop="loop"
      @change="onChange"
      v-bind="$attrs"
    >
      <template #default>
        <var-swipe-item
          class="var-image-preview__swipe-item"
          var-image-preview-cover
          v-for="image in images"
          :key="image"
        >
          <div
            class="var-image-preview__zoom-container"
            :style="{
              transform: `scale(${scale}) translate(${translateX}px, ${translateY}px)`,
              transitionTimingFunction,
              transitionDuration,
            }"
            @touchstart="handleTouchstart"
            @touchmove="handleTouchmove"
            @touchend="handleTouchend"
          >
            <img class="var-image-preview__image" :src="image" :alt="image" />
          </div>
        </var-swipe-item>
      </template>

      <template #indicator="{ index, length }">
        <slot name="indicator" :index="index" :length="length">
          <div class="var-image-preview__indicators" v-if="indicator && images.length > 1">
            {{ index + 1 }} / {{ length }}
          </div>
        </slot>
      </template>
    </var-swipe>

    <slot name="close-icon">
      <var-icon
        class="var-image-preview__close-icon"
        name="close-circle"
        var-image-preview-cover
        v-if="closeable"
        @click="handleCloseClick"
      />
    </slot>
  </var-popup>
</template>

<script lang="ts">
import Swipe from '../swipe'
import SwipeItem from '../swipe-item'
import Icon from '../icon'
import Popup from '../popup'
import { defineComponent, ref, computed, Ref, ComputedRef, watch } from 'vue'
import { props } from './props'
import { toNumber } from '../utils/shared'

type VarTouch = {
  clientX: number
  clientY: number
  timestamp: number
  target: HTMLElement
}

const DISTANCE_OFFSET = 12
const EVENT_DELAY = 200
const ANIMATION_DURATION = 200

export default defineComponent({
  name: 'VarImagePreview',
  components: {
    [Swipe.name]: Swipe,
    [SwipeItem.name]: SwipeItem,
    [Popup.name]: Popup,
    [Icon.name]: Icon,
  },
  inheritAttrs: false,
  props,
  setup(props) {
    const popupShow: Ref<boolean> = ref(false)
    const swipe: Ref<typeof Swipe | null> = ref(null)
    const initialIndex: ComputedRef<number> = computed(() => {
      const { images, current } = props
      const index = images.findIndex((image: string) => image === current)
      return index >= 0 ? index : 0
    })
    const scale: Ref<number> = ref(1)
    const translateX: Ref<number> = ref(0)
    const translateY: Ref<number> = ref(0)
    const transitionTimingFunction: Ref<string | null> = ref(null)
    const transitionDuration: Ref<string | null> = ref(null)
    const canSwipe: Ref<boolean> = ref(true)
    let startTouch: VarTouch | null = null
    let prevTouch: VarTouch | null = null
    let checker: number | null = null

    const getDistance = (touch: VarTouch, target: VarTouch): number => {
      const { clientX: touchX, clientY: touchY } = touch
      const { clientX: targetX, clientY: targetY } = target

      return Math.abs(Math.sqrt((targetX - touchX) ** 2 + (targetY - touchY) ** 2))
    }

    const createVarTouch = (touches: Touch, target: HTMLElement): VarTouch => ({
      clientX: touches.clientX,
      clientY: touches.clientY,
      timestamp: Date.now(),
      target,
    })

    const zoomIn = () => {
      scale.value = toNumber(props.zoom)
      canSwipe.value = false
      prevTouch = null

      window.setTimeout(() => {
        transitionTimingFunction.value = 'linear'
        transitionDuration.value = '0s'
      }, ANIMATION_DURATION)
    }

    const zoomOut = () => {
      scale.value = 1
      translateX.value = 0
      translateY.value = 0
      canSwipe.value = true
      prevTouch = null
      transitionTimingFunction.value = null
      transitionDuration.value = null
    }

    const isDoubleTouch = (currentTouch: VarTouch) => {
      if (!prevTouch) {
        return false
      }

      return (
        getDistance(prevTouch, currentTouch) <= DISTANCE_OFFSET &&
        currentTouch.timestamp - prevTouch.timestamp <= EVENT_DELAY &&
        prevTouch.target === currentTouch.target
      )
    }

    const isTapTouch = (target: HTMLElement) => {
      if (!startTouch || !prevTouch) {
        return false
      }

      return getDistance(startTouch, prevTouch) <= DISTANCE_OFFSET && target === startTouch.target
    }

    const handleTouchend = (event: Event) => {
      checker = window.setTimeout(() => {
        if (isTapTouch(event.target as HTMLElement)) {
          if (scale.value > 1) {
            zoomOut()
            setTimeout(() => props['onUpdate:show']?.(false), ANIMATION_DURATION)
            return
          }

          props['onUpdate:show']?.(false)
        }
        startTouch = null
      }, EVENT_DELAY)
    }

    const handleTouchstart = (event: TouchEvent) => {
      checker && window.clearTimeout(checker)
      const { touches } = event
      const currentTouch: VarTouch = createVarTouch(touches[0], event.currentTarget as HTMLElement)
      startTouch = currentTouch

      if (isDoubleTouch(currentTouch)) {
        scale.value > 1 ? zoomOut() : zoomIn()
        return
      }

      prevTouch = currentTouch
    }

    const handleTouchmove = (event: TouchEvent) => {
      if (!prevTouch) {
        return
      }

      const target = event.currentTarget as HTMLElement
      const { touches } = event
      const currentTouch: VarTouch = createVarTouch(touches[0], target)

      if (scale.value !== 1) {
        const moveX = currentTouch.clientX - prevTouch.clientX
        const moveY = currentTouch.clientY - prevTouch.clientY
        const zoom = toNumber(props.zoom)

        const { offsetWidth: zoomContainerOffsetWidth, offsetHeight: zoomContainerOffsetHeight } = target
        const { offsetWidth, offsetHeight } = target.querySelector('.var-image-preview__image') as HTMLElement
        const limitX = Math.abs(offsetWidth * zoom - zoomContainerOffsetWidth) / (2 * zoom)
        const limitY = Math.abs(zoomContainerOffsetHeight - offsetHeight * zoom) / (2 * zoom)

        if (translateX.value + moveX >= limitX) {
          translateX.value = limitX
        } else if (translateX.value + moveX <= -limitX) {
          translateX.value = -limitX
        } else {
          translateX.value += moveX
        }

        if (translateY.value + moveY >= limitY) {
          translateY.value = limitY
        } else if (translateY.value + moveY <= -limitY) {
          translateY.value = -limitY
        } else {
          translateY.value += moveY
        }
      }

      prevTouch = currentTouch
    }

    const handleCloseClick = () => {
      if (scale.value > 1) {
        zoomOut()
      }
      setTimeout(() => props['onUpdate:show']?.(false), ANIMATION_DURATION)
    }

    watch(
      () => props.current,
      () => swipe.value?.resize()
    )

    watch(
      () => props.show,
      (newValue) => {
        popupShow.value = newValue
      },
      { immediate: true }
    )

    return {
      initialIndex,
      popupShow,
      scale,
      translateX,
      translateY,
      canSwipe,
      transitionTimingFunction,
      transitionDuration,
      handleTouchstart,
      handleTouchmove,
      handleTouchend,
      handleCloseClick,
    }
  },
})
</script>

<style lang="less">
@import '../icon/icon';
@import '../swipe/swipe';
@import '../swipe-item/swipeItem';
@import '../popup/popup';
@import './imagePreview';
</style>
