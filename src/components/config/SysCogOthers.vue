<template>
    <div class="others-settings" v-loading="loading">
        <div class="page-header">
            <h3 class="first-title">{{ $t('sysConfigTabs.otherSettings') }}</h3>
        </div>

        <!-- 一级设置：其他设置 -->
        <div class="settings-stack">
            <section class="settings-section">
                <div class="section-header">
                    <span class="section-title">{{ $t('sysOthers.remoteTelemetry') }}</span>
                    <el-tooltip :content="$t('sysOthers.remoteTelemetryTooltip')" placement="right">
                        <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                    </el-tooltip>
                </div>
                <el-form :model="settings.telemetry" label-width="120px">
                    <el-form-item :label="$t('sysOthers.enable')">
                        <el-switch v-model="settings.telemetry.enabled" :disabled="settings.telemetry.fixed"></el-switch>
                    </el-form-item>
                </el-form>
            </section>

            <section class="settings-section">
                <div class="section-header">
                    <span class="section-title">{{ $t('sysOthers.cloudflareApiToken') }}</span>
                    <el-tooltip :content="$t('sysOthers.cloudflareApiTokenTooltip')" placement="right" raw-content>
                        <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                    </el-tooltip>
                </div>
                <el-form :model="settings.cloudflareApiToken" label-width="120px">
                    <el-form-item :label="$t('sysOthers.zoneId')">
                        <el-input v-model="settings.cloudflareApiToken.CF_ZONE_ID" :disabled="settings.cloudflareApiToken.fixed"></el-input>
                    </el-form-item>
                    <el-form-item :label="$t('sysOthers.accountEmail')">
                        <el-input v-model="settings.cloudflareApiToken.CF_EMAIL" :disabled="settings.cloudflareApiToken.fixed"></el-input>
                    </el-form-item>
                    <el-form-item label="API Key">
                        <el-input v-model="settings.cloudflareApiToken.CF_API_KEY" :disabled="settings.cloudflareApiToken.fixed" type="password" show-password autocomplete="new-password"></el-input>
                    </el-form-item>
                </el-form>
            </section>

            <section class="settings-section">
                <div class="section-header">
                    <span class="section-title">{{ $t('sysOthers.randomImageApi') }}</span>
                    <el-tooltip :content="$t('sysOthers.randomImageApiTooltip')" placement="right">
                        <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                    </el-tooltip>
                </div>
                <el-form :model="settings.randomImageAPI" label-width="120px">
                    <el-form-item :label="$t('sysOthers.enable')">
                        <el-switch v-model="settings.randomImageAPI.enabled" :disabled="settings.randomImageAPI.fixed"></el-switch>
                    </el-form-item>
                    <el-form-item prop="randomImageAPI.allowedDir">
                        <template #label>
                            <span>{{ $t('sysOthers.directory') }}</span>
                            <el-tooltip :content="$t('sysOthers.directoryTooltip')" placement="right" raw-content>
                                <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                            </el-tooltip>
                        </template>
                        <el-input v-model="settings.randomImageAPI.allowedDir" :disabled="settings.randomImageAPI.fixed"></el-input>
                    </el-form-item>
                </el-form>
            </section>

            <section class="settings-section">
                <div class="section-header">
                    <span class="section-title">{{ $t('sysOthers.publicBrowse') }}</span>
                    <el-tooltip :content="$t('sysOthers.publicBrowseTooltip')" placement="right" raw-content>
                        <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                    </el-tooltip>
                </div>
                <el-form :model="settings.publicBrowse" label-width="120px">
                    <el-form-item :label="$t('sysOthers.enable')">
                        <el-switch v-model="settings.publicBrowse.enabled" :disabled="settings.publicBrowse.fixed"></el-switch>
                    </el-form-item>
                    <el-form-item prop="publicBrowse.allowedDir">
                        <template #label>
                            <span>{{ $t('sysOthers.openDirectory') }}</span>
                            <el-tooltip placement="right" raw-content>
                                <template #content>
                                    <div style="max-width: 320px; line-height: 1.6;" v-html="$t('sysOthers.openDirectoryTooltip')">
                                    </div>
                                </template>
                                <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                            </el-tooltip>
                        </template>
                        <el-input v-model="settings.publicBrowse.allowedDir" :disabled="settings.publicBrowse.fixed" placeholder="wallpaper,photos,album"></el-input>
                    </el-form-item>
                </el-form>
            </section>

            <section class="settings-section section-wide">
                <div class="section-header">
                    <span class="section-title">{{ $t('sysOthers.webdav') }}</span>
                    <el-tooltip :content="$t('sysOthers.webdavTooltip')" placement="right" raw-content>
                        <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                    </el-tooltip>
                </div>
                <el-form :model="settings.webDAV" label-width="120px">
                    <el-form-item :label="$t('sysOthers.enable')">
                        <el-switch v-model="settings.webDAV.enabled" :disabled="settings.webDAV.fixed"></el-switch>
                    </el-form-item>
                    <el-form-item :label="$t('sysOthers.webdavUsername')">
                        <el-input v-model="settings.webDAV.username" :disabled="settings.webDAV.fixed"></el-input>
                    </el-form-item>
                    <el-form-item :label="$t('sysOthers.webdavPassword')">
                        <el-input v-model="settings.webDAV.password" :disabled="settings.webDAV.fixed" type="password" show-password autocomplete="new-password"></el-input>
                    </el-form-item>
                    <el-form-item :label="$t('sysOthers.webdavUploadChannel')">
                        <el-select v-model="settings.webDAV.uploadChannel" :disabled="settings.webDAV.fixed" :placeholder="$t('sysOthers.webdavDefaultChannel')" clearable>
                            <el-option label="Telegram" value="telegram"></el-option>
                            <el-option label="Cloudflare R2" value="cfr2"></el-option>
                            <el-option label="S3" value="s3"></el-option>
                            <el-option label="Discord" value="discord"></el-option>
                            <el-option label="HuggingFace" value="huggingface"></el-option>
                            <el-option label="WebDAV" value="webdav"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item :label="$t('sysOthers.webdavChannelName')" v-if="settings.webDAV.uploadChannel && webdavChannelList.length > 1">
                        <el-select v-model="settings.webDAV.channelName" :disabled="settings.webDAV.fixed" :placeholder="$t('uploadSettings.autoSelect')" clearable>
                            <el-option
                                v-for="ch in webdavChannelList"
                                :key="ch.name"
                                :label="ch.name"
                                :value="ch.name"
                            ></el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </section>
        </div>

    
        <!-- 悬浮保存按钮 -->
        <FloatingSaveButton :show="!loading" @click="saveSettings" />
    </div>
</template>

<script>
import fetchWithAuth from '@/utils/fetchWithAuth';
import FloatingSaveButton from '@/components/FloatingSaveButton.vue';

export default {
components: {
    FloatingSaveButton
},
data() {
    return {
        settings: {
            telemetry: {},
            randomImageAPI: {},
            cloudflareApiToken: {},
            webDAV: {},
            publicBrowse: {}
        },
        availableChannels: {}, // 可用渠道列表
        // 加载状态
        loading: true
    };
},
computed: {
    // WebDAV 当前渠道类型对应的渠道列表
    webdavChannelList() {
        const channelType = this.settings.webDAV?.uploadChannel;
        return channelType ? (this.availableChannels[channelType] || []) : [];
    }
},
watch: {
    'settings.webDAV.uploadChannel'() {
        // 切换渠道类型时清空指定的渠道名称
        if (this.settings.webDAV) {
            this.settings.webDAV.channelName = '';
        }
    }
},
methods: {
    saveSettings() {
        fetchWithAuth('/api/manage/sysConfig/others', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(this.settings)
        })
        .then(() => this.$message.success(this.$t('sysOthers.settingsSaved')));
    },
    async fetchAvailableChannels() {
        try {
            const response = await fetchWithAuth('/api/channels');
            if (response.ok) {
                this.availableChannels = await response.json();
            }
        } catch (error) {
            console.error('Failed to fetch available channels:', error);
        }
    }
},
mounted() {
    this.loading = true;
    // 获取上传设置
    fetchWithAuth('/api/manage/sysConfig/others')
    .then((response) => response.json())
    .then((data) => {
        this.settings = data;
    })
    .finally(() => {
        this.loading = false;
    });
    // 获取可用渠道列表
    this.fetchAvailableChannels();
}
};
</script>

<style scoped>
.others-settings {
    padding: 18px 0 28px;
    min-height: 500px;
}

.page-header {
    margin-bottom: 14px;
}

.first-title {
    display: flex;
    align-items: center;
    gap: 8px;
    margin: 0;
    padding-bottom: 10px;
    border-bottom: 1px solid var(--flat-border);
    font-size: 20px;
    font-weight: 600;
}

.settings-stack {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 14px;
}

.settings-section {
    background: var(--flat-surface);
    border: 1px solid var(--flat-border);
    border-radius: 8px;
    padding: 16px;
}

.settings-section:hover {
    border-color: var(--flat-border-strong);
}

.section-wide {
    grid-column: 1 / -1;
}

.settings-section .section-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 10px;
    margin-bottom: 14px;
    padding-bottom: 10px;
    border-bottom: 1px solid var(--flat-border);
}

.section-title {
    font-size: 15px;
    font-weight: 600;
    color: var(--flat-text);
}

.section-help-icon {
    color: var(--flat-text-muted);
    cursor: pointer;
    font-size: 13px;
}

.settings-section :deep(.el-form) {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 12px;
}

.settings-section :deep(.el-form-item) {
    margin-bottom: 0;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    min-width: 0;
    padding: 12px 14px;
    background: var(--flat-surface-soft);
    border: 1px solid var(--flat-border);
    border-radius: 8px;
}

.settings-section :deep(.el-form-item:only-child) {
    grid-column: 1 / -1;
}

.settings-section :deep(.el-form-item__label) {
    text-align: left;
    padding-bottom: 6px;
    font-weight: 500;
    color: var(--el-text-color-primary);
    width: auto !important;
    display: flex;
    align-items: center;
    gap: 5px;
}

.settings-section :deep(.el-form-item__content) {
    width: 100%;
    max-width: 100%;
    margin-left: 0 !important;
}

.settings-section :deep(.el-input) {
    width: 100%;
}

.settings-section :deep(.el-select) {
    width: 100%;
}

.settings-section :deep(.el-switch) {
    --el-switch-on-color: var(--el-color-primary);
}

@media (max-width: 1200px) {
    .settings-stack,
    .settings-section :deep(.el-form) {
        grid-template-columns: 1fr;
    }
}

/* 移动端适配 */
@media (max-width: 768px) {
    .others-settings {
        padding: 12px;
        padding-bottom: 80px;
    }
}
</style>
