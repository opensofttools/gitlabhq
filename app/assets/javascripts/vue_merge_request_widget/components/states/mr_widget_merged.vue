<script>
  import Flash from '~/flash';
  import tooltip from '~/vue_shared/directives/tooltip';
  import loadingIcon from '~/vue_shared/components/loading_icon.vue';
  import { s__, __ } from '~/locale';
  import ClipboardButton from '~/vue_shared/components/clipboard_button.vue';
  import mrWidgetAuthorTime from '../../components/mr_widget_author_time.vue';
  import statusIcon from '../mr_widget_status_icon.vue';
  import eventHub from '../../event_hub';

  export default {
    name: 'MRWidgetMerged',
    directives: {
      tooltip,
    },
    components: {
      mrWidgetAuthorTime,
      loadingIcon,
      statusIcon,
      ClipboardButton,
    },
    props: {
      mr: {
        type: Object,
        required: true,
        default: () => ({}),
      },
      service: {
        type: Object,
        required: true,
        default: () => ({}),
      },
    },
    data() {
      return {
        isMakingRequest: false,
      };
    },
    computed: {
      shouldShowRemoveSourceBranch() {
        const {
          sourceBranchRemoved,
          isRemovingSourceBranch,
          canRemoveSourceBranch,
        } = this.mr;

        return !sourceBranchRemoved &&
          canRemoveSourceBranch &&
          !this.isMakingRequest &&
          !isRemovingSourceBranch;
      },
      shouldShowSourceBranchRemoving() {
        const {
          sourceBranchRemoved,
          isRemovingSourceBranch,
        } = this.mr;
        return !sourceBranchRemoved &&
          (isRemovingSourceBranch || this.isMakingRequest);
      },
      shouldShowMergedButtons() {
        const {
          canRevertInCurrentMR,
          canCherryPickInCurrentMR,
          revertInForkPath,
          cherryPickInForkPath,
        } = this.mr;

        return canRevertInCurrentMR ||
          canCherryPickInCurrentMR ||
          revertInForkPath ||
          cherryPickInForkPath;
      },
      revertTitle() {
        return s__('mrWidget|Revert this merge request in a new merge request');
      },
      cherryPickTitle() {
        return s__('mrWidget|Cherry-pick this merge request in a new merge request');
      },
      revertLabel() {
        return s__('mrWidget|Revert');
      },
      cherryPickLabel() {
        return s__('mrWidget|Cherry-pick');
      },
    },
    methods: {
      removeSourceBranch() {
        this.isMakingRequest = true;

        this.service.removeSourceBranch()
          .then(res => res.data)
          .then((data) => {
            if (data.message === 'Branch was removed') {
              eventHub.$emit('MRWidgetUpdateRequested', () => {
                this.isMakingRequest = false;
              });
            }
          })
          .catch(() => {
            this.isMakingRequest = false;
            Flash(__('Something went wrong. Please try again.'));
          });
      },
    },
  };
</script>
<template>
  <div class="mr-widget-body media">
    <status-icon status="success" />
    <div class="media-body">
      <div class="space-children">
        <mr-widget-author-time
          :action-text="s__('mrWidget|Merged by')"
          :author="mr.metrics.mergedBy"
          :date-title="mr.metrics.mergedAt"
          :date-readable="mr.metrics.readableMergedAt"
        />
        <a
          v-if="mr.canRevertInCurrentMR"
          v-tooltip
          class="btn btn-close btn-sm"
          href="#modal-revert-commit"
          data-toggle="modal"
          data-container="body"
          :title="revertTitle"
        >
          {{ revertLabel }}
        </a>
        <a
          v-else-if="mr.revertInForkPath"
          v-tooltip
          class="btn btn-close btn-sm"
          data-method="post"
          :href="mr.revertInForkPath"
          :title="revertTitle"
        >
          {{ revertLabel }}
        </a>
        <a
          v-if="mr.canCherryPickInCurrentMR"
          v-tooltip
          class="btn btn-default btn-sm"
          href="#modal-cherry-pick-commit"
          data-toggle="modal"
          data-container="body"
          :title="cherryPickTitle"
        >
          {{ cherryPickLabel }}
        </a>
        <a
          v-else-if="mr.cherryPickInForkPath"
          v-tooltip
          class="btn btn-default btn-sm"
          data-method="post"
          :href="mr.cherryPickInForkPath"
          :title="cherryPickTitle"
        >
          {{ cherryPickLabel }}
        </a>
      </div>
      <section class="mr-info-list">
        <p>
          {{ s__("mrWidget|The changes were merged into") }}
          <span class="label-branch">
            <a :href="mr.targetBranchPath">{{ mr.targetBranch }}</a>
          </span>
          with
          <a
            :href="mr.mergeCommitPath"
            class="commit-sha js-mr-merged-commit-sha"
          >
            {{ mr.shortMergeCommitSha }}
          </a>
          <clipboard-button
            :title="__('Copy commit SHA to clipboard')"
            :text="mr.shortMergeCommitSha"
            css-class="btn-default btn-transparent btn-clipboard js-mr-merged-copy-sha"
          />
        </p>
        <p v-if="mr.sourceBranchRemoved">
          {{ s__("mrWidget|The source branch has been removed") }}
        </p>
        <p
          v-if="shouldShowRemoveSourceBranch"
          class="space-children"
        >
          <span>{{ s__("mrWidget|You can remove source branch now") }}</span>
          <button
            @click="removeSourceBranch"
            :disabled="isMakingRequest"
            type="button"
            class="btn btn-sm btn-default js-remove-branch-button"
          >
            {{ s__("mrWidget|Remove Source Branch") }}
          </button>
        </p>
        <p v-if="shouldShowSourceBranchRemoving">
          <loading-icon :inline="true" />
          <span>
            {{ s__("mrWidget|The source branch is being removed") }}
          </span>
        </p>
      </section>
    </div>
  </div>
</template>
