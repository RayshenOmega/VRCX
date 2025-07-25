<template>
    <safe-dialog
        class="x-dialog"
        :visible="isLaunchOptionsDialogVisible"
        :title="t('dialog.launch_options.header')"
        width="600px"
        @close="closeDialog">
        <div style="font-size: 12px">
            {{ t('dialog.launch_options.description') }} <br />
            {{ t('dialog.launch_options.example') }}
            <el-tag size="mini"
                >--fps=144 --enable-debug-gui --enable-sdk-log-levels --enable-udon-debug-logging
            </el-tag>
        </div>

        <el-input
            v-model="launchOptionsDialog.launchArguments"
            type="textarea"
            size="mini"
            show-word-limit
            :autosize="{ minRows: 2, maxRows: 5 }"
            placeholder=""
            style="margin-top: 10px">
        </el-input>

        <template v-if="!isLinux">
            <div style="font-size: 12px; margin-top: 10px">
                {{ t('dialog.launch_options.path_override') }}
            </div>

            <el-input
                v-model="launchOptionsDialog.vrcLaunchPathOverride"
                type="textarea"
                placeholder="C:\Program Files (x86)\Steam\steamapps\common\VRChat"
                :rows="1"
                style="display: block; margin-top: 10px">
            </el-input>
        </template>

        <template #footer>
            <div style="display: flex">
                <el-button size="small" @click="openExternalLink('https://docs.vrchat.com/docs/launch-options')">
                    {{ t('dialog.launch_options.vrchat_docs') }}
                </el-button>
                <el-button
                    size="small"
                    @click="openExternalLink('https://docs.unity3d.com/Manual/CommandLineArguments.html')">
                    {{ t('dialog.launch_options.unity_manual') }}
                </el-button>
                <el-button type="primary" size="small" style="margin-left: auto" @click="updateLaunchOptions">
                    {{ t('dialog.launch_options.save') }}
                </el-button>
            </div>
        </template>
    </safe-dialog>
</template>

<script setup>
    import { storeToRefs } from 'pinia';
    import { computed, getCurrentInstance, ref } from 'vue';
    import { useI18n } from 'vue-i18n-bridge';
    import configRepository from '../../../service/config';
    import { openExternalLink } from '../../../shared/utils';
    import { useLaunchStore } from '../../../stores';

    const { t } = useI18n();

    const instance = getCurrentInstance();
    const $message = instance.proxy.$message;

    const launchStore = useLaunchStore();
    const { isLaunchOptionsDialogVisible } = storeToRefs(launchStore);

    const launchOptionsDialog = ref({
        launchArguments: '',
        vrcLaunchPathOverride: ''
    });

    const isLinux = computed(() => LINUX);

    function init() {
        configRepository
            .getString('launchArguments')
            .then((launchArguments) => (launchOptionsDialog.value.launchArguments = launchArguments));

        configRepository.getString('vrcLaunchPathOverride').then((vrcLaunchPathOverride) => {
            if (vrcLaunchPathOverride === null || vrcLaunchPathOverride === 'null') {
                launchOptionsDialog.value.vrcLaunchPathOverride = '';
                configRepository.setString('vrcLaunchPathOverride', '');
            } else {
                launchOptionsDialog.value.vrcLaunchPathOverride = vrcLaunchPathOverride;
            }
        });
    }

    // created
    init();

    function updateLaunchOptions() {
        const D = launchOptionsDialog.value;
        D.launchArguments = String(D.launchArguments).replace(/\s+/g, ' ').trim();
        configRepository.setString('launchArguments', D.launchArguments);
        if (
            D.vrcLaunchPathOverride &&
            D.vrcLaunchPathOverride.endsWith('.exe') &&
            !D.vrcLaunchPathOverride.endsWith('launch.exe')
        ) {
            $message({
                message: 'Invalid path, you must enter VRChat folder or launch.exe',
                type: 'error'
            });
            return;
        }
        configRepository.setString('vrcLaunchPathOverride', D.vrcLaunchPathOverride);
        $message({
            message: 'Updated launch options',
            type: 'success'
        });
        closeDialog();
    }

    function closeDialog() {
        isLaunchOptionsDialogVisible.value = false;
    }
</script>
