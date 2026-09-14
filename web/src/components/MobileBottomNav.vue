<script setup lang="ts">
const items = [
  { path: '/', label: '首页', icon: 'i-carbon-home' },
  { path: '/game-mall', label: '商城', icon: 'i-carbon-shopping-cart' },
  { path: '/personal', label: '农场', icon: 'i-carbon-sprout' },
  { path: '/activity', label: '活动', icon: 'i-carbon-events' },
  { path: '/settings', label: '设置', icon: 'i-carbon-settings' },
]
</script>

<template>
  <nav class="mobile-bottom-nav lg:hidden" aria-label="主要导航">
    <RouterLink
      v-for="item in items"
      :key="item.path"
      :to="item.path"
      class="mobile-bottom-nav__item"
      exact-active-class="mobile-bottom-nav__item--active"
    >
      <div :class="item.icon" class="mobile-bottom-nav__icon" />
      <span>{{ item.label }}</span>
    </RouterLink>
  </nav>
</template>

<style scoped>
.mobile-bottom-nav {
  position: fixed;
  z-index: 35;
  right: 0;
  bottom: 0;
  left: 0;
  display: none;
  grid-template-columns: repeat(5, minmax(0, 1fr));
  min-height: 56px;
  padding: 4px 4px calc(4px + env(safe-area-inset-bottom));
  border: none;
  border-top: 1px solid var(--ui-border);
  border-radius: 0;
  background: #f7f7f7;
  box-shadow: none;
}

@media (max-width: 1023px) {
  .mobile-bottom-nav {
    display: grid;
  }
}

.mobile-bottom-nav__item {
  display: flex;
  min-width: 0;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 3px;
  border-radius: 17px;
  color: var(--ui-muted);
  font-size: 11px;
  line-height: 1.1;
  text-decoration: none;
  transition:
    color 0.16s ease,
    background-color 0.16s ease;
}

.mobile-bottom-nav__icon {
  width: 22px;
  height: 22px;
}

.mobile-bottom-nav__item--active {
  color: var(--ui-primary);
  background: transparent;
  box-shadow: none !important;
}
</style>
