<template>
  <tr
    :data-pivot-id="resource['id'].pivotValue"
    :dusk="resource['id'].value + '-row'"
    class="group"
    :class="{
      'divide-x divide-gray-100 dark:divide-gray-700': shouldShowColumnBorders,
    }"
    @click.stop.prevent="navigateToDetail"
  >
    <!-- Resource Selection Checkbox -->
    <td
      v-if="shouldShowCheckboxes"
      :class="{
        'py-2': !shouldShowTight,
        'cursor-pointer': resource.authorizedToView,
      }"
      class="td-fit pl-5 pr-5 dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
      @click.stop
    >
      <Checkbox
        v-if="shouldShowCheckboxes"
        :aria-label="__('Select Resource :title', { title: resource.title })"
        :checked="checked"
        :data-testid="`${testId}-checkbox`"
        :dusk="`${resource['id'].value}-checkbox`"
        @input="toggleSelection"
      />
    </td>

    <td
      :class="{
        'py-2': !shouldShowTight,
        'cursor-pointer': resource.authorizedToView,
      }"
      class="px-2 td-fit text-right align-middle dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
    >
      <div class="flex items-center justify-end space-x-0 text-gray-400">
        <InlineActionDropdown
          v-if="availableActions.length > 0"
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
          :disabled="!resource.authorizedToView"
          v-tooltip.click="__('View')"
          :aria-label="__('View')"
          :dusk="`${resource['id'].value}-view-button`"
          :href="viewURL"
          class="toolbar-button hover:text-primary-500 px-2 disabled:opacity-50 disabled:pointer-events-none"
          @click.stop
        >
          <svg
            class="h-4 w-4"
            viewBox="0 0 20 20"
            fill="currentColor"
            aria-hidden="true"
          >
            <path d="M10 3C6.364 3 3.259 5.29 2 8.5 3.259 11.71 6.364 14 10 14s6.741-2.29 8-5.5C16.741 5.29 13.636 3 10 3Zm0 9a3.5 3.5 0 1 1 0-7 3.5 3.5 0 0 1 0 7Z" />
            <path d="M10 10.5a2 2 0 1 0 0-4 2 2 0 0 0 0 4Z" />
          </svg>
        </Link>

        <!-- Edit Pivot Button -->
        <Link
          v-if="authorizedToUpdateAnyResources && viaManyToMany"
          :as="!resource.authorizedToUpdate ? 'button' : 'a'"
          :disabled="!resource.authorizedToUpdate"
          v-tooltip.click="__('Edit Attached')"
          :aria-label="__('Edit Attached')"
          :dusk="`${resource['id'].value}-edit-attached-button`"
          :href="updateURL"
          class="toolbar-button hover:text-primary-500 px-2 disabled:opacity-50 disabled:pointer-events-none"
          @click.stop
        >
          <svg
            class="h-4 w-4"
            viewBox="0 0 20 20"
            fill="currentColor"
            aria-hidden="true"
          >
            <path d="m13.828 2.172 4 4a1 1 0 0 1 0 1.414l-9.193 9.192a1 1 0 0 1-.465.263l-4 1a1 1 0 0 1-1.212-1.212l1-4a1 1 0 0 1 .263-.465l9.193-9.193a1 1 0 0 1 1.414 0ZM12.414 5 5.849 11.565l-.646 2.586 2.586-.646L14.354 6.94 12.414 5Z" />
          </svg>
        </Link>

        <!-- Edit Resource Link -->
        <Link
          v-else-if="authorizedToUpdateAnyResources"
          :as="!resource.authorizedToUpdate ? 'button' : 'a'"
          :disabled="!resource.authorizedToUpdate"
          v-tooltip.click="__('Edit')"
          :aria-label="__('Edit')"
          :dusk="`${resource['id'].value}-edit-button`"
          :href="updateURL"
          class="toolbar-button hover:text-primary-500 px-2 disabled:opacity-50 disabled:pointer-events-none"
          @click.stop
        >
          <svg
            class="h-4 w-4"
            viewBox="0 0 20 20"
            fill="currentColor"
            aria-hidden="true"
          >
            <path d="m13.828 2.172 4 4a1 1 0 0 1 0 1.414l-9.193 9.192a1 1 0 0 1-.465.263l-4 1a1 1 0 0 1-1.212-1.212l1-4a1 1 0 0 1 .263-.465l9.193-9.193a1 1 0 0 1 1.414 0ZM12.414 5 5.849 11.565l-.646 2.586 2.586-.646L14.354 6.94 12.414 5Z" />
          </svg>
        </Link>

        <!-- Delete Resource Link -->
        <button
          v-if="
            authorizedToDeleteAnyResources &&
            (!resource.softDeleted || viaManyToMany)
          "
          v-tooltip.click="__(viaManyToMany ? 'Detach' : 'Delete')"
          :aria-label="__(viaManyToMany ? 'Detach' : 'Delete')"
          :data-testid="`${testId}-delete-button`"
          :disabled="!resource.authorizedToDelete"
          :dusk="`${resource['id'].value}-delete-button`"
          class="toolbar-button hover:text-primary-500 px-2 disabled:opacity-50 disabled:pointer-events-none"
          @click.stop="openDeleteModal"
        >
          <svg
            class="h-4 w-4"
            viewBox="0 0 20 20"
            fill="currentColor"
            aria-hidden="true"
          >
            <path d="M7 2a1 1 0 0 0-1 1v1H3.5a1 1 0 1 0 0 2h.544l.82 9.015A2 2 0 0 0 6.856 17h6.288a2 2 0 0 0 1.992-1.985L15.956 6h.544a1 1 0 1 0 0-2H14V3a1 1 0 0 0-1-1H7Zm5 2H8v0h4v0Zm-3 4a1 1 0 0 0-1 1v5a1 1 0 1 0 2 0V9a1 1 0 0 0-1-1Zm4 1a1 1 0 1 0-2 0v5a1 1 0 1 0 2 0V9Z" />
          </svg>
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
          :dusk="`${resource['id'].value}-restore-button`"
          class="toolbar-button hover:text-primary-500 px-2 disabled:opacity-50 disabled:pointer-events-none"
          @click.stop="openRestoreModal"
        >
          <svg
            class="h-4 w-4"
            viewBox="0 0 20 20"
            fill="currentColor"
            aria-hidden="true"
          >
            <path d="M4.457 4.457A7 7 0 0 1 15.95 6H14a1 1 0 1 0 0 2h4a1 1 0 0 0 1-1V3a1 1 0 1 0-2 0v1.382A9 9 0 1 0 19 10a1 1 0 1 0-2 0 7 7 0 1 1-12.543-5.543Z" />
          </svg>
        </button>

        <DeleteResourceModal
          :mode="viaManyToMany ? 'detach' : 'delete'"
          :show="deleteModalOpen"
          @close="closeDeleteModal"
          @confirm="confirmDelete"
        />

        <RestoreResourceModal
          :show="restoreModalOpen"
          @close="closeRestoreModal"
          @confirm="confirmRestore"
        >
          <ModalHeader v-text="__('Restore Resource')" />
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
      :class="{
        'px-6': index == 0 && !shouldShowCheckboxes,
        'px-2': index != 0 || shouldShowCheckboxes,
        'py-2': !shouldShowTight,
        'whitespace-nowrap': !field.wrapping,
        'cursor-pointer': clickableRow,
      }"
      class="dark:bg-gray-800 group-hover:bg-gray-50 dark:group-hover:bg-gray-900"
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
import filter from 'lodash/filter'
import { router as Inertia } from '@inertiajs/vue3'

export default {
  emits: ['actionExecuted'],

  inject: [
    'authorizedToViewAnyResources',
    'authorizedToUpdateAnyResources',
    'authorizedToDeleteAnyResources',
    'authorizedToRestoreAnyResources',
  ],

  props: [
    'testId',
    'deleteResource',
    'restoreResource',
    'resource',
    'resourcesSelected',
    'resourceName',
    'relationshipType',
    'viaRelationship',
    'viaResource',
    'viaResourceId',
    'viaManyToMany',
    'checked',
    'actionsAreAvailable',
    'actionsEndpoint',
    'shouldShowCheckboxes',
    'shouldShowColumnBorders',
    'tableStyle',
    'updateSelectionStatus',
    'queryString',
    'clickAction',
  ],

  data: () => ({
    commandPressed: false,
    deleteModalOpen: false,
    restoreModalOpen: false,
    previewModalOpen: false,
  }),

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

    navigateToDetail(e) {
      if (this.clickAction === 'edit') {
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
        : Inertia.visit(this.viewURL)
    },

    navigateToEditView(e) {
      if (!this.resource.authorizedToUpdate) {
        return
      }
      this.commandPressed
        ? window.open(this.updateURL, '_blank')
        : Inertia.visit(this.updateURL)
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
      return filter(this.resource.actions, a => a.showOnTableRow)
    },

    shouldShowTight() {
      return this.tableStyle === 'tight'
    },

    clickableRow() {
      if (this.clickAction === 'edit') {
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
  },
}
</script>
