<template>
  <div v-if="nav.currentPage.value > 1" class="ark-hud">
    <span class="ark-hud-stage">
      STAGE {{ String(nav.currentPage.value).padStart(2, '0') }}/{{ String(nav.total.value).padStart(2, '0') }}
    </span>
    <span class="ark-hud-bricks" aria-hidden="true">
      <span
        v-for="i in nav.total.value"
        :key="i"
        class="ark-hud-brick"
        :class="{ on: i <= nav.currentPage.value }"
      />
    </span>
  </div>
</template>

<script setup>
import { useNav } from '@slidev/client'

const nav = useNav()
</script>

<style scoped>
.ark-hud {
  position: fixed;
  left: 1.1rem;
  bottom: 0.7rem;
  z-index: 30;
  display: flex;
  align-items: center;
  gap: 0.7rem;
  pointer-events: none;
}

.ark-hud-stage {
  font-family: 'VT323', monospace;
  font-size: 1.05rem;
  letter-spacing: 0.08em;
  color: rgba(0, 229, 255, 0.75);
  text-shadow: 0 0 8px rgba(0, 229, 255, 0.3);
}

.ark-hud-bricks {
  display: flex;
  gap: 3px;
}

.ark-hud-brick {
  width: 9px;
  height: 5px;
  border-radius: 1px;
  background: rgba(255, 255, 255, 0.12);
  transition: background 200ms ease-out, box-shadow 200ms ease-out;
}

.ark-hud-brick.on {
  background: var(--ark-cyan, #00e5ff);
  box-shadow: 0 0 5px rgba(0, 229, 255, 0.6);
}
</style>
