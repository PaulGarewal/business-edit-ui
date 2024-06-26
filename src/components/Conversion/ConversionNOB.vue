<template>
  <div
    id="nature-of-business"
    class="section-container"
    :class="{'invalid-section': invalidSection}"
  >
    <v-row no-gutters>
      <v-col
        cols="12"
        sm="3"
        class="pr-4"
      >
        <label :class="{'error-text': invalidSection}">Nature of Business</label>
        <v-chip
          v-if="hasNaicsChanged"
          id="changed-chip"
          x-small
          label
          color="primary"
          text-color="white"
        >
          <span>{{ getEditedLabel }}</span>
        </v-chip>
      </v-col>

      <v-col
        cols="12"
        sm="9"
      >
        <!-- Edit mode -->
        <template v-if="onEditMode">
          <p class="ma-0">
            Provide a brief description of the nature of business (e.g., corner grocery store,
            automotive repair service, landscaping, etc.).
          </p>
          <v-form
            ref="form"
            lazy-validation
          >
            <v-textarea
              v-model="naicsText"
              filled
              auto-grow
              class="mt-5"
              autocomplete="chrome-off"
              placeholder="Enter Nature of Business"
              rows="3"
              counter="300"
              :rules="naicsRules"
              validate-on-blur
            />
            <div class="float-right mb-2">
              <v-btn
                id="nob-done-btn"
                large
                color="primary"
                class="mr-2"
                @click="onDoneClicked()"
              >
                <span>Done</span>
              </v-btn>
              <v-btn
                id="nob-cancel-btn"
                large
                outlined
                color="primary"
                @click="onCancelClicked()"
              >
                <span>Cancel</span>
              </v-btn>
            </div>
          </v-form>
        </template>

        <!-- Display mode -->
        <div
          v-if="!onEditMode"
          class="d-flex justify-space-between align-start"
        >
          <span id="naics-summary">{{ naicsSummary }}</span>

          <div
            v-if="!hasNaicsChanged"
            class="my-n2 mr-n3"
          >
            <v-btn
              id="nob-change-btn"
              text
              color="primary"
              @click="onChangeClicked()"
            >
              <v-icon small>
                mdi-pencil
              </v-icon>
              <span>{{ getEditLabel }}</span>
            </v-btn>
          </div>

          <div
            v-else
            id="nob-more-actions"
            class="my-n2 mr-n3"
          >
            <v-btn
              id="nob-undo-btn"
              text
              color="primary"
              @click="onUndoClicked()"
            >
              <v-icon small>
                mdi-undo
              </v-icon>
              <span>Undo</span>
            </v-btn>
            <v-menu
              v-model="dropdown"
              offset-y
              left
              nudge-bottom="4"
            >
              <template #activator="{ on }">
                <v-btn
                  id="nob-menu-btn"
                  text
                  small
                  color="primary"
                  v-on="on"
                >
                  <v-icon>{{ dropdown ? 'mdi-menu-up' : 'mdi-menu-down' }}</v-icon>
                </v-btn>
              </template>
              <v-btn
                id="more-changes-btn"
                text
                color="primary"
                class="py-5"
                @click="onChangeClicked(); dropdown = false"
              >
                <v-icon
                  small
                  color="primary"
                >
                  mdi-pencil
                </v-icon>
                <span>Change</span>
              </v-btn>
            </v-menu>
          </div>
        </div>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Mixins, Watch } from 'vue-property-decorator'
import { Action, Getter } from 'pinia-class'
import { CommonMixin } from '@/mixins/'
import { ActionKvIF, FlagsCompanyInfoIF } from '@/interfaces/'
import { NaicsIF } from '@bcrs-shared-components/interfaces/'
import { isEqual } from 'lodash'
import { useStore } from '@/store/store'

@Component({})
export default class ConversionNOB extends Mixins(CommonMixin) {
  @Getter(useStore) getComponentValidate!: boolean
  @Getter(useStore) getCurrentNaics!: NaicsIF
  @Getter(useStore) getEditLabel!: string
  @Getter(useStore) getEditedLabel!: string
  @Getter(useStore) getFlagsCompanyInfo!: FlagsCompanyInfoIF
  @Getter(useStore) getSnapshotNaics!: NaicsIF
  @Getter(useStore) hasNaicsChanged!: boolean

  @Action(useStore) setNaics!: (x: NaicsIF) => void
  @Action(useStore) setValidComponent!: (x: ActionKvIF) => void

  // local variables
  dropdown = false // v-model for dropdown menu
  onEditMode = false
  naicsText = ''

  readonly naicsRules = [
    (v: string) => !!v || 'Nature of Business is required',
    (v: string) => (v?.length <= 300) || 'Maximum 300 characters reached'
  ]

  /** The section validity state (when prompted by app). */
  get invalidSection (): boolean {
    return (this.getComponentValidate && !this.getFlagsCompanyInfo.isValidNatureOfBusiness)
  }

  /** The NAICS code, description or (Not entered). */
  get naicsSummary (): string {
    const code = this.getCurrentNaics.naicsCode
    const desc = this.getCurrentNaics.naicsDescription

    if (code && desc) {
      this.naicsText = this.hasNaicsChanged ? this.naicsText : `${code} - ${desc}`
      return `${code} - ${desc}`
    } else if (desc) {
      this.naicsText = desc
      return desc
    } else {
      return '(Not entered)'
    }
  }

  /** Called when user has clicked the Change button. */
  onChangeClicked (): void {
    this.onEditMode = true
  }

  /** Called when user has clicked the Done button. */
  onDoneClicked (): void {
    // eslint-disable-next-line no-undef
    let validForm = (this.$refs.form as Vue & { validate: () => boolean }).validate()
    if (validForm) {
      if (!isEqual(this.naicsText, this.naicsSummary)) {
        this.setNaics({
          naicsCode: '',
          naicsDescription: this.naicsText
        })
      }
      this.onEditMode = false
    }
  }

  /** Called when user has clicked the Cancel button. */
  onCancelClicked (): void {
    this.onEditMode = false
  }

  /** Called when user has clicked the Undo button. */
  onUndoClicked (): void {
    const code = this.getSnapshotNaics.naicsCode
    const desc = this.getSnapshotNaics.naicsDescription
    this.naicsText = null
    if (code && desc) {
      this.naicsText = this.hasNaicsChanged ? this.naicsText : `${code} - ${desc}`
    } else if (desc) {
      this.naicsText = desc
    } else {
      this.naicsText = ''
    }
    this.setNaics(this.getSnapshotNaics)
  }

  /** Called when this edit mode has changed. */
  @Watch('onEditMode', { immediate: true })
  private onIsEditModeChanged (): void {
    this.updateValidity()
  }

  /** Called when NAICS Description has changed. */
  @Watch('getCurrentNaics.naicsDescription', { immediate: true })
  private onNaicsDescriptionChanged (): void {
    this.updateValidity()
  }

  private updateValidity (): void {
    const isValid = (!this.onEditMode && !!this.getCurrentNaics.naicsDescription)
    this.setValidComponent({ key: 'isValidNatureOfBusiness', value: isValid })
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';

// prevent buttons from wrapping
#nob-more-actions {
  min-width: 140px;
}
</style>
