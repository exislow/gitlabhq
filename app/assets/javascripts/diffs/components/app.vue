<script>
import { mapState, mapGetters, mapActions } from 'vuex';
import Icon from '~/vue_shared/components/icon.vue';
import { __ } from '~/locale';
import createFlash from '~/flash';
import LoadingIcon from '../../vue_shared/components/loading_icon.vue';
import CompareVersions from './compare_versions.vue';
import ChangedFiles from './changed_files.vue';
import DiffFile from './diff_file.vue';
import NoChanges from './no_changes.vue';
import HiddenFilesWarning from './hidden_files_warning.vue';

export default {
  name: 'DiffsApp',
  components: {
    Icon,
    LoadingIcon,
    CompareVersions,
    ChangedFiles,
    DiffFile,
    NoChanges,
    HiddenFilesWarning,
  },
  props: {
    endpoint: {
      type: String,
      required: true,
    },
    shouldShow: {
      type: Boolean,
      required: false,
      default: false,
    },
    currentUser: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      activeFile: '',
    };
  },
  computed: {
    ...mapState({
      isLoading: state => state.diffs.isLoading,
      diffFiles: state => state.diffs.diffFiles,
      diffViewType: state => state.diffs.diffViewType,
      mergeRequestDiffs: state => state.diffs.mergeRequestDiffs,
      mergeRequestDiff: state => state.diffs.mergeRequestDiff,
      latestVersionPath: state => state.diffs.latestVersionPath,
      startVersion: state => state.diffs.startVersion,
      commit: state => state.diffs.commit,
      targetBranchName: state => state.diffs.targetBranchName,
      renderOverflowWarning: state => state.diffs.renderOverflowWarning,
      numTotalFiles: state => state.diffs.realSize,
      numVisibleFiles: state => state.diffs.size,
      plainDiffPath: state => state.diffs.plainDiffPath,
      emailPatchPath: state => state.diffs.emailPatchPath,
    }),
    ...mapGetters(['isParallelView']),
    targetBranch() {
      return {
        branchName: this.targetBranchName,
        versionIndex: -1,
        path: '',
      };
    },
    notAllCommentsDisplayed() {
      if (this.commit) {
        return __('Only comments from the following commit are shown below');
      } else if (this.startVersion) {
        return __(
          "Not all comments are displayed because you're comparing two versions of the diff.",
        );
      }
      return __(
        "Not all comments are displayed because you're viewing an old version of the diff.",
      );
    },
    showLatestVersion() {
      if (this.commit) {
        return __('Show latest version of the diff');
      }
      return __('Show latest version');
    },
  },
  watch: {
    diffViewType() {
      this.adjustView();
    },
    shouldShow() {
      this.adjustView();
    },
  },
  mounted() {
    this.setEndpoint(this.endpoint);
    this
      .fetchDiffFiles()
      .catch(() => {
        createFlash(__('Something went wrong on our end. Please try again!'));
      });
  },
  created() {
    this.adjustView();
  },
  methods: {
    ...mapActions(['setEndpoint', 'fetchDiffFiles']),
    setActive(filePath) {
      this.activeFile = filePath;
    },
    unsetActive(filePath) {
      if (this.activeFile === filePath) {
        this.activeFile = '';
      }
    },
    adjustView() {
      if (this.shouldShow && this.isParallelView) {
        window.mrTabs.expandViewContainer();
      } else {
        window.mrTabs.resetViewContainer();
      }
    },
  },
};
</script>

<template>
  <div v-if="shouldShow">
    <div
      v-if="isLoading"
      class="loading"
    >
      <loading-icon />
    </div>
    <div
      v-else
      :class="{ active: shouldShow }"
      id="diffs"
      class="diffs tab-pane"
    >
      <compare-versions
        v-if="!commit && mergeRequestDiffs.length > 1"
        :merge-request-diffs="mergeRequestDiffs"
        :merge-request-diff="mergeRequestDiff"
        :start-version="startVersion"
        :target-branch="targetBranch"
      />

      <hidden-files-warning
        v-if="renderOverflowWarning"
        :visible="numVisibleFiles"
        :total="numTotalFiles"
        :plain-diff-path="plainDiffPath"
        :email-patch-path="emailPatchPath"
      />

      <div
        v-if="commit || startVersion || (mergeRequestDiff && !mergeRequestDiff.latest)"
        class="mr-version-controls"
      >
        <div class="content-block comments-disabled-notif clearfix">
          <i class="fa fa-info-circle"></i>
          {{ notAllCommentsDisplayed }}
          <div class="pull-right">
            <a
              :href="latestVersionPath"
              class="btn btn-sm"
            >
              {{ showLatestVersion }}
            </a>
          </div>
        </div>
      </div>

      <changed-files
        :diff-files="diffFiles"
        :active-file="activeFile"
      />

      <div
        v-if="diffFiles.length > 0"
        class="files"
      >
        <diff-file
          v-for="file in diffFiles"
          :key="file.newPath"
          :file="file"
          :current-user="currentUser"
          @setActive="setActive(file.filePath)"
          @unsetActive="unsetActive(file.filePath)"
        />
      </div>
      <no-changes v-else />
    </div>
  </div>
</template>