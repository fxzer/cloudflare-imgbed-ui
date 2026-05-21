<template>
    <div class="tabs">
        <el-segmented
            class="tabs-segmented"
            :model-value="activeTab"
            :options="tabOptions"
            @change="handleTabClick"
        >
            <template #default="{ item }">
                <span class="tabs-segmented-item">
                    <font-awesome-icon :icon="item.icon" class="tabs-segmented-icon"/>
                    <span class="tabs-segmented-label">{{ item.label }}</span>
                </span>
            </template>
        </el-segmented>
        <template v-if="showUtilities">
            <AdminToggleDark/>
            <LanguageSwitcher class="tabs-language-switcher"/>
        </template>
    </div>
</template>

<script>
import AdminToggleDark from './dashboard/AdminToggleDark.vue';
import LanguageSwitcher from '@/components/LanguageSwitcher.vue';

export default {
    name: 'DashboardTabs',
    props: {
        activeTab: {
            type: String,
            default: 'dashboard'
        },
        showUploadTab: {
            type: Boolean,
            default: true
        },
        showUtilities: {
            type: Boolean,
            default: true
        }
    },
    components: {
        AdminToggleDark,
        LanguageSwitcher
    },
    computed: {
        tabOptions() {
            const options = [
                { value: 'dashboard', label: this.$t('dashboardTabs.fileManagement'), icon: 'images' },
                { value: 'customerConfig', label: this.$t('dashboardTabs.userManagement'), icon: 'user-cog' },
                { value: 'systemConfig', label: this.$t('dashboardTabs.systemSettings'), icon: 'cogs' }
            ];
            return options;
        }
    },
    methods: {
        handleTabClick(tab) {
            if (tab === this.activeTab) return;
            const path = tab === 'upload' ? '/' : `/${tab}`;
            this.$router.push(path);
        }
    }
}
</script>

<style scoped>
.tabs {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
}

.tabs-segmented {
    --el-segmented-bg-color: rgba(99, 102, 241, 0.06);
    --el-segmented-item-selected-bg-color: linear-gradient(135deg, #6366f1, #8b5cf6);
    --el-segmented-item-selected-color: #ffffff;
    --el-border-radius-base: 10px;
    min-height: 42px;
    padding: 4px;
    border-radius: 12px;
    background: rgba(99, 102, 241, 0.06);
    border: 1px solid rgba(99, 102, 241, 0.12);
}

.tabs-segmented :deep(.el-segmented__item) {
    min-height: 34px;
    border-radius: 8px;
    transition: all 0.25s ease;
}

.tabs-segmented :deep(.el-segmented__item-selected) {
    background: linear-gradient(135deg, #6366f1, #8b5cf6);
    color: #ffffff;
    border: 1px solid rgba(255, 255, 255, 0.36);
}

.tabs-segmented-item {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 14px;
    font-weight: 500;
    padding: 6px 6px;
}

.tabs-segmented-icon {
    font-size: 0.95em;
    width: 16px;
}

/* 移动端：隐藏文字，仅显示图标，节省横向空间 */
@media (max-width: 768px) {
    .tabs-segmented-label {
        display: none;
    }
    .tabs-segmented-item {
        padding: 4px 3px;
    }
    .tabs-segmented-icon {
        font-size: 1.05em;
    }
}

/* 移动端适配 */
@media (max-width: 768px) {
    .tabs {
        gap: 6px;
    }

    .tabs-language-switcher {
        --lang-icon-size: 1.1em;
        padding: 3px;
    }
}

.tabs-language-switcher {
    --lang-icon-size: 1.3em;
    --lang-icon-color: var(--admin-theme-toggle-color);
    transition: color 0.3s ease;
    padding: 5px;
}

/* el-segmented 样式在 scoped 中定义 */
</style>

<style>
/* 移除旧的 dropdown popper 全局样式 */
</style>
