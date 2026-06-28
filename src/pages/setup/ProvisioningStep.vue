<template>
  <OnboardingShell :step="4">
    <div class="flex flex-col py-2">
      <!-- Header: small server badge on the left, title + subtext beside it -->
      <div class="flex items-center gap-3">
        <span class="relative grid size-10 shrink-0 place-items-center">
          <span class="provision-glow absolute inset-0 rounded-xl bg-surface-gray-3" />
          <span class="relative grid size-10 place-items-center rounded-xl bg-surface-gray-2 ring-1 ring-outline-gray-2">
            <span class="lucide-server size-5 text-ink-gray-7" />
          </span>
        </span>
        <div class="min-w-0">
          <h1 class="text-base font-semibold text-ink-gray-9">Setting up your server</h1>
          <p class="text-p-sm text-ink-gray-5">Usually about 2 minutes.</p>
        </div>
      </div>

      <!-- A normal progress bar, filling steadily as it provisions -->
      <Progress :value="progressValue" size="lg" class="mt-5 w-full" />

      <!-- A rotating tip turns the wait into a head start -->
      <div class="mt-4 min-h-[2.5rem]">
        <Transition name="tip" mode="out-in">
          <p :key="tipIndex" class="text-p-sm text-ink-gray-6">{{ tips[tipIndex] }}</p>
        </Transition>
      </div>

      <!-- Reassure: they don't have to wait here -->
      <p class="mt-1 flex items-center gap-1.5 text-p-sm text-ink-gray-5">
        <span class="lucide-mail size-3.5 shrink-0" />
        Leave anytime — we'll email you when it's ready.
      </p>

      <Button variant="ghost" size="sm" label="Skip the wait (demo)" class="-ml-2 mt-4 self-start" @click="finish" />
    </div>
  </OnboardingShell>
</template>

<script setup>
import { onBeforeUnmount, onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import { Button, Progress } from 'frappe-ui'
import OnboardingShell from '../../components/OnboardingShell.vue'
import { useCloudStore } from '../../stores/cloud'

const store = useCloudStore()
const router = useRouter()

// Punchy, reassuring things a new account should know — one at a time.
const tips = [
  'Push to GitHub and it deploys automatically.',
  'Daily backups run on their own — restore in seconds.',
  'Add a custom domain anytime; we handle the SSL.',
  'Resize your plan in a minute, with no downtime.',
  'Invite your team with exactly the access they need.',
]
const tipIndex = ref(0)
const progressValue = ref(0)

let provisionTimer = null
let tipTimer = null
let progressTimer = null

onMounted(() => {
  if (!store.server) {
    router.replace('/setup/account')
    return
  }
  if (store.server.status === 'active') {
    router.replace('/app')
    return
  }
  // Fill steadily toward ~96% over the ~30s wait — never hit 100 until it's done.
  progressTimer = setInterval(() => {
    progressValue.value = Math.min(96, progressValue.value + 0.64)
  }, 200)
  tipTimer = setInterval(() => {
    tipIndex.value = (tipIndex.value + 1) % tips.length
  }, 4500)
  // Mockup: hold on this screen ~30s so the loading + tips are actually visible.
  provisionTimer = setTimeout(finish, 30000)
})

onBeforeUnmount(() => {
  clearInterval(progressTimer)
  clearInterval(tipTimer)
  clearTimeout(provisionTimer)
})

// No "you're live" stop in between — the site itself is the payoff.
function finish() {
  clearInterval(progressTimer)
  clearInterval(tipTimer)
  clearTimeout(provisionTimer)
  progressValue.value = 100
  store.completeProvisioning()
  router.replace('/app')
}
</script>

<style scoped>
/* A soft halo breathes out behind the badge — a calmer pulse than animate-ping. */
.provision-glow {
  animation: provision-glow 2.6s cubic-bezier(0.23, 1, 0.32, 1) infinite;
}
@keyframes provision-glow {
  0% {
    transform: scale(1);
    opacity: 0.5;
  }
  100% {
    transform: scale(1.5);
    opacity: 0;
  }
}

/* Tip swap: old slides up, new rises from below; blur masks the crossfade so the
   eye reads one moving line instead of two overlapping ones. */
.tip-enter-active,
.tip-leave-active {
  transition:
    opacity 320ms cubic-bezier(0.23, 1, 0.32, 1),
    transform 320ms cubic-bezier(0.23, 1, 0.32, 1),
    filter 320ms cubic-bezier(0.23, 1, 0.32, 1);
}
.tip-enter-from {
  opacity: 0;
  transform: translateY(6px);
  filter: blur(3px);
}
.tip-leave-to {
  opacity: 0;
  transform: translateY(-6px);
  filter: blur(3px);
}

/* Reduced motion: keep the meaning (fade), drop the movement. */
@media (prefers-reduced-motion: reduce) {
  .provision-glow {
    animation: none;
    opacity: 0.35;
  }
  .tip-enter-active,
  .tip-leave-active {
    transition: opacity 200ms ease;
  }
  .tip-enter-from,
  .tip-leave-to {
    transform: none;
    filter: none;
  }
}
</style>
