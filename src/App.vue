<template>
  <RouterView />
  <DemoSwitch />
  <ToastProvider />
</template>

<script setup>
import { onMounted, onUnmounted, watchEffect } from 'vue'
import { ToastProvider } from 'frappe-ui'
import DemoSwitch from './components/DemoSwitch.vue'
import { useCloudStore } from './stores/cloud'

// Apply the account-wide colour theme. frappe-ui ships dark tokens under
// [data-theme="dark"] on the root, so we just set that attribute. 'system'
// follows the OS and tracks live changes.
const store = useCloudStore()
const media = window.matchMedia('(prefers-color-scheme: dark)')

function applyTheme() {
  const dark = store.theme === 'dark' || (store.theme === 'system' && media.matches)
  document.documentElement.dataset.theme = dark ? 'dark' : 'light'
}

watchEffect(applyTheme) // runs now and whenever store.theme changes
onMounted(() => media.addEventListener('change', applyTheme))
onUnmounted(() => media.removeEventListener('change', applyTheme))
</script>
