<template>
  <!-- TODO 优化为Footer显示 -->
  <div
      v-if="message !== ''"
      class="absolute select-none pointer-events-none z-30 top-0 left-0 w-full h-full flex justify-center items-center text-3xl text-stone-50 bg-stone-800/30">
    {{ message }}
  </div>
  <List
      v-slot="{currentItem}"
      :items="quicklinks"
      :page-size="8"
      :handler-select="(it) => exportQuicklinks.has(it.id as string) ? exportQuicklinks.delete(it.id as string) : exportQuicklinks.add(it.id as string)"
      :lock="lock"
      class="p-2 w-full"
      @keydown.alt="lock = true"
      @keyup.alt="lock = false"
      @keydown.meta="lock = true"
      @keyup.meta="lock = false"
  >
    <Checkbox
        @dblclick="() => {
          const allQuicklinks = quicklinks.map(it => it.id as string)
          if (allQuicklinks.some(it => !exportQuicklinks.has(it))) {
            allQuicklinks.forEach(it => exportQuicklinks.add(it))
          } else {
            exportQuicklinks.clear()
          }
        }"
        :checked="exportQuicklinks.has(currentItem.id as string)">
    </Checkbox>
  </List>
</template>

<script setup lang="ts">
import {onMounted, ref} from "vue";
import {Checkbox, Footer, getPlatform, List, Quicklink, QuicklinkInfo, SimpleItem} from 'sofast-extensions'
import {useMagicKeys, whenever} from "@vueuse/core";

const quicklinks = ref<SimpleItem[]>([])
const exportQuicklinks = ref<Set<string>>(new Set<string>())
const lock = ref(false)
const message = ref('')

const exportQuicklink = () => {
  Quicklink.export(quicklinks.value
      .filter(it => exportQuicklinks.value.has(it.id as string))
      .map(it => it.id as string))
  message.value = '导出快捷链接成功！'
}

const {Ctrl_Enter, Meta_Enter} = useMagicKeys()

whenever(Ctrl_Enter, exportQuicklink)
whenever(Meta_Enter, exportQuicklink)
whenever(message, () => setTimeout(() => message.value = '', 1500))

onMounted(async () => {
  const os = await getPlatform()
  Footer.init([
    {
      align: 'right',
      component: 'ButtonWithKeys',
      props: {
        label: '导出',
        keys: [os === 'windows' ? 'Alt' : os === 'macos' ? 'cmd' : '?', 'enter'],
        toggle: false,
        fn: exportQuicklink,
      }
    },
  ])
  quicklinks.value = (await Quicklink.getAll()).map((it: QuicklinkInfo) => ({
    id: it.id,
    name: it.name,
    icon: it.icon,
  } as SimpleItem))
})
</script>
