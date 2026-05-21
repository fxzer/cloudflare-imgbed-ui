<template>
    <div class="upload-list-item">
        <a :href="file.url" target="_blank" class="upload-list-item-preview">
            <video
                v-if="isVideo(file.name)"
                class="preview-media"
                autoplay
                muted
                playsinline
                loop
            >
                <source :src="file.url" type="video/mp4" />
                Your browser does not support the video tag.
            </video>
            <img
                v-else-if="isImage(file.name)"
                class="preview-media"
                :src="file.url"
                :alt="file.name"
                @error="$emit('preview-error', file)"
            />
            <div v-else class="preview-file-placeholder">
                <font-awesome-icon icon="file" class="file-icon"></font-awesome-icon>
            </div>
        </a>
        <div class="upload-list-item-content">
            <div class="upload-list-item-name-wrapper">
                <span class="upload-list-item-name">{{ file.name }}</span>
            </div>
            <div class="upload-list-item-url" v-if="file.status==='done'">
                <el-input v-model="file.finalURL" readonly @click="selectAllText" :size="urlSize">
                    <template #prepend>URL</template>
                </el-input>
                <el-input v-model="file.mdURL" readonly @click="selectAllText" :size="urlSize">
                    <template #prepend>MarkDown</template>
                </el-input>
                <el-input v-model="file.htmlURL" readonly @click="selectAllText" :size="urlSize">
                    <template #prepend>HTML</template>
                </el-input>
                <el-input v-model="file.ubbURL" readonly @click="selectAllText" :size="urlSize">
                    <template #prepend>BBCode</template>
                </el-input>
            </div>
            <div class="upload-list-item-progress" v-else>
                <el-progress :percentage="file.progreess" :status="file.status" :show-text="false"/>
            </div>
        </div>
        <div class="upload-list-item-action">
            <button class="modern-file-action-btn modern-file-action-btn-primary" @click="$emit('copy', file)">
                <el-icon><Link /></el-icon>
            </button>
            <button class="modern-file-action-btn modern-file-action-btn-danger" @click="$emit('remove', file)">
                <el-icon><Delete /></el-icon>
            </button>
        </div>
    </div>
</template>

<script>
import { Link, Delete } from '@element-plus/icons-vue'

const IMAGE_EXTENSIONS = [
    'jpg', 'jpeg', 'png', 'gif', 'bmp', 'webp', 'svg', 'tiff', 'ico', 'avif', 'heic',
    'jfif', 'pjpeg', 'pjp', 'raw', 'cr2', 'nef', 'dng', 'eps', 'ai', 'emf', 'wmf'
]
const VIDEO_EXTENSIONS = ['mp4', 'webm', 'ogg', 'mkv']

export default {
    name: 'UploadFileItem',
    components: { Link, Delete },
    props: {
        file: { type: Object, required: true }
    },
    emits: ['copy', 'remove', 'preview-error'],
    computed: {
        urlSize() {
            return window.innerWidth < 768 ? 'small' : 'default'
        }
    },
    methods: {
        isImage(fileName) {
            const ext = fileName.split('.').pop().toLowerCase()
            return IMAGE_EXTENSIONS.includes(ext)
        },
        isVideo(fileName) {
            const ext = fileName.split('.').pop().toLowerCase()
            return VIDEO_EXTENSIONS.includes(ext)
        },
        selectAllText(event) {
            navigator.clipboard.writeText(event.target.value)
                .then(() => {
                    this.$message({ type: 'success', message: this.$t('uploadForm.copySuccess') })
                })
                .catch(() => {
                    this.$message({ type: 'error', message: this.$t('uploadForm.copyFailed') })
                })
        }
    }
}
</script>

<style scoped>
.upload-list-item {
    display: flex;
    align-items: stretch;
    justify-content: space-between;
    gap: 12px;
    margin: 8px 10px;
    border: 1px solid var(--upload-list-item-border-color, rgba(64, 158, 255, 0.1));
    padding: 10px 12px;
    border-radius: 16px;
    background: var(--upload-list-item-bg, linear-gradient(135deg, rgba(255, 255, 255, 0.9) 0%, rgba(255, 255, 255, 0.7) 100%));
    backdrop-filter: blur(10px);
    box-shadow: 0 2px 8px var(--upload-list-item-shadow, rgba(0, 0, 0, 0.04));
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.upload-list-item:hover {
    border-color: var(--upload-list-item-hover-border, rgba(64, 158, 255, 0.25));
    box-shadow: 0 4px 16px var(--upload-list-item-hover-shadow, rgba(64, 158, 255, 0.12));
    transform: translateY(-2px);
}
.upload-list-item-preview {
    flex-shrink: 0;
    width: 88px;
    min-height: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    border-radius: 10px;
    background: var(--upload-list-preview-bg, rgba(64, 158, 255, 0.06));
    border: 1px solid var(--upload-list-item-border-color, rgba(64, 158, 255, 0.1));
    text-decoration: none;
    align-self: stretch;
}
.preview-media {
    display: block;
    max-width: 100%;
    max-height: 100%;
    width: auto;
    height: auto;
    object-fit: contain;
    border-radius: 8px;
}
.preview-file-placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    min-height: 48px;
}
.file-icon {
    font-size: 28px;
    color: var(--upload-list-file-icon-color);
}
.upload-list-item-content {
    display: flex;
    flex: 1;
    flex-direction: column;
    min-width: 0;
}
.upload-list-item-action {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    gap: 6px;
    align-self: stretch;
}
.upload-list-item-url {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 8px;
    width: 100%;
}

/* Name Wrapper */
.upload-list-item-name-wrapper {
    display: flex;
    align-items: center;
    width: 100%;
    padding: 8px 16px;
    background: var(--file-name-bg, linear-gradient(135deg, rgba(64, 158, 255, 0.08) 0%, rgba(64, 158, 255, 0.03) 100%));
    border-radius: 10px;
    margin-bottom: 8px;
    border: 1px solid var(--file-name-border, rgba(64, 158, 255, 0.12));
    backdrop-filter: blur(4px);
    transition: all 0.3s ease;
    box-sizing: border-box;
}
.upload-list-item-name-wrapper:hover {
    background: var(--file-name-hover-bg, linear-gradient(135deg, rgba(64, 158, 255, 0.12) 0%, rgba(64, 158, 255, 0.06) 100%));
    border-color: var(--file-name-hover-border, rgba(64, 158, 255, 0.2));
}
.upload-list-item-name {
    font-size: 14px;
    font-weight: 600;
    width: 100%;
    color: var(--el-text-color-primary);
    letter-spacing: 0.3px;
    word-break: break-all;
    line-height: 1.4;
}

/* Progress Bar */
.upload-list-item-progress {
    margin-top: 8px;
    width: 100%;
    padding: 4px 8px;
    background: var(--progress-wrapper-bg, linear-gradient(135deg, rgba(64, 158, 255, 0.05) 0%, rgba(64, 158, 255, 0.02) 100%));
    border-radius: 12px;
    border: 1px solid var(--progress-wrapper-border, rgba(64, 158, 255, 0.1));
}
.upload-list-item-progress :deep(.el-progress) {
    --el-color-primary: #409eff;
}
.upload-list-item-progress :deep(.el-progress-bar) {
    padding-right: 0;
    margin-right: 0;
}
.upload-list-item-progress :deep(.el-progress-bar__outer) {
    height: 10px !important;
    border-radius: 8px;
    background: var(--progress-outer-bg, linear-gradient(135deg, rgba(0, 0, 0, 0.06) 0%, rgba(0, 0, 0, 0.03) 100%));
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.08);
    overflow: hidden;
}
.upload-list-item-progress :deep(.el-progress-bar__inner) {
    border-radius: 8px;
    background: linear-gradient(90deg, #409eff 0%, #66b1ff 50%, #409eff 100%) !important;
    box-shadow: 0 0 12px rgba(64, 158, 255, 0.5), 0 0 4px rgba(64, 158, 255, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.3);
    position: relative;
    overflow: hidden;
    transition: width 0.3s ease;
}
.upload-list-item-progress :deep(.el-progress-bar__inner::after) {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    background: linear-gradient(90deg, transparent 0%, rgba(255, 255, 255, 0.2) 50%, transparent 100%);
    pointer-events: none;
}
.upload-list-item-progress :deep(.el-progress-bar__inner::before) {
    content: '';
    position: absolute;
    top: 0; left: -100%; width: 300%; height: 100%;
    background: repeating-linear-gradient(45deg, transparent, transparent 8px, rgba(255, 255, 255, 0.15) 8px, rgba(255, 255, 255, 0.15) 16px);
    animation: progressStripes 1s linear infinite;
}
.upload-list-item-progress :deep(.el-progress--success .el-progress-bar__inner) {
    background: linear-gradient(90deg, #67c23a 0%, #85ce61 25%, #95d475 50%, #85ce61 75%, #67c23a 100%) !important;
    background-size: 200% 100%;
    box-shadow: 0 0 12px rgba(103, 194, 58, 0.5), 0 0 4px rgba(103, 194, 58, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.3);
    animation: none;
}
.upload-list-item-progress :deep(.el-progress--success .el-progress-bar__inner::before),
.upload-list-item-progress :deep(.el-progress--success .el-progress-bar__inner::after) {
    animation: none;
    background: none;
}
.upload-list-item-progress :deep(.el-progress--exception .el-progress-bar__inner) {
    background: linear-gradient(90deg, #f56c6c 0%, #f78989 25%, #f9a7a7 50%, #f78989 75%, #f56c6c 100%) !important;
    background-size: 200% 100%;
    box-shadow: 0 0 12px rgba(245, 108, 108, 0.5), 0 0 4px rgba(245, 108, 108, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.3);
    animation: progressPulse 1s ease-in-out infinite;
}
.upload-list-item-progress :deep(.el-progress--exception .el-progress-bar__inner::before) {
    animation: none;
    background: none;
}
@keyframes progressStripes {
    0% { transform: translateX(0); }
    100% { transform: translateX(22.627px); }
}
@keyframes progressPulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.7; }
}

/* URL Input Styles */
.upload-list-item-url :deep(.el-input) {
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.upload-list-item-url :deep(.el-input:hover) {
    transform: translateY(-1px);
}
.upload-list-item-url :deep(.el-input__wrapper) {
    border-radius: 10px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    background: var(--el-fill-color-blank);
    border: 1px solid var(--el-border-color-lighter);
    overflow: hidden;
    position: relative;
    padding: 0;
}
.upload-list-item-url :deep(.el-input-group > .el-input__wrapper) {
    border-radius: 0 9px 9px 0 !important;
}
.upload-list-item-url :deep(.el-input__wrapper:hover) {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.12);
    border-color: var(--el-color-primary-light-5);
}
.upload-list-item-url :deep(.el-input__wrapper.is-focus) {
    box-shadow: 0 0 0 2px var(--el-color-primary-light-8), 0 4px 12px rgba(0, 0, 0, 0.15);
    border-color: var(--el-color-primary);
}
.upload-list-item-url :deep(.el-input__wrapper.is-focus::before) {
    content: '';
    position: absolute;
    top: 0; left: -100%; width: 100%; height: 100%;
    background: linear-gradient(90deg, transparent, rgba(64, 158, 255, 0.08), transparent);
    animation: shimmer 2s infinite;
    z-index: 0;
}
@keyframes shimmer {
    0% { left: -100%; }
    100% { left: 100%; }
}
.upload-list-item-url :deep(.el-input__inner) {
    font-size: 13px;
    font-family: 'Courier New', Monaco, monospace;
    color: var(--el-text-color-regular);
    transition: all 0.3s ease;
    padding-left: 12px;
    position: relative;
    z-index: 1;
    border-radius: 0 10px 10px 0;
}
.upload-list-item-url :deep(.el-input__inner::selection) {
    background-color: var(--el-color-primary-light-7);
}
.upload-list-item-url :deep(.el-input-group__prepend) {
    background: var(--el-color-primary-light-9);
    color: var(--el-color-primary);
    font-weight: 600;
    font-size: 12px;
    border: none;
    padding: 0 14px;
    margin: 0;
    border-radius: 9px 0 0 9px;
    box-shadow: none;
    transition: all 0.3s ease;
    letter-spacing: 0.5px;
    position: relative;
    z-index: 1;
}
.upload-list-item-url :deep(.el-input-group__prepend::after) {
    content: '';
    position: absolute;
    right: 0; top: 50%;
    transform: translateY(-50%);
    height: 60%; width: 1px;
    background: var(--el-color-primary-light-7);
    opacity: 0.3;
    transition: all 0.3s ease;
}
.upload-list-item-url :deep(.el-input:hover .el-input-group__prepend) {
    background: var(--el-color-primary-light-8);
}
.upload-list-item-url :deep(.el-input:hover .el-input-group__prepend::after) {
    opacity: 0.5;
}
.upload-list-item-url :deep(.el-input.is-focus .el-input-group__prepend) {
    background: var(--el-color-primary);
    color: white;
    animation: prependPulse 2s ease-in-out infinite;
}
.upload-list-item-url :deep(.el-input.is-focus .el-input-group__prepend::after) {
    background: rgba(255, 255, 255, 0.3);
    opacity: 1;
}
@keyframes prependPulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.95; }
}

/* File Action Buttons */
.modern-file-action-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 30px;
    height: 30px;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    font-size: 16px;
    position: relative;
    overflow: hidden;
    margin: 4px 0;
}
.modern-file-action-btn::before {
    content: '';
    position: absolute;
    top: 50%; left: 50%;
    width: 0; height: 0;
    background: rgba(255, 255, 255, 0.3);
    border-radius: 50%;
    transform: translate(-50%, -50%);
    transition: width 0.4s ease, height 0.4s ease;
}
.modern-file-action-btn:active::before {
    width: 100%;
    height: 100%;
}
.modern-file-action-btn-primary {
    background: var(--file-action-primary-bg, linear-gradient(145deg, #409eff 0%, #53a8ff 50%, #66b1ff 100%));
    color: white;
    box-shadow: 0 3px 10px rgba(64, 158, 255, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.2);
}
.modern-file-action-btn-primary:hover {
    transform: translateY(-3px) scale(1.08);
    box-shadow: 0 6px 20px rgba(64, 158, 255, 0.45), inset 0 1px 0 rgba(255, 255, 255, 0.25);
}
.modern-file-action-btn-primary:active {
    transform: translateY(-1px) scale(1.02);
}
.modern-file-action-btn-danger {
    background: var(--file-action-danger-bg, linear-gradient(145deg, #f56c6c 0%, #f78989 50%, #f9a7a7 100%));
    color: white;
    box-shadow: 0 3px 10px rgba(245, 108, 108, 0.3), inset 0 1px 0 rgba(255, 255, 255, 0.2);
}
.modern-file-action-btn-danger:hover {
    transform: translateY(-3px) scale(1.08);
    box-shadow: 0 6px 20px rgba(245, 108, 108, 0.45), inset 0 1px 0 rgba(255, 255, 255, 0.25);
}
.modern-file-action-btn-danger:active {
    transform: translateY(-1px) scale(1.02);
}

/* Mobile */
@media (max-width: 768px) {
    .upload-list-item-preview {
        width: 64px;
    }
    .file-icon {
        font-size: 22px;
    }
    .upload-list-item-url {
        grid-template-columns: 1fr;
    }
    .upload-list-item-progress {
        padding: 3px 6px;
    }
    .upload-list-item-progress :deep(.el-progress-bar__outer) {
        height: 8px !important;
    }
    .upload-list-item-url :deep(.el-input__wrapper) {
        border-radius: 8px;
    }
    .upload-list-item-url :deep(.el-input__inner) {
        font-size: 12px;
    }
    .upload-list-item-url :deep(.el-input-group__prepend) {
        font-size: 11px;
        padding: 0 8px;
        border-radius: 8px 0 0 8px;
    }
    .modern-file-action-btn {
        width: 34px;
        height: 34px;
        border-radius: 10px;
        font-size: 14px;
    }
    .upload-list-item-name-wrapper {
        padding: 4px 10px;
        border-radius: 8px;
    }
    .upload-list-item-name {
        font-size: 12px;
    }
}
</style>
