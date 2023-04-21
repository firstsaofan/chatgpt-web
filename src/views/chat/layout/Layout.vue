<template>
  <div class="h-full">
    <div class="h-full dark:bg-[#24272e] transition-all" :class="[isMobile ? 'p-0' : 'p-4']">
      <div class="h-full" :class="getMobileClass">
        <NLayout class="z-40 transition" :class="getContainerClass" has-sider>
          <Sider />
          <NTabs type="line" animated>
            <NTabPane name="AI聊天模型" tab="AI聊天模型">
              <div class="scrollable" >
                <RouterView v-slot="{ Component, route }">
                  <component :is="Component" :key="route.fullPath" />
                </RouterView>
              </div>
            </NTabPane>
            <NTabPane name="AI绘图" tab="AI绘图">
              <ImageView/>
            </NTabPane>
          </NTabs>
          <Helped />
        </NLayout>
      </div>
      <Permission :visible="needPermission" />
    </div>
  </div>
</template>

<script setup lang='ts'>
import { computed } from 'vue'
import { NLayout, NLayoutContent ,NCard,
  NTabs,
  NTabPane,
  NInputGroup,
  NImage} from 'naive-ui'
import { useRouter } from 'vue-router'
import Sider from './sider/index.vue'
import Helped from './helped/index.vue'
import Permission from './Permission.vue'
import ImageView from './image/index.vue'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { useAppStore, useAuthStore, useChatStore } from '@/store'

const router = useRouter()
const appStore = useAppStore()
const chatStore = useChatStore()
const authStore = useAuthStore()

router.replace({ name: 'Chat', params: { uuid: chatStore.active } })

const { isMobile } = useBasicLayout()

const collapsed = computed(() => appStore.siderCollapsed)

const needPermission = computed(() => !!authStore.session?.auth && !authStore.token)

const getMobileClass = computed(() => {
  if (isMobile.value)
    return ['rounded-none', 'shadow-none']
  return ['border', 'rounded-md', 'shadow-md', 'dark:border-neutral-800']
})

const getContainerClass = computed(() => {
  return [
    'h-full',
    { 'pl-[260px]': !isMobile.value && !collapsed.value },
  ]
})
</script>

<style scoped>
.scrollable {
  height: calc(100vh - 10px);
  overflow-y: scroll;
}
</style>
