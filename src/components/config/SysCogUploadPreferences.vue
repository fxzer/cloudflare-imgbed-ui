<template>
    <div class="upload-preferences" v-loading="loading">
        <div class="page-header">
            <h3 class="first-title">{{ $t('uploadSettings.title') }}</h3>
        </div>

        <div class="settings-grid">
            <section class="preference-panel panel-wide workflow-panel" data-step="01">
                <div class="section-header">
                    <span class="section-title">{{ $t('uploadSettings.uploadChannel') }}</span>
                </div>
                <div class="section-content target-grid">
                    <div class="setting-item setting-item-stack setting-item-full">
                        <span class="setting-label">{{ $t('uploadSettings.channelType') }}</span>
                        <el-radio-group v-model="uploadChannel" class="radio-card-group compact">
                            <el-radio label="telegram" class="radio-card">
                                <font-awesome-icon icon="paper-plane" class="channel-icon"/>
                                <span>TG</span>
                            </el-radio>
                            <el-radio label="cfr2" class="radio-card">
                                <font-awesome-icon icon="cloud" class="channel-icon"/>
                                <span>R2</span>
                            </el-radio>
                            <el-radio label="s3" class="radio-card">
                                <font-awesome-icon icon="database" class="channel-icon"/>
                                <span>S3</span>
                            </el-radio>
                            <el-radio label="discord" class="radio-card">
                                <font-awesome-icon icon="comments" class="channel-icon"/>
                                <span>DC</span>
                            </el-radio>
                            <el-radio label="huggingface" class="radio-card">
                                <font-awesome-icon icon="robot" class="channel-icon"/>
                                <span>HF</span>
                            </el-radio>
                            <el-radio label="webdav" class="radio-card">
                                <font-awesome-icon icon="folder" class="channel-icon"/>
                                <span>WD</span>
                            </el-radio>
                        </el-radio-group>
                    </div>
                    <div class="setting-item" v-if="currentChannelList.length > 1">
                        <span class="setting-label">
                            {{ $t('uploadSettings.channelName') }}
                            <el-tooltip :content="$t('uploadSettings.channelNameTooltip')" placement="top">
                                <font-awesome-icon icon="question-circle" class="inline-help-icon"/>
                            </el-tooltip>
                        </span>
                        <el-select v-model="channelName" :placeholder="$t('uploadSettings.autoSelect')" clearable class="setting-input">
                            <el-option
                                v-for="ch in currentChannelList"
                                :key="ch.name"
                                :label="ch.name"
                                :value="ch.name"
                            />
                        </el-select>
                    </div>
                    <div class="setting-item">
                        <span class="setting-label">{{ $t('uploadSettings.uploadDirectory') }}</span>
                        <el-input
                            v-model="uploadFolder"
                            :placeholder="$t('uploadSettings.uploadDirectoryPlaceholder')"
                            class="setting-input"
                            @blur="handleUploadFolderBlur"
                        />
                    </div>
                    <div class="setting-item">
                        <span class="setting-label">
                            {{ $t('uploadSettings.autoSwitch') }}
                            <el-tooltip :content="$t('uploadSettings.autoSwitchTooltip')" placement="top">
                                <font-awesome-icon icon="question-circle" class="inline-help-icon"/>
                            </el-tooltip>
                        </span>
                        <el-switch v-model="autoRetry" />
                    </div>
                </div>
            </section>

            <section class="preference-panel panel-wide workflow-panel" data-step="02">
                <div class="section-header">
                    <span class="section-title">{{ $t('uploadSettings.filePreprocess') }}</span>
                    <el-tooltip :content="$t('uploadSettings.filePreprocessTooltip')" placement="top">
                        <font-awesome-icon icon="question-circle" class="section-help-icon"/>
                    </el-tooltip>
                </div>
                <div class="section-content preprocess-grid">
                    <div class="setting-item">
                        <span class="setting-label">
                            {{ $t('uploadSettings.convertToWebp') }}
                            <el-tooltip :content="$t('uploadSettings.convertToWebpTooltip')" placement="top">
                                <font-awesome-icon icon="question-circle" class="inline-help-icon"/>
                            </el-tooltip>
                        </span>
                        <el-switch v-model="convertToWebp" />
                    </div>
                    <div class="setting-item">
                        <span class="setting-label">{{ $t('uploadSettings.fileCompress') }}</span>
                        <el-switch v-model="customerCompress" />
                    </div>
                    <div class="setting-item slider-item" v-if="customerCompress">
                        <span class="setting-label">
                            {{ $t('uploadSettings.compressThreshold') }}
                            <el-tooltip :content="$t('uploadSettings.compressThresholdTooltip')" placement="top">
                                <font-awesome-icon icon="question-circle" class="inline-help-icon"/>
                            </el-tooltip>
                        </span>
                        <div class="slider-wrapper">
                            <el-slider v-model="compressBar" :min="1" :max="20" :format-tooltip="value => `${value} MB`"/>
                            <div class="slider-input-wrapper">
                                <el-input-number v-model="compressBar" :min="1" :max="20" :step="1" :value-on-clear="1" class="slider-input" controls-position="right"/>
                                <span class="slider-unit">MB</span>
                            </div>
                        </div>
                    </div>
                    <div class="setting-item slider-item" v-if="customerCompress">
                        <span class="setting-label">
                            {{ $t('uploadSettings.expectedSize') }}
                            <el-tooltip :content="$t('uploadSettings.expectedSizeTooltip')" placement="top">
                                <font-awesome-icon icon="question-circle" class="inline-help-icon"/>
                            </el-tooltip>
                        </span>
                        <div class="slider-wrapper">
                            <el-slider v-model="compressQuality" :min="0.5" :max="compressBar" :step="0.1" :format-tooltip="value => `${value} MB`"/>
                            <div class="slider-input-wrapper">
                                <el-input-number v-model="compressQuality" :min="0.5" :max="compressBar" :step="0.1" :precision="1" :value-on-clear="0.5" class="slider-input" controls-position="right"/>
                                <span class="slider-unit">MB</span>
                            </div>
                        </div>
                    </div>
                    <div class="setting-item" v-if="uploadChannel === 'telegram'">
                        <span class="setting-label">
                            {{ $t('uploadSettings.serverCompress') }}
                            <el-tooltip :content="$t('uploadSettings.serverCompressTooltip')" placement="top" raw-content>
                                <font-awesome-icon icon="question-circle" class="inline-help-icon"/>
                            </el-tooltip>
                        </span>
                        <el-switch v-model="serverCompress" />
                    </div>
                </div>
            </section>

            <section class="preference-panel workflow-panel" data-step="03">
                <div class="section-header">
                    <span class="section-title">{{ $t('uploadSettings.fileNaming') }}</span>
                </div>
                <div class="section-content">
                    <el-radio-group v-model="uploadNameType" class="radio-card-group grid-2x2">
                        <el-radio label="default" class="radio-card">
                            <font-awesome-icon icon="cog" class="radio-icon"/>
                            <span>{{ $t('uploadSettings.namingDefault') }}</span>
                        </el-radio>
                        <el-radio label="index" class="radio-card">
                            <font-awesome-icon icon="hashtag" class="radio-icon"/>
                            <span>{{ $t('uploadSettings.namingIndex') }}</span>
                        </el-radio>
                        <el-radio label="origin" class="radio-card">
                            <font-awesome-icon icon="file-signature" class="radio-icon"/>
                            <span>{{ $t('uploadSettings.namingOrigin') }}</span>
                        </el-radio>
                        <el-radio label="short" class="radio-card">
                            <font-awesome-icon icon="compress-alt" class="radio-icon"/>
                            <span>{{ $t('uploadSettings.namingShort') }}</span>
                        </el-radio>
                    </el-radio-group>
                </div>
            </section>

            <section class="preference-panel workflow-panel" data-step="04">
                <div class="section-header">
                    <span class="section-title">{{ $t('settings.linkFormatTitle') }}</span>
                </div>
                <div class="section-content">
                    <el-radio-group v-model="selectedUrlForm" class="radio-card-group grid-2x2">
                        <el-radio label="url" class="radio-card">
                            <font-awesome-icon icon="link" class="radio-icon"/>
                            <span>{{ $t('settings.rawLink') }}</span>
                        </el-radio>
                        <el-radio label="md" class="radio-card">
                            <font-awesome-icon icon="code" class="radio-icon"/>
                            <span>Markdown</span>
                        </el-radio>
                        <el-radio label="html" class="radio-card">
                            <font-awesome-icon icon="code-branch" class="radio-icon"/>
                            <span>HTML</span>
                        </el-radio>
                        <el-radio label="ubb" class="radio-card">
                            <font-awesome-icon icon="quote-right" class="radio-icon"/>
                            <span>BBCode</span>
                        </el-radio>
                    </el-radio-group>
                    <div class="setting-item link-custom-switch">
                        <span class="setting-label">{{ $t('settings.enableCustom') }}</span>
                        <el-switch v-model="useCustomUrl" active-value="true" inactive-value="false" />
                    </div>
                    <div class="setting-item" v-if="useCustomUrl === 'true'">
                        <span class="setting-label">{{ $t('settings.customPrefix') }}</span>
                        <el-input v-model="customUrlPrefix" :placeholder="$t('settings.customPrefixPlaceholder')" class="setting-input"/>
                    </div>
                </div>
            </section>

        </div>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';
import axios from '@/utils/axios';
import { validateFolderPath } from '@/utils/pathValidator';

export default {
    name: 'SysCogUploadPreferences',
    data() {
        return {
            loading: true,
            hydrating: true,
            selectedUrlForm: 'url',
            customerCompress: true,
            compressQuality: 4,
            compressBar: 5,
            convertToWebp: false,
            serverCompress: true,
            uploadChannel: 'telegram',
            channelName: '',
            availableChannels: {},
            uploadNameType: 'default',
            customUrlPrefix: '',
            useCustomUrl: 'false',
            autoRetry: true,
            uploadFolder: ''
        };
    },
    computed: {
        ...mapGetters([
            'userConfig',
            'uploadCopyUrlForm',
            'compressConfig',
            'storeUploadChannel',
            'storeChannelName',
            'storeUploadNameType',
            'customUrlSettings',
            'storeAutoRetry',
            'storeUploadFolder'
        ]),
        currentChannelList() {
            return this.availableChannels[this.uploadChannel] || [];
        }
    },
    watch: {
        selectedUrlForm(val) {
            if (!this.hydrating) this.$store.commit('setUploadCopyUrlForm', val);
        },
        customerCompress(val) {
            if (!this.hydrating) this.updateCompressConfig('customerCompress', val);
        },
        compressQuality(val) {
            if (!this.hydrating) this.updateCompressConfig('compressQuality', val);
        },
        compressBar(val) {
            if (this.hydrating) return;
            if (val === null || val === undefined || val < 1) {
                this.compressBar = 1;
                return;
            }
            if (this.compressQuality > val) {
                this.compressQuality = val;
            }
            this.updateCompressConfig('compressBar', val);
        },
        serverCompress(val) {
            if (!this.hydrating) this.updateCompressConfig('serverCompress', val);
        },
        convertToWebp(val) {
            if (!this.hydrating) this.updateCompressConfig('convertToWebp', val);
        },
        uploadChannel(val) {
            if (this.hydrating) return;
            this.$store.commit('setStoreUploadChannel', val);
            this.restoreChannelName();
        },
        channelName(val) {
            if (!this.hydrating) this.$store.commit('setStoreChannelName', val || '');
        },
        uploadNameType(val) {
            if (!this.hydrating) this.$store.commit('setStoreUploadNameType', val);
        },
        customUrlPrefix(val) {
            if (!this.hydrating) this.$store.commit('setCustomUrlSettings', { key: 'customUrlPrefix', value: val });
        },
        useCustomUrl(val) {
            if (!this.hydrating) this.$store.commit('setCustomUrlSettings', { key: 'useCustomUrl', value: val });
        },
        autoRetry(val) {
            if (!this.hydrating) this.$store.commit('setStoreAutoRetry', val);
        },
        uploadFolder(val) {
            if (this.hydrating) return;
            if (this.validateUploadFolder(val, false)) {
                const strictResult = validateFolderPath(val, { strict: true });
                if (strictResult.valid) {
                    this.$store.commit('setStoreUploadFolder', val);
                }
            } else {
                this.$nextTick(() => {
                    this.uploadFolder = this.storeUploadFolder;
                });
            }
        }
    },
    methods: {
        parseBoolean(value, defaultValue) {
            if (value === undefined || value === null) return defaultValue;
            if (typeof value === 'boolean') return value;
            if (typeof value === 'string') return value === 'true';
            return defaultValue;
        },
        parseNumber(value, defaultValue) {
            if (value === undefined || value === null) return defaultValue;
            const num = parseFloat(value);
            return isNaN(num) ? defaultValue : num;
        },
        updateCompressConfig(key, value) {
            this.$store.commit('setCompressConfig', { key, value });
        },
        validateUploadFolder(path, strict = true) {
            if (path && !path.startsWith('/')) {
                path = '/' + path;
                this.uploadFolder = path;
            }
            const result = validateFolderPath(path, { strict });
            if (!result.valid) {
                this.$message.error(result.error);
                return false;
            }
            return true;
        },
        handleUploadFolderBlur() {
            if (this.uploadFolder && !this.uploadFolder.startsWith('/')) {
                this.uploadFolder = '/' + this.uploadFolder;
            }
            if (!this.validateUploadFolder(this.uploadFolder, true)) {
                this.$nextTick(() => {
                    this.uploadFolder = this.storeUploadFolder;
                });
            }
        },
        restoreChannelName() {
            const list = this.currentChannelList;
            const savedChannelName = this.storeChannelName;
            const defaultChannelName = this.userConfig?.defaultChannelName;

            if (savedChannelName && list.some(ch => ch.name === savedChannelName)) {
                this.channelName = savedChannelName;
            } else if (savedChannelName !== '' && defaultChannelName && list.some(ch => ch.name === defaultChannelName)) {
                this.channelName = defaultChannelName;
            } else {
                this.channelName = '';
            }
        },
        async fetchAvailableChannels() {
            try {
                const response = await axios.get('/api/channels', { withAuthCode: true });
                if (response.data) {
                    this.availableChannels = response.data;
                }
            } catch (error) {
                console.error('Failed to fetch available channels:', error);
            }
        },
        initFromStore() {
            this.selectedUrlForm = this.uploadCopyUrlForm || 'url';
            this.customerCompress = this.compressConfig.customerCompress ?? this.parseBoolean(this.userConfig?.defaultCustomerCompress, true);
            this.compressQuality = this.compressConfig.compressQuality ?? this.parseNumber(this.userConfig?.defaultCompressQuality, 4);
            this.compressBar = this.compressConfig.compressBar ?? this.parseNumber(this.userConfig?.defaultCompressBar, 5);
            this.serverCompress = this.compressConfig.serverCompress ?? true;
            this.convertToWebp = this.compressConfig.convertToWebp ?? this.parseBoolean(this.userConfig?.defaultConvertToWebp, false);
            this.uploadChannel = this.storeUploadChannel || this.userConfig?.defaultUploadChannel || 'telegram';
            this.autoRetry = this.storeAutoRetry;
            this.uploadNameType = this.storeUploadNameType || this.userConfig?.defaultUploadNameType || 'default';
            this.customUrlPrefix = this.customUrlSettings.customUrlPrefix;
            this.useCustomUrl = this.customUrlSettings.useCustomUrl;
            this.uploadFolder = this.storeUploadFolder || this.userConfig?.defaultUploadFolder || '';
        }
    },
    async mounted() {
        this.hydrating = true;
        this.initFromStore();
        await this.fetchAvailableChannels();
        this.restoreChannelName();
        this.hydrating = false;
        this.loading = false;
    }
};
</script>

<style scoped>
.upload-preferences {
    padding: 18px 0 28px;
    min-height: 500px;
}

.page-header {
    margin-bottom: 20px;
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

.settings-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 14px;
}

.preference-panel {
    position: relative;
    background: var(--flat-surface);
    border: 1px solid var(--flat-border);
    border-radius: 8px;
    padding: 16px;
    overflow: hidden;
}

.panel-wide {
    grid-column: 1 / -1;
}

.workflow-panel::before {
    content: attr(data-step);
    position: absolute;
    top: 14px;
    right: 16px;
    color: var(--flat-text-muted);
    font-size: 12px;
    font-weight: 700;
}

.target-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 12px;
}

.section-content {
    display: grid;
    gap: 12px;
}

.preprocess-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    align-items: start;
}

.setting-item {
    min-width: 0;
    border-color: var(--flat-border);
    background: var(--flat-surface-soft);
    border-radius: 8px;
    align-items: center;
}

.setting-item-stack {
    align-items: flex-start;
    flex-direction: column;
    gap: 12px;
}

.setting-item-full {
    grid-column: 1 / -1;
}

.setting-input {
    width: min(100%, 360px);
    max-width: none;
}

.link-custom-switch {
    margin-top: 4px;
}

@media (max-width: 900px) {
    .settings-grid,
    .target-grid,
    .preprocess-grid {
        grid-template-columns: 1fr;
    }

    .setting-item-full {
        grid-column: auto;
    }
}

@media (max-width: 768px) {
    .upload-preferences {
        padding: 12px;
    }

    .preference-panel {
        padding: 14px;
    }

    .setting-input {
        width: 100%;
    }
}
</style>
