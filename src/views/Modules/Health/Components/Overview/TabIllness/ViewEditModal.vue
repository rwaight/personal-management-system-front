<template>
  <div>
    <Modal :is-visible="showModal"
           id="illness-view-edit"
           :title="$t('health.overview.tabs.common.viewEditModal.header')"
           :cancel-text="$t('components.modal.buttons.close')"
           :size="modalSize"
           @modal-closed="onModalClosed"
    >
      <template #content>

        <Tabs :tabs-with-content="tabsContent"
              :no-background="true"
              padding-bottom="0"
              id="illnessesViewEditModal"
              @submit="onTabSubmit"
              @files-saved="$emit('filesSaved')"
        />

      </template>
    </Modal>
  </div>
</template>

<script lang="ts">
import Modal                    from "@/components/Modal/Modal.vue";
import ResponsiveModalSizeMixin from "@/mixins/Responsive/ResponsiveModalSizeMixin.vue";
import Tabs                     from "@/components/Navigation/Tabs/Tabs.vue";
import TabFiles                 from "@/views/Modules/Health/Components/Overview/TabIllness/Components/ViewEditModal/Tabs/TabFiles.vue";
import TabIllness               from "@/views/Modules/Health/Components/Overview/TabIllness/Components/ViewEditModal/Tabs/TabIllness.vue";
import TabAppointments          from "@/views/Modules/Health/Components/Overview/TabIllness/Components/ViewEditModal/Tabs/TabAppointments.vue";

import {ComponentData} from "@/scripts/Vue/Types/Components/types";

import {DoctorAppointmentStore} from "@/scripts/Vue/Store/Module/Health/AppointmentStore";

import BaseError from "@/scripts/Core/Error/BaseError";

export default {
  data(): ComponentData {
    return {
      doctorAppointmentStore: null,
      initialSmallSizeModal: "medium",
      showModal: false,
      usedIllness: null as Record<string, unknown> | null,
    }
  },
  props: {
    illness: {
      type: [Object, null],
      required: false,
      default: null,
    },
    isModalVisible: {
      type     : Boolean,
      required : true,
      default  : false,
    }
  },
  components: {
    Tabs,
    Modal,
  },
  mixins: [
    ResponsiveModalSizeMixin,
  ],
  emits: [
    "modalClosed",
    "filesSaved"
  ],
  computed: {
    /**
     * @description returns the data structure for tabs
     */
    tabsContent(): Array<Record<string, unknown>> {
      let tabs = [];
      if (this?.usedIllness?.id) {
        tabs.push({
          tabName: this.$t('health.overview.tabs.subTabs.tabs.files.header.label'),
          tabComponent: TabFiles,
          tabComponentProps: {
            illness: this.usedIllness,
          }
        })
      }

      tabs.push({
        tabName: this.$t('health.overview.tabs.subTabs.tabs.illness.header.label'),
        tabComponent: TabIllness,
        tabComponentProps: {
          illness: this.usedIllness,
        }
      })

      if (this?.usedIllness?.id) {
        tabs.push({
          tabName: this.$t('health.overview.tabs.subTabs.tabs.appointments.header.label'),
          tabComponent: TabAppointments,
          tabComponentProps: {
            illness: this.usedIllness,
          }
        })
      }

      return tabs;
    }
  },
  methods: {
    /**
     * @description this is needed explicitly for "create" mode. Initially, when the modal is opened, we show only the
     *              "Illness" tab. With this code, when a user submits a new entry, we instantly show other tabs too.
     */
    onTabSubmit(data): void {
      if (!data) {
        this.$emit('modalClosed');
        return;
      }

      if (!data.singleRecord) {
        throw new BaseError("Illness (singleRecord) is missing in the data object");
      }

      this.usedIllness = data.singleRecord;
    },
    /**
     * @description handles the situation when modal get closed. Will pass the event further
     */
    onModalClosed(): void {
      this.$emit('modalClosed');
    },
  },
  created(): void {
    this.doctorAppointmentStore = DoctorAppointmentStore();
    this.doctorAppointmentStore.getAll();
    this.usedIllness = this.illness;
  },
  watch: {
    illness() : void {
      this.usedIllness = this.illness;
    },
    isModalVisible() {
      this.showModal = this.isModalVisible;
    }
  }
}
</script>