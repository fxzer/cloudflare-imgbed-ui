<template>
<div class="sidebar-container" ref="sidebar" :class="{ 'is-collapsed': isCollapse }" :style="sidebarStyle">
    <div class="menu-list">
        <div v-for="group in menuGroups" :key="group.titleKey" class="menu-section">
            <div class="menu-section-label">{{ $t(group.titleKey) }}</div>
            <div
                v-for="item in group.items"
                :key="item.index"
                class="menu-item"
                :class="{ 'is-active': activeIndex === item.index }"
                @click="handleSelect(item.index)"
            >
                <font-awesome-icon :icon="item.icon" class="menu-icon" />
                <span class="menu-text">{{ $t(item.titleKey) }}</span>
            </div>
        </div>
    </div>

    <div class="toggle-button" @click="toggleCollapse">
        <font-awesome-icon :icon="isCollapse ? 'angle-double-right' : 'angle-double-left'"></font-awesome-icon>
    </div>
</div>
</template>

<script>
export default {
name: 'SysConfigTabs',
props: {
    activeIndex: {
        type: String,
        default: 'status'
    },
    isCollapse: {
        type: Boolean,
        default: false
    }
},
data() {
    return {
        expandedWidth: null,
        menuGroups: [
            {
                titleKey: 'sysConfigTabs.groupRuntime',
                items: [
                    { index: 'status', icon: 'chart-bar', titleKey: 'sysConfigTabs.systemStatus' }
                ]
            },
            {
                titleKey: 'sysConfigTabs.groupUpload',
                items: [
                    { index: 'upload', icon: 'cloud-upload-alt', titleKey: 'sysConfigTabs.channelManagement' },
                    { index: 'uploadSettings', icon: 'sliders-h', titleKey: 'sysConfigTabs.uploadSettings' }
                ]
            },
            {
                titleKey: 'sysConfigTabs.groupAccess',
                items: [
                    { index: 'security', icon: 'shield-alt', titleKey: 'sysConfigTabs.securitySettings' },
                    { index: 'page', icon: 'pager', titleKey: 'sysConfigTabs.pageSettings' }
                ]
            },
            {
                titleKey: 'sysConfigTabs.groupMore',
                items: [
                    { index: 'others', icon: 'puzzle-piece', titleKey: 'sysConfigTabs.otherSettings' }
                ]
            }
        ]
    };
},
computed: {
    collapsedWidth() {
	        return window.innerWidth <= 768 ? 52 : 56;
    },
	    sidebarStyle() {
	        if (this.isCollapse) {
	            return {
	                width: `var(--sys-sidebar-collapsed-width, ${this.collapsedWidth}px)`,
	                maxWidth: `var(--sys-sidebar-collapsed-width, ${this.collapsedWidth}px)`
	            };
	        }
	        return {
	            width: 'var(--sys-sidebar-width, 208px)',
	            maxWidth: 'var(--sys-sidebar-width, 208px)'
	        };
	    }
},
watch: {
    '$i18n.locale'() {
        this.$nextTick(() => this.measureWidth());
    }
},
methods: {
    toggleCollapse() {
        this.$emit('update:isCollapse', !this.isCollapse);
    },
    checkMobile() {
        const isMobile = window.innerWidth <= 768;
        this.$emit('update:isCollapse', isMobile);
    },
    handleSelect(index) {
        this.$emit('update:activeIndex', index);
    },
    measureWidth() {
        const el = this.$refs.sidebar;
        if (!el) return;
        // Temporarily expand to measure natural content width
        const prevWidth = el.style.width;
        const prevTransition = el.style.transition;
        const wasCollapsed = el.classList.contains('is-collapsed');
        el.style.transition = 'none';
        if (wasCollapsed) {
            el.classList.remove('is-collapsed');
        }
        el.style.width = 'auto';
        // Force reflow to apply changes
        void el.offsetWidth;
        const natural = el.scrollWidth;
        // Restore original state
        el.style.width = prevWidth;
        if (wasCollapsed) {
            el.classList.add('is-collapsed');
        }
        void el.offsetWidth;
        el.style.transition = prevTransition;
        this.expandedWidth = natural;
    },
},
mounted() {
    this.checkMobile();
    this.$nextTick(() => this.measureWidth());
    window.addEventListener('resize', this.checkMobile);
},
beforeDestroy() {
    window.removeEventListener('resize', this.checkMobile);
}
};
</script>

<style scoped>
.sidebar-container {
    display: flex;
    flex-direction: column;
    position: fixed;
	    top: 100px;
	    bottom: 24px;
	    left: var(--sys-sidebar-left, 10vw);
	    z-index: 2001;
	    width: var(--sys-sidebar-width, 208px);
	    max-width: var(--sys-sidebar-width, 208px);
    background: var(--flat-surface);
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
    border: 1px solid var(--flat-border);
    border-radius: 12px;
    box-shadow: none;
	    transition: width 0.3s ease, max-width 0.3s ease, border-color 0.2s ease;
    overflow: hidden;
}

@media (max-width: 768px) {
    .sidebar-container {
      left: var(--sys-sidebar-left, 8px);
    }
}
	.sidebar-container.is-collapsed {
	    width: var(--sys-sidebar-collapsed-width, 56px);
	    max-width: var(--sys-sidebar-collapsed-width, 56px);
	}

/* 深色模式 */
html.dark .sidebar-container {
    background: var(--flat-surface);
    border: 1px solid var(--flat-border);
    box-shadow: none;
}

.sidebar-container:hover {
    border-color: var(--flat-border-strong);
    box-shadow: none;
}

html.dark .sidebar-container:hover {
    box-shadow: none;
}

.menu-list {
    flex: 1;
	    padding: 12px 8px;
    overflow-y: auto;
}

	.menu-section + .menu-section {
	    margin-top: 12px;
	    padding-top: 12px;
    border-top: 1px solid var(--flat-border);
}

	.menu-section-label {
	    padding: 0 12px 8px;
    color: var(--flat-text-muted);
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 0;
    text-align: left;
    white-space: nowrap;
}

.menu-item {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    padding: 12px 12px 12px 0;
    margin: 4px 0;
	    height: 44px;
    box-sizing: border-box;
    border-radius: 10px;
    cursor: pointer;
    transition: background 0.2s ease, color 0.2s ease;
    color: var(--admin-container-color, #333);
    gap: 0;
    overflow: hidden;
}

.sidebar-container.is-collapsed .menu-section {
    margin-top: 0;
    padding-top: 0;
    border-top: 0;
}

.sidebar-container.is-collapsed .menu-section + .menu-section {
    margin-top: 4px;
}

.sidebar-container.is-collapsed .menu-section-label {
    display: none;
}

.menu-item:hover {
    background: var(--flat-surface-soft);
}

html.dark .menu-item:hover {
    background: var(--flat-surface-soft);
}

.menu-item.is-active {
    background: var(--flat-primary-soft);
    color: var(--flat-primary);
    border: 1px solid rgba(37, 99, 235, 0.18);
}

html.dark .menu-item.is-active {
    background: var(--flat-primary-soft);
}

.menu-icon {
    width: 40px;
    min-width: 40px;
    font-size: 16px;
    flex-shrink: 0;
    text-align: center;
}

.menu-text {
    font-size: 14px;
    font-weight: 500;
    white-space: nowrap;
    overflow: hidden;
    opacity: 1;
    transition: opacity 0.2s ease 0.05s, max-width 0.25s ease;
}

.sidebar-container.is-collapsed .menu-text {
    opacity: 0;
    max-width: 0;
    transition: opacity 0.1s ease, max-width 0.2s ease;
}

.toggle-button {
    padding: 12px;
    text-align: center;
    cursor: pointer;
    border-top: 1px solid var(--flat-border);
    transition: all 0.2s ease;
    color: var(--admin-container-color, #333);
}

html.dark .toggle-button {
    border-top: 1px solid var(--flat-border);
}

.toggle-button:hover {
    background: var(--flat-surface-soft);
}

html.dark .toggle-button:hover {
    background: var(--flat-surface-soft);
}

/* 移动端 */
@media (max-width: 768px) {
	    .sidebar-container {
	        top: 88px;
	        bottom: 12px;
	        left: var(--sys-sidebar-left, 8px);
	        width: var(--sys-sidebar-width, 172px);
	        max-width: var(--sys-sidebar-width, 172px);
	    }

	    .sidebar-container.is-collapsed {
	        width: var(--sys-sidebar-collapsed-width, 52px);
	        max-width: var(--sys-sidebar-collapsed-width, 52px);
	    }
    
    .menu-icon {
        width: 34px;
        min-width: 34px;
    }
}
</style>
