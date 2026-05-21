<template>
    <button
      id="themeToggle"
      type="button"
      @click="handleToggleClick"
      :title="toggleTitle"
      :aria-label="toggleTitle"
    >
      <transition name="icon-fade" mode="out-in">
        <font-awesome-icon 
          :key="themeIcon"
          :icon="themeIcon"
          class="theme-toggle-icon"
        />
      </transition>
    </button>
</template>
  
<script>
export default {
  name: 'ToggleDark',
  data() {
    return {
      isDark: this.$store.getters.useDarkMode,
      isAuto: !this.$store.getters.cusDarkMode,
    };
  },
  computed: {
    themeIcon() {
      if (this.isAuto) return 'circle-half-stroke';
      return this.isDark ? 'moon' : 'sun';
    },
    toggleTitle() {
      if (this.isAuto) return '跟随系统';
      return this.isDark ? '暗色模式' : '亮色模式';
    }
  },
  methods: {
    handleToggleClick() {
      // 三种状态循环：亮色 -> 暗色 -> 跟随系统 -> 亮色
      if (this.isAuto) {
        // 当前是自动模式，切换到亮色
        this.isDark = false;
        this.isAuto = false;
        this.$store.commit('setCusDarkMode', true);
        this.$store.commit('setUseDarkMode', false);
      } else if (!this.isDark) {
        // 当前是亮色，切换到暗色
        this.isDark = true;
        this.$store.commit('setCusDarkMode', true);
        this.$store.commit('setUseDarkMode', true);
      } else {
        // 当前是暗色，切换到跟随系统
        this.isAuto = true;
        this.$store.commit('setCusDarkMode', false);
      }
    }
  }
};
</script>

<style scoped>
#themeToggle {
  border: 1px solid var(--theme-toggle-border, var(--flat-border, rgba(255, 255, 255, 0.18)));
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  width: var(--theme-toggle-size, 40px);
  height: var(--theme-toggle-size, 40px);
  padding: 0;
  border-radius: var(--theme-toggle-radius, 12px);
  background: var(--theme-toggle-control-bg, var(--toolbar-button-bg-color, transparent));
  color: var(--theme-toggle-color);
  box-sizing: border-box;
  box-shadow: none;
  transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease;
}

#themeToggle:hover {
  border-color: var(--flat-border-strong, var(--el-color-primary));
  background: var(--flat-surface-soft, var(--toolbar-button-bg-color, transparent));
}
@media (max-width: 768px) {
  #themeToggle {
    width: var(--theme-toggle-mobile-size, 32px);
    height: var(--theme-toggle-mobile-size, 32px);
  }
}

/* 图标切换过渡效果 */
.icon-fade-enter-active,
.icon-fade-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.icon-fade-enter-from {
  opacity: 0;
  transform: scale(0.88);
}

.icon-fade-leave-to {
  opacity: 0;
  transform: scale(0.88);
}

.icon-fade-enter-to,
.icon-fade-leave-from {
  opacity: 1;
  transform: scale(1) rotate(0deg);
}

.theme-toggle-icon {
  display: inline-block;
  width: 16px;
  font-size: 16px;
  color: currentColor;
}
</style>
