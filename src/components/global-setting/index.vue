<template lang="pug">
.fixed-settings(v-if="!appStore.navbar" @click="setVisible")
  a-button(type="primary")
    template(#icon)
      icon-settings
a-drawer(:width="262" unmount-on-close :visible="globalSettings" :mask-closable="true" @cancel="cancel" :footer="false" )
  template(#title)
    svg.drawer-icon 
      use(href="#setting2")
    | {{ $t('settings.title') }}
  SettingsForm.settings-form
</template>

<script lang="ts" setup>
  import { computed } from 'vue'
  import { Message } from '@arco-design/web-vue'
  import { useI18n } from 'vue-i18n'
  import { useClipboard } from '@vueuse/core'
  import { useAppStore, useDataBaseStore } from '@/store'
  import axios from 'axios'
  import SettingsForm from './settings-form.vue'

  const emit = defineEmits(['cancel'])

  const appStore = useAppStore()
  const dataBaseStore = useDataBaseStore()

  const { t } = useI18n()
  const { copy } = useClipboard()
  const { globalSettings } = storeToRefs(appStore)

  const cancel = () => {
    appStore.updateSettings({ globalSettings: false })
    axios.defaults.baseURL = appStore.host
    if (appStore.codeType === 'sql') {
      dataBaseStore.getTables()
    } else {
      dataBaseStore.getScriptsTable()
    }

    emit('cancel')
  }
  const copySettings = async () => {
    const text = JSON.stringify(appStore.$state, null, 2)
    await copy(text)
    Message.success(t('settings.copySettings.message'))
  }
  const setVisible = () => {
    appStore.updateSettings({ globalSettings: true })
  }

  onMounted(() => {
    axios.defaults.baseURL = appStore.host
  })
</script>

<style scoped lang="less">
  .fixed-settings {
    position: fixed;
    top: 280px;
    right: 0;

    svg {
      font-size: 18px;
      vertical-align: -4px;
    }
  }
</style>
