<template>
    <div class="container">
    <div class="upload-home">
        <div class="top-toolbar">
            <div class="top-control-group">
                <div class="upload-folder-container">
                    <div class="upload-folder" :class="{ 'active': isFolderInputActive }">
                        <DirectorySuggestionInput
                            v-if="showDirectorySuggestions"
                            v-model="uploadFolder"
                            class="inner-folder-input"
                            :placeholder="$t('upload.folderPlaceholder')"
                            @focus="handleFolderInputFocus"
                            @blur="handleFolderInputBlur"
                            @select="handleDirectorySelect"
                        />
                        <el-input
                            v-else
                            class="inner-folder-input"
                            v-model="uploadFolder"
                            :placeholder="$t('upload.folderPlaceholder')"
                            @focus="handleFolderInputFocus"
                            @blur="handleFolderInputBlur"
                        />
                    </div>
                    <DirectoryTreePicker
                        v-if="showDirectorySuggestions"
                        :current-directory="uploadFolder"
                        source="upload"
                        @select="handleDirectorySelect"
                    >
                        <template #trigger>
                            <el-button class="top-icon-button directory-tree-trigger">
                                <font-awesome-icon icon="folder-tree" />
                            </el-button>
                        </template>
                    </DirectoryTreePicker>
                </div>
                <el-tooltip :disabled="disableTooltip" :content="$t('upload.history')" placement="bottom">
                    <el-button class="top-icon-button" @click="showHistory = true" circle>
                        <font-awesome-icon icon="history" />
                    </el-button>
                </el-tooltip>
                <el-tooltip :disabled="disableTooltip" :content="$t('upload.viewDocs')" placement="bottom">
                    <el-button class="top-icon-button" @click="viewDocs" circle>
                        <font-awesome-icon icon="book" />
                    </el-button>
                </el-tooltip>
                <LanguageSwitcher class="top-language-switcher"/>
                <ToggleDark class="top-theme-toggle"/>
            </div>
            <div class="top-control-group account-control-group">
                <el-tooltip :disabled="disableTooltip" :content="$t('dashboardTabs.systemSettings')" placement="bottom">
                    <el-button class="top-icon-button" @click="handleManage" circle>
                        <font-awesome-icon icon="cogs" />
                    </el-button>
                </el-tooltip>
                <el-tooltip :disabled="disableTooltip" :content="$t('upload.logout')" placement="bottom">
                    <el-button class="top-icon-button logout-action" @click="handleLogout" circle>
                        <font-awesome-icon icon="sign-out-alt" />
                    </el-button>
                </el-tooltip>
            </div>
        </div>
        <Logo :useConfigLink="true" />
        <div class="header">
            <h1 class="title"><a class="main-title" href="https://github.com/MarSeventh/CloudFlare-ImgBed" target="_blank">{{ ownerName }}</a> ImgHub</h1>
            <el-segmented
                class="upload-method-segmented"
                :model-value="uploadMethod"
                :options="uploadMethodOptions"
                @change="setUploadMethod"
            />
        </div>
        <UploadForm 
            :selectedUrlForm="selectedUrlForm" 
            :customerCompress="customerCompress" 
            :compressQuality="compressQuality"
            :compressBar="compressBar"
            :serverCompress="serverCompress"
            :uploadChannel="uploadChannel"
            :channelName="channelName"
            :uploadNameType="uploadNameType"
            :useCustomUrl="useCustomUrl"
            :customUrlPrefix="customUrlPrefix"
            :autoRetry="autoRetry"
            :urlPrefix="urlPrefix"
            :uploadMethod="uploadMethod"
            :uploadFolder="uploadFolder"
            :convertToWebp="convertToWebp"
            class="upload"
        />
    </div>
    <Footer class="footer"/>
    <el-dialog :title="$t('upload.announcementTitle')" v-model="showAnnouncementDialog" :width="dialogWidth" :show-close="false" :close-on-click-modal="false" :close-on-press-escape="false" center>
        <div v-html="announcementContent"></div>
        <template #footer>
            <span class="dialog-footer">
                <el-button type="primary" @click="showAnnouncementDialog = false">{{ $t('upload.announcementAck') }}</el-button>
            </span>
        </template>
    </el-dialog>
    <UploadHistory :show="showHistory" @close="showHistory = false" />
    </div>
</template>

<script>
import UploadForm from '@/components/upload/UploadForm.vue'
import Footer from '@/components/Footer.vue'
import ToggleDark from '@/components/ToggleDark.vue'
import Logo from '@/components/Logo.vue'
import UploadHistory from '@/components/upload/UploadHistory.vue'
import DirectoryTreePicker from '@/components/DirectoryTreePicker.vue'
import DirectorySuggestionInput from '@/components/DirectorySuggestionInput.vue'
import LanguageSwitcher from '@/components/LanguageSwitcher.vue'
import backgroundManager from '@/mixins/backgroundManager'
import axios from '@/utils/axios'
import { ref } from 'vue'
import { mapGetters } from 'vuex'
import { validateFolderPath } from '@/utils/pathValidator'

export default {
    name: 'UploadHome',
    mixins: [backgroundManager],
    data() {
        return {
            selectedUrlForm: ref(''),
            customerCompress: true, //上传前压缩
            compressQuality: 4, //压缩后大小
            compressBar: 5, //压缩阈值
            convertToWebp: false, //转换为WebP格式
            serverCompress: true, //服务器端压缩
            uploadChannel: '', //上传渠道
            channelName: '', //指定的渠道名称
            availableChannels: {}, //可用渠道列表
            uploadNameType: '', //上传文件命名方式
            customUrlPrefix: '', //自定义链接前缀
            useCustomUrl: 'false', //是否启用自定义链接格式
            autoRetry: true, //失败自动切换
            useDefaultWallPaper: false,
            uploadMethod: 'default', //上传方式
            uploadFolder: '', // 上传文件夹
            isFolderInputActive: false,
            showAnnouncementDialog: false, // 控制公告弹窗的显示
            announcementContent: '', // 公告内容
            showHistory: false
        }
    },
    watch: {
        customerCompress(val) {
            this.updateCompressConfig('customerCompress', val)
        },
        compressQuality(val) {
            this.updateCompressConfig('compressQuality', val)
        },
        compressBar(val) {
            // 确保值在有效范围内
            if (val === null || val === undefined || val < 1) {
                this.compressBar = 1
                return
            }
            // 确保期望大小不超过压缩阈值
            if (this.compressQuality > val) {
                this.compressQuality = val
            }
            this.updateCompressConfig('compressBar', val)
        },
        serverCompress(val) {
            this.updateCompressConfig('serverCompress', val)
        },
        convertToWebp(val) {
            this.updateCompressConfig('convertToWebp', val)
        },
        uploadChannel(val) {
            this.updateStoreUploadChannel(val)
            // 切换渠道类型时，检查持久化的渠道名是否在新渠道列表中
            const newChannelList = this.availableChannels[val] || []
            const savedChannelName = this.storeChannelName
            if (savedChannelName && newChannelList.some(ch => ch.name === savedChannelName)) {
                // 持久化的渠道名在新渠道列表中，恢复它
                this.channelName = savedChannelName
            } else {
                // 否则清空
                this.channelName = ''
            }
        },
        channelName(val) {
            // 确保清空时保存空字符串而不是null
            this.$store.commit('setStoreChannelName', val || '')
        },
        uploadNameType(val) {
            this.updateStoreUploadNameType(val)
        },
        customUrlPrefix(val) {
            this.$store.commit('setCustomUrlSettings', { key: 'customUrlPrefix', value: val })
        },
        useCustomUrl(val) {
            this.$store.commit('setCustomUrlSettings', { key: 'useCustomUrl', value: val })
        },
        autoRetry(val) {
            this.$store.commit('setStoreAutoRetry', val)
        },
        uploadFolder(val) {
            // 实时输入时用非 strict 模式，不检查末尾的单独 . 以允许继续输入如 .123
            if (this.validateUploadFolder(val, false)) {
                // 非 strict 通过后，再用 strict 模式静默检查，只有完全合法才更新 store
                const strictResult = validateFolderPath(val, { strict: true })
                if (strictResult.valid) {
                    this.$store.commit('setStoreUploadFolder', val)
                }
                // strict 不通过时不更新 store，等失焦时提示并回滚
            } else {
                this.$nextTick(() => {
                    this.uploadFolder = this.storeUploadFolder
                })
            }
        }
    },
    computed: {
        ...mapGetters(['userConfig', 'uploadCopyUrlForm', 'compressConfig', 'storeUploadChannel', 'storeChannelName', 'storeUploadNameType', 'customUrlSettings', 'storeAutoRetry', 'storeUploadMethod', 'storeUploadFolder']),
        ownerName() {
            return this.userConfig?.ownerName || 'Sanyue'
        },
        dialogWidth() {
            return window.innerWidth > 768 ? '50%' : '90%'
        },
        disableTooltip() {
            return window.innerWidth < 768
        },
        urlPrefix() {
            // 全局自定义链接前缀
            return this.userConfig?.urlPrefix || `${window.location.protocol}//${window.location.host}/file/`
        },
        // 是否显示目录候选项（从 userConfig 获取）
        showDirectorySuggestions() {
            return this.userConfig?.showDirectorySuggestions ?? false
        },
        // 当前渠道类型对应的渠道列表
        currentChannelList() {
            return this.availableChannels[this.uploadChannel] || []
        },
        uploadMethodOptions() {
            return [
                { label: this.$t('upload.fileUpload'), value: 'default' },
                { label: this.$t('upload.pasteUpload'), value: 'paste' }
            ]
        }
    },
    mounted() {
        // 初始化背景图，启用自动创建元素
        this.initializeBackground('uploadBkImg', '.container', false, true)

        // 读取用户选择的链接格式
        this.selectedUrlForm = this.uploadCopyUrlForm || 'url'
        // 读取用户选择的压缩设置（优先用户设置，其次系统默认配置）
        this.customerCompress = this.compressConfig.customerCompress ?? this.parseBoolean(this.userConfig?.defaultCustomerCompress, true)
        this.compressQuality = this.compressConfig.compressQuality ?? this.parseNumber(this.userConfig?.defaultCompressQuality, 4)
        this.compressBar = this.compressConfig.compressBar ?? this.parseNumber(this.userConfig?.defaultCompressBar, 5)
        this.serverCompress = this.compressConfig.serverCompress ?? true
        this.convertToWebp = this.compressConfig.convertToWebp ?? this.parseBoolean(this.userConfig?.defaultConvertToWebp, false)
        // 读取用户选择的上传渠道
        this.uploadChannel = this.storeUploadChannel || this.userConfig?.defaultUploadChannel || 'telegram'
        // 用户定义的失败自动切换
        this.autoRetry = this.storeAutoRetry
        // 读取用户选择的上传文件命名方式
        this.uploadNameType = this.storeUploadNameType || this.userConfig?.defaultUploadNameType || 'default'
        // 读取用户自定义链接格式
        this.customUrlPrefix = this.customUrlSettings.customUrlPrefix
        this.useCustomUrl = this.customUrlSettings.useCustomUrl
        // 读取用户偏好的上传方式
        this.uploadMethod = this.storeUploadMethod
        // 获取可用渠道列表
        this.fetchAvailableChannels()
        // 读取用户设置的上传文件夹
        this.uploadFolder = this.storeUploadFolder || this.userConfig?.defaultUploadFolder || ''

        // 首次访问公告
        const visited = localStorage.getItem('visitedUploadHome')
        const announcement = this.userConfig?.announcement
        if (!visited && announcement) {
            this.announcementContent = announcement
            this.showAnnouncementDialog = true
            localStorage.setItem('visitedUploadHome', 'true')
        }
    },
    components: {
        UploadForm,
        Footer,
        ToggleDark,
        Logo,
        UploadHistory,
        DirectoryTreePicker,
        DirectorySuggestionInput,
        LanguageSwitcher
    },
    methods: {
        // 获取可用渠道列表
        async fetchAvailableChannels() {
            try {
                const response = await axios.get('/api/channels', { withAuthCode: true })
                if (response.data) {
                    this.availableChannels = response.data
                    // 恢复渠道名称：优先持久化的值，其次系统默认配置
                    const savedChannelName = this.storeChannelName
                    const defaultChannelName = this.userConfig?.defaultChannelName
                    const currentChannelList = this.availableChannels[this.uploadChannel] || []
                    
                    // 如果用户主动清空过（savedChannelName === ''），则保持为空
                    // 如果从未选择过（savedChannelName === null/undefined），则使用默认值
                    if (savedChannelName && currentChannelList.some(ch => ch.name === savedChannelName)) {
                        this.channelName = savedChannelName
                    } else if (savedChannelName === '' || savedChannelName === null || savedChannelName === undefined) {
                        // 用户主动清空或从未选择，检查是否使用默认值
                        if (savedChannelName !== '' && defaultChannelName && currentChannelList.some(ch => ch.name === defaultChannelName)) {
                            this.channelName = defaultChannelName
                        }
                        // 如果 savedChannelName === ''，说明用户主动清空，保持为空
                    }
                }
            } catch (error) {
                console.error('Failed to fetch available channels:', error)
            }
        },
        // 验证上传文件夹路径的合法性
        validateUploadFolder(path, strict = true) {
            // 自动补全前导 /
            if (path && !path.startsWith('/')) {
                path = '/' + path
                this.uploadFolder = path
            }
            const result = validateFolderPath(path, { strict })
            if (!result.valid) {
                this.$message.error(result.error)
                return false
            }
            return true
        },
        handleFolderInputFocus() {
            this.isFolderInputActive = true
        },
        handleFolderInputBlur() {
            this.isFolderInputActive = false
            // 失焦时自动补全前导 /
            if (this.uploadFolder && !this.uploadFolder.startsWith('/')) {
                this.uploadFolder = '/' + this.uploadFolder
            }
            // 失焦时做完整校验（包括末尾单独的 .）
            if (!this.validateUploadFolder(this.uploadFolder, true)) {
                this.$nextTick(() => {
                    this.uploadFolder = this.storeUploadFolder
                })
            }
        },
        handleManage() {
            this.$router.push('/systemConfig#uploadSettings')
        },
        // 解析布尔值
        parseBoolean(value, defaultValue) {
            if (value === undefined || value === null) return defaultValue
            if (typeof value === 'boolean') return value
            if (typeof value === 'string') return value === 'true'
            return defaultValue
        },
        // 解析数字
        parseNumber(value, defaultValue) {
            if (value === undefined || value === null) return defaultValue
            const num = parseFloat(value)
            return isNaN(num) ? defaultValue : num
        },
        handleLogout() {
            axios.post('/api/auth/logout', { authType: 'user' }, { withCredentials: true }).finally(() => {
                this.$store.commit('setUserLoggedIn', false);
                this.$router.push('/login')
                this.$message.success(this.$t('upload.logoutSuccess'))
            })
        },
        updateCompressConfig(key, value) {
            this.$store.commit('setCompressConfig', { key, value })
        },
        updateStoreUploadChannel(value) {
            this.$store.commit('setStoreUploadChannel', value)
        },
        updateStoreUploadNameType(value) {
            this.$store.commit('setStoreUploadNameType', value)
        },
        setUploadMethod(method) {
            if (method === this.uploadMethod) return
            this.uploadMethod = method
            this.$store.commit('setUploadMethod', method)
        },
        viewDocs() {
            window.open('https://cfbed.sanyue.de/qa/', '_blank')
        },
        // 处理目录选择
        handleDirectorySelect(path) {
            // 填入选择的目录路径
            this.uploadFolder = path
            // 触发路径验证逻辑
            if (this.validateUploadFolder(path, true)) {
                this.$store.commit('setStoreUploadFolder', path)
            }
        }
    }
}
</script>

<style scoped>
.container {
    background: var(--bg-color);
    min-height: 100vh;
}

/* 定义顺时针和逆时针旋转动画 */
.rotate {
    animation: spin 2s ease-in-out; /* 动画时长为2秒，执行一次 */
}

/* 定义放大缩小动画 */
.scale {
    animation: scale 0.5s ease-in-out; /* 动画时长为0.5秒，执行一次 */
}

/* 关键帧：先顺时针旋转，再逆时针旋转 */
@keyframes spin {
    0% {
        transform: rotate(0deg); /* 初始位置 */
    }
    25% {
        transform: rotate(5deg); /* 顺时针旋转20度 */
    }
    50% {
        transform: rotate(0deg); /* 顺时针旋转回到初始位置 */
    }
    75% {
        transform: rotate(-3deg); /* 逆时针旋转20度 */
    }
    100% {
        transform: rotate(0deg); /* 逆时针旋转回到初始位置 */
    }
}

@keyframes streamer {
    0% {
        background-position: 200% center;
    }
    100% {
        background-position: -200% center;
    }
}


/* 关键帧：旋转抖动 */
@keyframes rotate-shake {
    0% {
        transform: rotate(0deg); /* 初始位置 */
    }
    50% {
        transform: rotate(10deg); /* 旋转10度 */
    }
    100% {
        transform: rotate(0deg); /* 回到初始位置 */
    }
}

/* 关键帧：左右抖动 */
@keyframes shake {
    0% {
        transform: translateX(0); /* 初始位置 */
    }
    50% {
        transform: translateX(-1px); /* 向右移动3像素 */
    }
    100% {
        transform: translateX(0); /* 回到初始位置 */
    }
}

/* 关键帧：放大缩小 */
@keyframes scale {
    0% {
        transform: scale(1); /* 初始大小 */
    }
    50% {
        transform: scale(1.1); /* 放大到1.2倍 */
    }
    100% {
        transform: scale(1); /* 回到初始大小 */
    }
}


.top-toolbar {
    position: fixed;
    top: 30px;
    right: 30px;
    z-index: 120;
    display: flex;
    align-items: center;
    gap: 20px;
}

.top-control-group {
    display: flex;
    align-items: center;
    gap: 8px;
}

.top-control-group :deep(.el-button) {
    margin-left: 0;
}

.top-icon-button {
    width: 2.5rem;
    height: 2.5rem;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border: 1px solid rgba(255, 255, 255, 0.18);
    transition: all 0.3s ease;
    background-color: var(--toolbar-button-bg-color);
    box-shadow: none;
    backdrop-filter: blur(10px);
    color: var(--theme-toggle-color);
    border-radius: 12px;
    outline: none;
    padding: 0;
}

.top-language-switcher {
    --lang-icon-size: 1.18em;
    --lang-icon-color: var(--theme-toggle-color);
    width: 2.5rem;
    height: 2.5rem;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    border-radius: 12px;
    background-color: var(--toolbar-button-bg-color);
    border: 1px solid rgba(255, 255, 255, 0.18);
    box-shadow: none;
    backdrop-filter: blur(10px);
    box-sizing: border-box;
}

.top-theme-toggle {
    width: 2.5rem !important;
    height: 2.5rem !important;
    background-color: var(--toolbar-button-bg-color);
    border: 1px solid rgba(255, 255, 255, 0.18) !important;
    box-shadow: none;
    backdrop-filter: blur(10px);
    border-radius: 12px;
    box-sizing: border-box;
}

.top-icon-button.logout-action {
    color: #f56c6c;
}

.top-icon-button.is-disabled {
    opacity: 0.35;
}

.top-language-switcher :deep(.lang-current) {
    min-width: 0;
    min-height: 0;
    border: none;
    background: transparent;
}

/* 上传文件输入框容器样式 */
.upload-folder-container {
    display: flex;
    align-items: center;
    gap: 8px;
}
@media (max-width: 768px) {
    .top-toolbar {
        top: 24px;
        left: 12px;
        right: 12px;
        justify-content: flex-end;
        gap: 14px;
    }

    .top-control-group {
        gap: 4px;
    }

    .top-icon-button,
    .top-language-switcher {
        width: 1.85rem;
        height: 1.85rem;
        border-radius: 10px;
    }

    .top-language-switcher {
        --lang-icon-size: 1em;
    }

    .top-theme-toggle {
        width: 1.85rem !important;
        height: 1.85rem !important;
    }
}

/* 上传文件输入框样式 */
.upload-folder {
    width: 100px;
    height: 2.5rem;
    border-radius: 12px;
    transition: all 0.3s ease, width 0.4s ease;
}
.upload-folder.active {
    width: 200px;
}
@media (max-width: 768px) {
    .upload-folder {
        width: 76px;
        height: 1.85rem;
    }
    .upload-folder.active {
        width: 96px;
    }
}

/* 目录树触发按钮 */
.directory-tree-trigger {
    width: 2.5rem;
    height: 2.5rem;
    display: flex;
    justify-content: center;
    align-items: center;
    border: none;
    margin-left: 0;
    transition: all 0.3s ease;
    background-color: var(--toolbar-button-bg-color);
    box-shadow: none;
    backdrop-filter: blur(10px);
    color: var(--theme-toggle-color);
    border-radius: 12px;
    outline: none;
}
.directory-tree-trigger:hover {
    transform: scale(1.05);
    box-shadow: none;
}
@media (max-width: 768px) {
    .directory-tree-trigger {
        width: 1.85rem;
        height: 1.85rem;
    }
}

.upload-folder :deep(.inner-folder-input) {
    width: 100%;
    height: 100%;
}

.upload-folder :deep(.el-input) {
    height: 100%;
}

.upload-folder :deep(.el-input__wrapper) {
    border-radius: 12px;
    background-color: var(--toolbar-button-bg-color);
    box-shadow: none;
    backdrop-filter: blur(10px);
    border: 1px solid rgba(64, 158, 255, 0.36);
    height: 100%;
}

.upload-folder.active :deep(.el-input__wrapper),
.upload-folder:hover :deep(.el-input__wrapper) {
    border-color: var(--el-color-primary);
}

/* 按钮悬停效果 */
.info-container:hover,
.top-icon-button:hover,
.top-language-switcher:hover,
.top-theme-toggle:hover {
    transform: scale(1.05);
    box-shadow: none;
}


:deep(.el-dialog) {
    border-radius: 12px;
    background-color: var(--dialog-bg-color);
    backdrop-filter: blur(10px);
    box-shadow: var(--dialog-box-shadow);
}
.dialog-action {
    display: flex;
    justify-content: center;
    margin-top: 20px;
}


.header {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 15px;
    margin-top: 5vh;
    color: var(--upload-header-color);
    user-select: none;
    text-decoration: none;
    position: relative;
    top: -3vh;
    transition: all 0.3s ease;
}

.upload-method-segmented {
    --el-segmented-bg-color: rgba(64, 158, 255, 0.08);
    --el-segmented-item-selected-bg-color: var(--el-color-primary);
    --el-segmented-item-selected-color: #fff;
    margin-top: 6px;
    min-height: 38px;
    padding: 4px;
    border-radius: 12px;
    background: rgba(64, 158, 255, 0.08);
    /* border: 1px solid rgba(255, 255, 255, 0.18); */
    box-sizing: border-box;
}

.upload-method-segmented :deep(.el-segmented__item) {
    min-width: 86px;
    min-height: 30px;
    font-size: 14px;
    font-weight: 600;
}
.title {
    font-size: 2.5rem;
    font-weight: 400;
    font-family: 'Righteous', 'Noto Sans SC', sans-serif;
    position: relative;
    padding-bottom: 8px;
    cursor: pointer;
    transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
    animation: float 4s ease-in-out infinite;
    letter-spacing: 3px;
}
.title:hover {
    transform: scale(1.08) translateY(-3px);
    filter: drop-shadow(0 0 20px var(--el-upload-dragger-uniform-color));
}
.title::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 0;
    height: 3px;
    background: linear-gradient(90deg, 
        transparent, 
        var(--el-upload-dragger-uniform-color), 
        transparent);
    border-radius: 3px;
    transition: width 0.5s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 0 10px var(--el-upload-dragger-uniform-color);
}
.title:hover::after {
    width: 80%;
}

/* 动态流光标题 */
.main-title {
    background: var(--upload-main-title-color);
    background-size: 200% auto;
    background-clip: text;
    -webkit-background-clip: text;
    color: transparent;
    text-decoration: none;
    display: inline-block;
    animation: titleShimmer 3s ease-in-out infinite;
    position: relative;
    filter: drop-shadow(0 0 8px rgba(255, 255, 255, 0.3));
}



.title:hover .main-title {
    animation: titleShimmer 1s ease-in-out infinite;
    filter: brightness(1.2);
}

/* 漂浮动画 */
@keyframes float {
    0%, 100% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(-5px);
    }
}

/* 标题流光动画 */
@keyframes titleShimmer {
    0% {
        background-position: 200% center;
    }
    100% {
        background-position: -200% center;
    }
}

@media (max-width: 768px) {
    .title {
        font-size: 1.8rem;
        letter-spacing: 1px;
    }
    .title:hover {
        transform: scale(1.05) translateY(-2px);
    }
}

.upload-home {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-height: 94vh;
    background-color: var(--admin-container-bg-color);
}
.upload {
    margin-bottom: 5px;
    position: relative;
    top: -3vh;
}

.footer {
    height: 6vh;
}
</style>
