<template>
  <span
    class="language-switcher"
    @click="toggleLocale"
    role="button"
    :title="switchTitle"
    :aria-label="switchTitle"
  >
    <transition name="lang-fade" mode="out-in">
      <span :key="currentLocaleLabel" class="lang-current" :class="{ 'is-zh': isZh, 'is-en': !isZh }">
        <span class="lang-code">{{ currentLocaleLabel }}</span>
      </span>
    </transition>
  </span>
</template>

<script>
import { setLocale } from '@/locales'

export default {
  name: 'LanguageSwitcher',
  computed: {
    isZh() {
      return this.$i18n.locale === 'zh-CN'
    },
    currentLocaleLabel() {
      return this.isZh ? '中' : 'EN'
    },
    currentLocaleIcon() {
      return this.isZh ? 'language' : 'globe'
    },
    switchTitle() {
      return this.isZh ? '切换到 English' : 'Switch to 中文'
    }
  },
  methods: {
    toggleLocale() {
      const next = this.isZh ? 'en' : 'zh-CN'
      setLocale(next)
    }
  }
}
</script>

<style scoped>
.language-switcher {
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: var(--lang-control-size, 40px);
  height: var(--lang-control-size, 40px);
  border: 1px solid var(--lang-control-border, var(--flat-border, rgba(255, 255, 255, 0.18)));
  border-radius: var(--lang-control-radius, 12px);
  background: var(--lang-control-bg, var(--toolbar-button-bg-color, transparent));
  color: var(--lang-icon-color, var(--theme-toggle-color));
  box-sizing: border-box;
  user-select: none;
  transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease;
}

.language-switcher:hover {
  border-color: var(--flat-border-strong, var(--el-color-primary));
  background: var(--flat-surface-soft, var(--toolbar-button-bg-color, transparent));
}

.lang-current {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  width: 100%;
  height: 100%;
  font-weight: 600;
  line-height: 1;
  font-size: 13px;
}

.lang-symbol {
  width: 14px;
  font-size: var(--lang-icon-size, 14px);
}

.lang-code {
  display: inline-block;
  min-width: 14px;
  font-size: 12px;
  text-align: center;
}

.lang-fade-enter-active,
.lang-fade-leave-active {
  transition: opacity 0.18s ease, transform 0.18s ease;
}

.lang-fade-enter-from,
.lang-fade-leave-to {
  opacity: 0;
  transform: translateY(4px) scale(0.9);
}
</style>
