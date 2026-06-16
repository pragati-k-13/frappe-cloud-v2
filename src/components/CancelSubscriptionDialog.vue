<template>
  <Dialog v-model:open="open" size="sm">
    <template #title>
      <span class="text-xl font-semibold text-ink-gray-9">{{ stepTitle }}</span>
    </template>

    <!-- Step 1 — what happens -->
    <div v-if="step === 1" class="space-y-4">
      <div class="flex items-start gap-3 rounded-lg border border-outline-red-1 bg-surface-red-1 p-3">
        <span class="lucide-triangle-alert mt-0.5 size-4 shrink-0 text-ink-red-3" />
        <p class="text-sm text-ink-red-4">
          This stops billing on every server at once and <span class="font-medium">can't be undone</span>.
        </p>
      </div>

      <div>
        <div class="text-sm font-medium text-ink-gray-8">Here's exactly what happens</div>
        <ul class="mt-2 space-y-2.5">
          <li v-for="item in whatHappens" :key="item.text" class="flex items-start gap-2.5 text-sm text-ink-gray-6">
            <span class="mt-0.5 size-4 shrink-0 text-ink-gray-5" :class="item.icon" />
            <span>{{ item.text }}</span>
          </li>
        </ul>
      </div>
    </div>

    <!-- Step 2 — done -->
    <div v-else class="py-2 text-center">
      <span class="mx-auto grid size-12 place-items-center rounded-full bg-surface-green-2 text-ink-green-3">
        <span class="lucide-check size-6" />
      </span>
      <p class="mx-auto mt-3 max-w-sm text-sm text-ink-gray-6">
        {{ result.removed.length }} server{{ result.removed.length === 1 ? '' : 's' }} and their sites were stopped. You won't be charged again.
      </p>
      <p class="mx-auto mt-2 max-w-sm text-p-sm text-ink-gray-5">
        A final invoice for this cycle's usage will be emailed to {{ billingEmail }}. Backups are kept for 30 days.
      </p>
    </div>

    <template #actions>
      <div class="flex justify-end gap-2">
        <template v-if="step === 1">
          <Button label="Back" @click="open = false" />
          <Button variant="solid" theme="red" label="Delete and cancel subscription" @click="confirm" />
        </template>
        <Button v-else variant="solid" label="Done" @click="open = false" />
      </div>
    </template>
  </Dialog>
</template>

<script setup>
import { computed, reactive, ref, watch } from 'vue'
import { Button, Dialog, toast } from 'frappe-ui'
import { useCloudStore } from '../stores/cloud'
import { inr } from '../utils/format'

const open = defineModel('open', { type: Boolean, default: false })
const store = useCloudStore()

const step = ref(1)
const result = reactive({ removed: [], all: false })

const billingEmail = computed(
  () => store.billingProfile.invoiceRecipient || store.billingProfile.billingEmail || store.user.email,
)

const whatHappens = computed(() => {
  const rows = [
    { icon: 'lucide-receipt', text: 'Usage up to today is charged: a final invoice goes to your payment method on file.' },
  ]
  if (store.walletBalance > 0) {
    rows.push({ icon: 'lucide-wallet', text: `Unused wallet credit (${inr(store.walletBalance)}) is non-refundable and will be forfeited.` })
  }
  rows.push({ icon: 'lucide-archive', text: 'Backups are retained for 30 days, in case you need to restore later.' })
  return rows
})

const stepTitle = computed(() => (step.value === 1 ? 'Cancel subscription' : 'Subscription cancelled'))

function confirm() {
  const r = store.cancelSubscription()
  result.removed = r.removed
  result.all = r.all
  step.value = 2
  toast.success('Subscription cancelled')
}

// Reset to a clean step-1 state every time the dialog opens.
watch(open, (isOpen) => {
  if (isOpen) step.value = 1
})
</script>
