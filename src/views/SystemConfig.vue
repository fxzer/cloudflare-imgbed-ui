<template>
    <div class="container">
        <el-header>
            <div class="header-content">
                <DashboardTabs activeTab="systemConfig" :show-upload-tab="false" :show-utilities="false"></DashboardTabs>
                <div class="header-action">
                    <AdminToggleDark class="header-theme-toggle"/>
                    <LanguageSwitcher class="header-language-switcher"/>
                    <el-tooltip :disabled="disableTooltip" :content="$t('dashboardTabs.fileUpload')" placement="bottom">
                        <font-awesome-icon icon="upload" class="header-icon" @click="goUpload"></font-awesome-icon>
                    </el-tooltip>
                    <el-tooltip :disabled="disableTooltip" :content="$t('sysConfig.logout')" placement="bottom">
                        <font-awesome-icon icon="sign-out-alt" class="header-icon" @click="handleLogout"></font-awesome-icon>
                    </el-tooltip>
                </div>
            </div>
        </el-header>
        <SysConfigTabs
            v-model:activeIndex="activeIndex"
            v-model:isCollapse="isSidebarCollapse"
        />
        <!-- 根据锚点动态渲染子页面 -->
        <component :is="currentComponent" :class="['main-container', { 'collapsed': isSidebarCollapse }]" />
    </div>
</template>
<script>
import DashboardTabs from '@/components/DashboardTabs.vue';
import AdminToggleDark from '@/components/dashboard/AdminToggleDark.vue';
import LanguageSwitcher from '@/components/LanguageSwitcher.vue';
import SysConfigTabs from '@/components/config/SysConfigTabs.vue';
import SysCogStatus from '@/components/config/SysCogStatus.vue';
import SysCogUpload from '@/components/config/SysCogUpload.vue';
import SysCogUploadPreferences from '@/components/config/SysCogUploadPreferences.vue';
import SysCogSecurity from '@/components/config/SysCogSecurity.vue';
import SysCogPage from '@/components/config/SysCogPage.vue';
import SysCogOthers from '@/components/config/SysCogOthers.vue';
import backgroundManager from '@/mixins/backgroundManager';

export default {
    name: 'SystemConfig',
    mixins: [backgroundManager],
    data() {
        return {
            activeIndex: 'status',
            isSidebarCollapse: false
        }
    },
    watch: {
        // 监听锚点变化
        '$route.hash': {
            immediate: true,
            handler(newHash) {
                this.activeIndex = newHash.replace('#', '');
                window.scrollTo(0, 0); // 滚动到页面顶部
            }
        },
        activeIndex(newIndex) {
            // 更新锚点
            const hash = `#${newIndex}`;
            this.$router.push({ hash });
        }
    },
    components: {
        DashboardTabs,
        AdminToggleDark,
        LanguageSwitcher,
        SysConfigTabs,
        SysCogStatus,
        SysCogUpload,
        SysCogUploadPreferences,
        SysCogSecurity,
        SysCogPage,
        SysCogOthers
    },
    computed: {
        disableTooltip() {
            return window.innerWidth < 768;
        },
        // 根据锚点动态返回对应的组件
        currentComponent() {
            const hash = this.$route.hash.replace('#', '');
            switch (hash) {
                case 'status':
                    return SysCogStatus;
                case 'upload':
                    return SysCogUpload;
                case 'uploadSettings':
                    return SysCogUploadPreferences;
                case 'security':
                    return SysCogSecurity;
                case 'page':
                    return SysCogPage;
                case 'others':
                    return SysCogOthers;
                default:
                    return SysCogStatus;
            }
        }
    },
    methods: {
        goUpload() {
            this.$router.push('/');
        },
        handleLogout() {
            const url = process.env.NODE_ENV === 'production' ? '/api/auth/logout' : '/api/api/auth/logout';
            fetch(url, {
                method: 'POST',
                credentials: 'include',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ authType: 'admin' })
            }).finally(() => {
                this.$store.commit('setAdminLoggedIn', false);
                this.$router.push('/adminLogin');
            });
        },
        // 设置默认锚点
        setDefaultHash() {
            const defaultHash = '#status'; // 默认锚点
            window.location.hash = defaultHash;
            this.activeIndex = defaultHash.replace('#', '');
        },
    },
    mounted() {
        // 初始化背景图
        this.initializeBackground('adminBkImg', '.container', false, true);

        // 如果 URL 中没有锚点，则设置默认锚点
        if (!window.location.hash) {
            this.setDefaultHash();
        }
    },
}
</script>
<style scoped>
.container {
    --sys-sidebar-left: 10vw;
    --sys-sidebar-width: 208px;
    --sys-sidebar-collapsed-width: 56px;
    --sys-content-gap: 24px;
    --sys-content-right: 10vw;
    background: var(--admin-container-bg-color);
    min-height: 100vh;
    font-family: 'Arial', sans-serif;
    color: var(--admin-container-color);
    margin: 0;
    padding: 0;
    overflow-x: hidden;
}

.header-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 24px;
    /* macOS 风格毛玻璃效果 */
    background: rgba(255, 255, 255, 0.72);
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
    /* 顶部边框形成玻璃边缘光泽 */
    border: 1px solid rgba(255, 255, 255, 0.3);
    border-top: 1px solid rgba(255, 255, 255, 0.5);
    /* 悬浮阴影效果 */
    box-shadow: none;
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    border-radius: 16px;
    position: fixed;
    top: 8px;
    left: 50%;
    transform: translateX(-50%);
    width: 80vw;
    z-index: 2001;
    min-height: 48px;
}


/* 深色模式毛玻璃效果 */
html.dark .header-content {
    background: var(--flat-surface);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-top: 1px solid rgba(255, 255, 255, 0.12);
    box-shadow: none;
}

@media (max-width: 768px) {
    .container {
        --sys-sidebar-left: 8px;
        --sys-sidebar-width: 172px;
        --sys-sidebar-collapsed-width: 52px;
        --sys-content-gap: 16px;
        --sys-content-right: 12px;
    }

    .header-content {
        flex-direction: column;
        top: 8px;
        width: 90vw;
        border-radius: 14px;
        padding: 8px 12px;
        gap: 4px;
    }
    
    .header-icon {
        font-size: 1em;
    }
}

.header-content:hover {
    background: var(--flat-surface);
    box-shadow: none;
    transform: translateX(-50%);
}

html.dark .header-content:hover {
    background: var(--flat-surface);
    box-shadow: none;
}

.header-icon {
    font-size: 1.5em;
    cursor: pointer;
    transition: all 0.3s ease;
    color: var(--admin-container-color);
    outline: none;
}

.header-icon:hover {
    color: #B39DDB; /* 使用柔和的淡紫色 */
    transform: none;
}

.header-action {
    display: flex;
    align-items: center;
    gap: 16px;
}

.header-theme-toggle,
.header-language-switcher,
.header-icon {
    flex: 0 0 auto;
}

.header-language-switcher {
    --lang-control-size: 36px;
    --lang-control-radius: 10px;
    --lang-icon-size: 14px;
    --lang-icon-color: var(--admin-theme-toggle-color);
    --lang-control-bg: var(--flat-surface);
    --lang-control-border: var(--flat-border);
}

.main-container {
  margin-top: 40px;
  margin-left: calc(var(--sys-sidebar-left) + var(--sys-sidebar-width) + var(--sys-content-gap));
  margin-right: var(--sys-content-right);
  padding-bottom: 24px;
  box-sizing: border-box;
  transition: margin-left 0.3s ease, width 0.3s ease;
  width: calc(100vw - var(--sys-sidebar-left) - var(--sys-sidebar-width) - var(--sys-content-gap) - var(--sys-content-right));
  max-width: calc(100vw - var(--sys-sidebar-left) - var(--sys-sidebar-width) - var(--sys-content-gap) - var(--sys-content-right));
}

.main-container.collapsed {
  margin-left: calc(var(--sys-sidebar-left) + var(--sys-sidebar-collapsed-width) + var(--sys-content-gap));
  width: calc(100vw - var(--sys-sidebar-left) - var(--sys-sidebar-collapsed-width) - var(--sys-content-gap) - var(--sys-content-right));
  max-width: calc(100vw - var(--sys-sidebar-left) - var(--sys-sidebar-collapsed-width) - var(--sys-content-gap) - var(--sys-content-right));
}

/* 移动端不压缩内容，但让出折叠侧边栏宽度 */
@media (max-width: 768px) {
  .main-container,
  .main-container.collapsed {
    margin-top: 96px;
    width: calc(100vw - var(--sys-sidebar-left) - var(--sys-sidebar-collapsed-width) - var(--sys-content-gap) - var(--sys-content-right));
    max-width: calc(100vw - var(--sys-sidebar-left) - var(--sys-sidebar-collapsed-width) - var(--sys-content-gap) - var(--sys-content-right));
    margin-left: calc(var(--sys-sidebar-left) + var(--sys-sidebar-collapsed-width) + var(--sys-content-gap));
    margin-right: var(--sys-content-right);
    padding: 0;
    min-height: calc(100vh - 60px);
    box-sizing: border-box;
  }
}
</style>
