<template>
  <span class="language-switcher" @click="toggleLocale" role="button" :title="switchTitle" :aria-label="switchTitle">
    <span class="lang-current" :class="{ 'is-zh': isZh, 'is-en': !isZh }">
      <transition name="lang-fade" mode="out-in">
        <span :key="currentLocaleLabel" class="lang-code">{{ currentLocaleLabel }}</span>
      </transition>
    </span>
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
  user-select: none;
  transition: all 0.3s ease;
}

.lang-current {
  width: calc(var(--lang-icon-size, 1.5em) * 1.45);
  height: calc(var(--lang-icon-size, 1.5em) * 1.45);
  min-width: 28px;
  min-height: 28px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  color: var(--lang-icon-color, var(--theme-toggle-color));
  font-weight: 700;
  line-height: 1;
  font-size:16px
}


.lang-code {
  display: inline-block;
}

.lang-fade-enter-active,
.lang-fade-leave-active {
  transition: opacity 0.18s ease, transform 0.18s ease;
}

.lang-fade-enter-from,
.lang-fade-leave-to {
  opacity: 0;
  transform: translateY(3px) scale(0.9);
}
</style>
