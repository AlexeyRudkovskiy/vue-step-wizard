<template>
  <FormWizard
      :classes="bootstrapClasses"
      :progress-before-tab="false"
      :progress-after-tabs="true"

      @onComplete="onComplete"
      @onNextStep="nextStep"
      @onPreviousStep="previousStep"
      @onReset="reset">
    <template v-slot:tab-layout="tabProps">
      <a class="nav-link rounded-0 pt-2 pb-2" :class="{active: tabProps.tab.isActive}"><span class="d-inline">{{ tabProps.tab.title }}</span></a>
    </template>
    <slot></slot>
    <template v-slot:footer="footer">
      <hr>
      <ul class="list-inline mb-0 wizard">
        <template v-if="!footer.submitSuccess">
          <li class="previous list-inline-item">
            <button type="button" class="btn btn btn-secondary btn-secondary" :disabled="!footer.previousEnabled" @click="footer.previousTab">Previous</button>
          </li>
          <li class="next list-inline-item float-right" v-if="footer.nextEnabled">
            <button type="button" class="btn btn btn-secondary btn-secondary" @click="footer.nextTab"><span>Next</span></button>
          </li>
          <li class="next list-inline-item float-right" v-if="footer.submitEnabled">
            <button type="button" class="btn btn btn-secondary btn-secondary" @click="footer.onSubmit"><span>Submit</span></button>
          </li>
        </template>
        <template v-else>
          <li class="next list-inline-item float-right">
            <button type="button" class="btn btn btn-secondary btn-secondary" @click="footer.reset"><span>Reset</span></button>
          </li>
        </template>
      </ul>
    </template>
  </FormWizard>
</template>

<script>
import FormWizard from "./FormWizard";

export default {
  name: "BootstrapFormWizard",
  components: {
    FormWizard
  },
  data() {
    return {
      bootstrapClasses: {
        previous_button: 'step-button step-button-previous',
        next_button: 'step-button step-button-next',
        submit_button: 'step-button step-button-submit',
        reset_button: 'step-button step-button-reset',

        // Tabs
        tabs: 'nav nav-pills bg-light nav-justified form-wizard-header mb-3',
        tab_single: 'nav-item',

        // Container
        wizard_container: '',
        wizard_content: 'tab-content b-0 mb-0 pt-0',
        wizard_header : 'wizard-header',

        wizard_progress: 'progress mb-3',
        progress_bar: 'bar progress-bar progress-bar-striped progress-bar-animated bg-success',
      }
    }
  },
  methods: {
    onComplete() {
      this.$emit('onComplete', arguments);
    },
    nextStep() {
      this.$emit('nextStep', arguments);
    },
    previousStep() {
      this.$emit('previousStep', arguments);
    },
    reset() {
      this.$emit('reset', arguments);
    }
  }
}
</script>

<style scoped>

</style>
