<template>
  <tr
      :data-pivot-id="resource.id.pivotValue"
      @click.stop.prevent="handleClick"
      class="group"
      :class="{
      'divide-x divide-gray-100 dark:divide-gray-700': shouldShowColumnBorders,
    }"
      :dusk="`${resource.id.value}-row`"
  >
    <!-- Resource Selection Checkbox -->
    <td
        v-if="showShowCheckboxesRow"
        @click.stop
        class="w-[1%] white-space-nowrap pl-5 pr-5 dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
        :class="{
        'py-2': !shouldShowTight,
        'cursor-pointer': resource.authorizedToView,
      }"
    >
      <Checkbox
          v-if="shouldShowCheckboxes"
          :model-value="checked"
          @change="toggleSelection"
          :dusk="`${resource.id.value}-checkbox`"
          :aria-label="__('Select Resource :title', { title: resource.title })"
      />
    </td>

    <td
        :class="{
        'py-2': !shouldShowTight,
        'cursor-pointer': resource.authorizedToView,
      }"
        class="px-2 w-[1%] white-space-nowrap text-right align-middle dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
    >
      <div class="flex items-center justify-end space-x-0 text-gray-400">
        <InlineActionDropdown
            v-if="shouldShowActionDropdown"
            :actions="availableActions"
            :endpoint="actionsEndpoint"
            :resource="resource"
            :resource-name="resourceName"
            :via-many-to-many="viaManyToMany"
            :via-resource="viaResource"
            :via-resource-id="viaResourceId"
            :via-relationship="viaRelationship"
            @actionExecuted="$emit('actionExecuted')"
            @show-preview="navigateToPreviewView"
        />

        <!-- View Resource Link -->
        <Link
            v-if="authorizedToViewAnyResources"
            :as="!resource.authorizedToView ? 'button' : 'a'"
            :href="viewURL"
            :disabled="!resource.authorizedToView ? true : null"
            @click.stop
            class="inline-flex items-center justify-center h-9 w-9"
            :class="
            resource.authorizedToView
              ? 'text-gray-500 dark:text-gray-400 hover:[&:not(:disabled)]:text-primary-500 dark:hover:[&:not(:disabled)]:text-primary-500'
              : 'disabled:cursor-not-allowed disabled:opacity-50'
          "
            :dusk="`${resource['id'].value}-view-button`"
            :aria-label="__('View')"
            v-tooltip.click="__('View')"
        >
          <span class="flex items-center gap-1">
            <span>
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" aria-hidden="true" data-slot="icon" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M2.036 12.322a1.012 1.012 0 0 1 0-.639C3.423 7.51 7.36 4.5 12 4.5c4.638 0 8.573 3.007 9.963 7.178.07.207.07.431 0 .639C20.577 16.49 16.64 19.5 12 19.5c-4.638 0-8.573-3.007-9.963-7.178Z"></path><path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z"></path></svg>
            </span>
          </span>
        </Link>

        <!-- Edit Button -->
        <Link
            v-if="authorizedToUpdateAnyResources"
            :as="!resource.authorizedToUpdate ? 'button' : 'a'"
            :href="updateURL"
            :disabled="!resource.authorizedToUpdate ? true : null"
            @click.stop
            class="inline-flex items-center justify-center h-9 w-9"
            :class="
            resource.authorizedToUpdate
              ? 'text-gray-500 dark:text-gray-400 hover:[&:not(:disabled)]:text-primary-500 dark:hover:[&:not(:disabled)]:text-primary-500'
              : 'disabled:cursor-not-allowed disabled:opacity-50'
          "
            :dusk="
            viaManyToMany
              ? `${resource['id'].value}-edit-attached-button`
              : `${resource['id'].value}-edit-button`
          "
            :aria-label="viaManyToMany ? __('Edit Attached') : __('Edit')"
            v-tooltip.click="viaManyToMany ? __('Edit Attached') : __('Edit')"
        >
          <span class="flex items-center gap-1">
            <span>
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" aria-hidden="true" data-slot="icon" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L10.582 16.07a4.5 4.5 0 0 1-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 0 1 1.13-1.897l8.932-8.931Zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0 1 15.75 21H5.25A2.25 2.25 0 0 1 3 18.75V8.25A2.25 2.25 0 0 1 5.25 6H10"></path></svg>
            </span>
          </span>
        </Link>

        <!-- Delete Resource Link -->
        <button
            v-if="
            authorizedToDeleteAnyResources &&
            (!resource.softDeleted || viaManyToMany)
          "
            @click.stop="openDeleteModal"
            v-tooltip.click="__(viaManyToMany ? 'Detach' : 'Delete')"
            :aria-label="__(viaManyToMany ? 'Detach' : 'Delete')"
            :dusk="`${resource.id.value}-delete-button`"
            :disabled="!resource.authorizedToDelete"
            type="button"
            class="inline-flex items-center justify-center h-9 w-9 text-gray-500 dark:text-gray-400 hover:[&:not(:disabled)]:text-danger-500 dark:hover:[&:not(:disabled)]:text-danger-500 disabled:cursor-not-allowed disabled:opacity-50"
        >
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" aria-hidden="true" data-slot="icon" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0"></path></svg>
        </button>

        <!-- Restore Resource Link -->
        <button
            v-if="
            authorizedToRestoreAnyResources &&
            resource.softDeleted &&
            !viaManyToMany
          "
            v-tooltip.click="__('Restore')"
            :aria-label="__('Restore')"
            :disabled="!resource.authorizedToRestore"
            :dusk="`${resource.id.value}-restore-button`"
            type="button"
            @click.stop="openRestoreModal"
            class="inline-flex items-center justify-center h-9 w-9 text-gray-500 dark:text-gray-400 hover:[&:not(:disabled)]:text-primary-500 dark:hover:[&:not(:disabled)]:text-primary-500 disabled:cursor-not-allowed disabled:opacity-50"
        >
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" aria-hidden="true" data-slot="icon" class="w-6 h-6"><path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99"></path></svg>
        </button>

        <DeleteResourceModal
            :mode="viaManyToMany ? 'detach' : 'delete'"
            :resource-name="resourceName"
            :show="deleteModalOpen"
            @close="closeDeleteModal"
            @confirm="confirmDelete"
        />

        <RestoreResourceModal
            :resource-name="resourceName"
            :show="restoreModalOpen"
            @close="closeRestoreModal"
            @confirm="confirmRestore"
        >
          <ModalContent>
            <p class="leading-normal">
              {{ __('Are you sure you want to restore this resource?') }}
            </p>
          </ModalContent>
        </RestoreResourceModal>
      </div>
    </td>

    <!-- Fields -->
    <td
        v-for="(field, index) in resource.fields"
        :key="field.uniqueKey"
        class="dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
        :class="{
        'px-6': index === 0 && !shouldShowCheckboxes,
        'px-2': index !== 0 || shouldShowCheckboxes,
        'py-2': !shouldShowTight,
        'whitespace-nowrap': !field.wrapping,
        'cursor-pointer': clickableRow,
      }"
    >
      <component
          :is="'index-' + field.component"
          :class="`text-${field.textAlign}`"
          :field="field"
          :resource="resource"
          :resource-name="resourceName"
          :via-resource="viaResource"
          :via-resource-id="viaResourceId"
      />
    </td>
  </tr>

  <PreviewResourceModal
      v-if="previewModalOpen"
      :resource-id="resource.id.value"
      :resource-name="resourceName"
      :show="previewModalOpen"
      @close="closePreviewModal"
      @confirm="closePreviewModal"
  />
</template>

<script>
import { router } from '@inertiajs/vue3'

export default {
  emits: ['actionExecuted'],

  inject: [
    'resourceHasId',
    'authorizedToViewAnyResources',
    'authorizedToUpdateAnyResources',
    'authorizedToDeleteAnyResources',
    'authorizedToRestoreAnyResources',
  ],

  props: [
    'actionsAreAvailable',
    'actionsEndpoint',
    'checked',
    'clickAction',
    'deleteResource',
    'queryString',
    'relationshipType',
    'resource',
    'resourceName',
    'resourcesSelected',
    'restoreResource',
    'selectedResources',
    'shouldShowCheckboxes',
    'shouldShowSelectAllCheckboxes',
    'shouldShowColumnBorders',
    'tableStyle',
    'testId',
    'updateSelectionStatus',
    'viaManyToMany',
    'viaRelationship',
    'viaResource',
    'viaResourceId',
  ],

  data: () => ({
    commandPressed: false,
    deleteModalOpen: false,
    restoreModalOpen: false,
    previewModalOpen: false,
  }),

  beforeMount() {
    this.isSelected = this.selectedResources.indexOf(this.resource) > -1
  },

  mounted() {
    window.addEventListener('keydown', this.handleKeydown)
    window.addEventListener('keyup', this.handleKeyup)
  },

  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown)
    window.removeEventListener('keyup', this.handleKeyup)
  },

  methods: {
    /**
     * Select the resource in the parent component
     */
    toggleSelection() {
      this.updateSelectionStatus(this.resource)
    },

    handleKeydown(e) {
      if (e.key === 'Meta' || e.key === 'Control') {
        this.commandPressed = true
      }
    },

    handleKeyup(e) {
      if (e.key === 'Meta' || e.key === 'Control') {
        this.commandPressed = false
      }
    },

    handleClick(e) {
      if (this.resourceHasId === false) {
        return
      } else if (this.clickAction === 'edit') {
        return this.navigateToEditView(e)
      } else if (this.clickAction === 'select') {
        return this.toggleSelection()
      } else if (this.clickAction === 'ignore') {
        return
      } else if (this.clickAction === 'detail') {
        return this.navigateToDetailView(e)
      } else if (this.clickAction === 'preview') {
        return this.navigateToPreviewView(e)
      } else {
        return this.navigateToDetailView(e)
      }
    },

    navigateToDetailView(e) {
      if (!this.resource.authorizedToView) {
        return
      }
      this.commandPressed
          ? window.open(this.viewURL, '_blank')
          : router.visit(this.viewURL)
    },

    navigateToEditView(e) {
      if (!this.resource.authorizedToUpdate) {
        return
      }
      this.commandPressed
          ? window.open(this.updateURL, '_blank')
          : router.visit(this.updateURL)
    },

    navigateToPreviewView(e) {
      if (!this.resource.authorizedToView) {
        return
      }
      this.openPreviewModal()
    },

    openPreviewModal() {
      this.previewModalOpen = true
    },

    closePreviewModal() {
      this.previewModalOpen = false
    },

    openDeleteModal() {
      this.deleteModalOpen = true
    },

    confirmDelete() {
      this.deleteResource(this.resource)
      this.closeDeleteModal()
    },

    closeDeleteModal() {
      this.deleteModalOpen = false
    },

    openRestoreModal() {
      this.restoreModalOpen = true
    },

    confirmRestore() {
      this.restoreResource(this.resource)
      this.closeRestoreModal()
    },

    closeRestoreModal() {
      this.restoreModalOpen = false
    },
  },

  computed: {
    currentUser() {
      return this.$store?.getters?.currentUser ?? {}
    },

    updateURL() {
      if (this.viaManyToMany) {
        return this.$url(
            `/resources/${this.viaResource}/${this.viaResourceId}/edit-attached/${this.resourceName}/${this.resource.id.value}`,
            {
              viaRelationship: this.viaRelationship,
              viaPivotId: this.resource.id.pivotValue,
            }
        )
      }

      return this.$url(
          `/resources/${this.resourceName}/${this.resource.id.value}/edit`,
          {
            viaResource: this.viaResource,
            viaResourceId: this.viaResourceId,
            viaRelationship: this.viaRelationship,
          }
      )
    },

    viewURL() {
      return this.$url(
          `/resources/${this.resourceName}/${this.resource.id.value}`
      )
    },

    availableActions() {
      return this.resource.actions.filter(a => a.showOnTableRow)
    },

    shouldShowTight() {
      return this.tableStyle === 'tight'
    },

    clickableRow() {
      if (this.resourceHasId === false) {
        return false
      } else if (this.clickAction === 'edit') {
        return this.resource.authorizedToUpdate
      } else if (this.clickAction === 'select') {
        return this.shouldShowCheckboxes
      } else if (this.clickAction === 'ignore') {
        return false
      } else if (this.clickAction === 'detail') {
        return this.resource.authorizedToView
      } else if (this.clickAction === 'preview') {
        return this.resource.authorizedToView
      } else {
        return this.resource.authorizedToView
      }
    },

    showShowCheckboxesRow() {
      return this.shouldShowSelectAllCheckboxes || this.shouldShowCheckboxes
    },

    shouldShowActionDropdown() {
      return this.availableActions.length > 0 || this.userHasAnyOptions
    },

    shouldShowPreviewLink() {
      return this.resource.authorizedToView && this.resource.previewHasFields
    },

    userHasAnyOptions() {
      return (
          this.resourceHasId &&
          (this.resource.authorizedToReplicate ||
              this.shouldShowPreviewLink ||
              this.canBeImpersonated)
      )
    },

    canBeImpersonated() {
      return (
          Boolean(this.currentUser.canImpersonate) &&
          this.resource.authorizedToImpersonate
      )
    },
  },
}
</script>
