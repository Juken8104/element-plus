<template>
  <transition
    :name="ns.b('fade')"
    @before-leave="onClose"
    @after-leave="$emit('destroy')"
  >
    <div
      v-show="visible"
      :id="id"
      role="alert"
      :class="[
        ns.b(),
        { [ns.m(type)]: type && !icon },
        ns.is('center', center),
        ns.is('closable', showClose),
        customClass,
      ]"
      :style="customStyle"
      @mouseenter="clearTimer"
      @mouseleave="startTimer"
    >
      <div style="display: flex">
        <el-badge
          v-if="repeatNum > 1"
          :value="repeatNum"
          :type="badgeType"
          :class="ns.e('badge')"
        />
        <el-icon v-if="iconComponent" :class="[ns.e('icon'), typeClass]">
          <component :is="iconComponent" />
        </el-icon>
        <slot>
          <p v-if="!dangerouslyUseHTMLString" :class="ns.e('content')">
            {{ message }}
            <span
              v-show="showCollapse"
              style="cursor: pointer"
              @click.stop="collapse = !collapse"
              >more</span
            >
          </p>
          <!-- Caution here, message could've been compromised, never use user's input as message -->
          <p v-else :class="ns.e('content')" v-html="message" />
        </slot>
        <el-icon v-if="showClose" :class="ns.e('closeBtn')" @click.stop="close">
          <close />
        </el-icon>
      </div>
      <div>
        <el-collapse-transition>
          <div
            v-if="collapse"
            :id="ns.b(`content-${id}`)"
            :class="ns.be('item', 'wrap')"
            role="tabpanel"
            :aria-hidden="!collapse"
            :aria-labelledby="ns.b(`head-${id}`)"
          >
            <div :class="ns.e('content')">{{ collapseMessage }}</div>
          </div>
        </el-collapse-transition>
      </div>
    </div>
  </transition>
</template>
<script lang="ts">
import { computed, defineComponent, onMounted, ref, watch } from 'vue'
import { useEventListener, useTimeoutFn } from '@vueuse/core'
import { TypeComponents, TypeComponentsMap } from '@element-plus/utils'
import { EVENT_CODE } from '@element-plus/constants'
import ElBadge from '@element-plus/components/badge'
import { ElIcon } from '@element-plus/components/icon'
import ElCollapseTransition from '@element-plus/components/collapse-transition'

import { useNamespace } from '@element-plus/hooks'
import { messageEmits, messageProps } from './message'
import type { BadgeProps } from '@element-plus/components/badge'

import type { CSSProperties } from 'vue'

export default defineComponent({
  name: 'ElMessage',

  components: {
    ElBadge,
    ElIcon,
    ElCollapseTransition,
    ...TypeComponents,
  },

  props: messageProps,
  emits: messageEmits,

  setup(props) {
    const collapse = ref(false)
    const ns = useNamespace('message')
    const visible = ref(false)
    const badgeType = ref<BadgeProps['type']>(
      props.type ? (props.type === 'error' ? 'danger' : props.type) : 'info'
    )
    let stopTimer: (() => void) | undefined = undefined

    const typeClass = computed(() => {
      const type = props.type
      return { [ns.bm('icon', type)]: type && TypeComponentsMap[type] }
    })

    const iconComponent = computed(() => {
      return props.icon || TypeComponentsMap[props.type] || ''
    })

    const customStyle = computed<CSSProperties>(() => ({
      top: `${props.offset}px`,
      zIndex: props.zIndex,
    }))

    function startTimer() {
      if (props.duration > 0) {
        ;({ stop: stopTimer } = useTimeoutFn(() => {
          if (visible.value) close()
        }, props.duration))
      }
    }

    function clearTimer() {
      stopTimer?.()
    }

    function close() {
      visible.value = false
    }

    function keydown({ code }: KeyboardEvent) {
      if (code === EVENT_CODE.esc) {
        // press esc to close the message
        if (visible.value) {
          close()
        }
      } else {
        startTimer() // resume timer
      }
    }

    onMounted(() => {
      startTimer()
      visible.value = true
    })

    watch(
      () => props.repeatNum,
      () => {
        clearTimer()
        startTimer()
      }
    )

    useEventListener(document, 'keydown', keydown)

    return {
      ns,
      typeClass,
      iconComponent,
      customStyle,
      visible,
      badgeType,

      close,
      clearTimer,
      startTimer,
      collapse,
    }
  },
})
</script>
