<template>
    <div :class="getClassesFor('wizard_container')">
        <div :class="getClassesFor('wizard_header')">
          <slot name="progress-bar" v-bind:progress="progress" v-if="progressBeforeTabs">
            <div :class="getClassesFor('wizard_progress')">
              <div :class="getClassesFor('progress_bar')" :style="{ width: progress + '%' }"></div>
            </div>
          </slot>
          <ul :class="getClassesFor('tabs')">
            <li @click.prevent.stop="selectTab(index)" :class="getTabClasses(tab)" v-for="(tab, index) in tabs" v-bind:key="`tab-${index}`">
              <slot name="tab-layout" v-bind:tab="tab" v-bind:index="index">
                <a class="step-link" href="#">
                  <span class="tabStatus">{{index+1}} </span>
                  <span class="tabLabel">{{tab.title}}</span>
                </a>
              </slot>
            </li>
          </ul>
        </div>
        <div :class="getClassesFor('wizard_content')">
          <slot name="progress-bar" v-bind:progress="progress" v-if="progressAfterTabs">
            <div :class="getClassesFor('wizard_progress')">
              <div :class="getClassesFor('progress_bar')" :style="{ width: progress + '%' }"></div>
            </div>
          </slot>

          <form>
            <slot></slot>
          </form>
        </div>
        <slot name="footer"
              v-bind:submitSuccess="submitSuccess"
              v-bind:previousEnabled="previousEnabled"
              v-bind:nextEnabled="nextEnabled"
              v-bind:submitEnabled="submitEnabled"
              v-bind:previousTab="previousTab"
              v-bind:nextTab="nextTab"
              v-bind:onSubmit="onSubmit"
              v-bind:reset="reset">
          <div class="step-footer">
            <div class="btn-group" role="group">
              <template v-if="!submitSuccess">
                <button @click="previousTab" :disabled="!previousEnabled" :class="getClassesFor('previous_button')">Previous</button>
                <button @click="nextTab" v-if="nextEnabled" :class="getClassesFor('next_button')">Next</button>
                <button @click="onSubmit" v-if="submitEnabled" :class="getClassesFor('submit_button')">Submit</button>
              </template>
              <template v-else>
                <button @click="reset" :class="getClassesFor('reset_button')">Reset</button>
              </template>
            </div>
          </div>
        </slot>
    </div>
</template>
<script>
import { store } from "./store.js";
export default {
    name: 'form-wizard',
    props: [
      'classes',
      'progressBeforeTabs',
      'progressAfterTabs',
    ],
    data(){
        return{
            tabs: [],
            currentTab : 0,
            totalTabs : 0,
            storeState: store.state,
            submitSuccess : false,
            progress: 0,
            isValidationSupport: false,
            wizardClasses: {
              previous_button: 'step-button step-button-previous',
              next_button: 'step-button step-button-next',
              submit_button: 'step-button step-button-submit',
              reset_button: 'step-button step-button-reset',
              tabs: 'step-pills',
              tab_single: 'step-item',

              // Container
              wizard_container: 'vue-step-wizard',
              wizard_content: 'step-body',
              wizard_header: 'step-header',
              wizard_progress: 'step-progress',
              progress_bar: 'bar progressbar',
            }
        }
    },
    mounted(){
        if (typeof this.classes !== "undefined" && this.classes !== null) {
            this.wizardClasses = this.mergeClassesWith(this.classes);
        }

        this.tabs = this.$children;
        this.totalTabs = this.tabs.length;
        this.currentTab = this.tabs
            .filter(tab => !tab.disabled)
            .findIndex((tab) => tab.isActive === true);

        //Select first tab if none is marked selected
        if(this.currentTab === -1 && this.totalTabs > 0){
            const firstNotDisabled = this.tabs.filter(tab => !tab.disabled).shift();
            firstNotDisabled.isActive = true;

            this.currentTab = this.tabs.indexOf(firstNotDisabled);
        }

        //Setup Initial Progress
        this.progress = ((this.currentTab + 1) / this.totalTabs * 100);
    },

    updated(){
        /*
          Using this lifehook - since store variable gets updated after component is mounted
          isValidationSupport checks if user is looking to perform validation on each step
        */
        this.isValidationSupport = (Object.keys(this.storeState.v).length !== 0 && this.storeState.v.constructor === Object);
    },

    methods:{
        previousTab(){
            const nextTabIndex = this._findNextTab(-1);
            this._switchTab(nextTabIndex);

            this.$emit('onPreviousStep'); 
        },

        nextTab(){

            if(this._validateCurrentTab() === false)
                return;

            const nextTabIndex = this._findNextTab(1);
            this._switchTab(nextTabIndex);

            this.$emit('onNextStep');          
              
        },

        reset(){

           this.tabs.forEach(tab => {
             tab.isActive = false;
             tab.isValidated = false;
           });

           this.currentTab = 0;

           const firstTab = this._findNextTab(1);
           this._switchTab(firstTab);
           this.submitSuccess = false;
           this.storeState.v.$reset();

           this.$emit('onReset');
        },

        changeStatus(){
            this.submitSuccess = true;
        },

        selectTab(index){
            if (this.tabs[index].disabled) {
              return;
            }

            //Only switch to filled previous tabs
            if(index < this.currentTab){
              this._switchTab(index);
              return;
            }

            if(this._validateCurrentTab() === false){
                return;
            }

            if(this.tabs[index - 1].isValidated === false){
                return;
            }

            this._switchTab(index);
            
        },


        onSubmit(){
            if(this._validateCurrentTab() === false)
                return;
            this.$emit('onComplete');
        },

        _findNextTab(direction = 1, offset = 1) {
            const nextTabIndex = this.currentTab + direction * offset;
            if (nextTabIndex < 0 || nextTabIndex >= this.tabs.length) {
              return false;
            }

            const nextTab = this.tabs[nextTabIndex];
            if (nextTab.disabled) {
                return this._findNextTab(direction, offset + 1);
            }
            return nextTabIndex;
        },

        _switchTab(index){
            //Disable all tabs
            this.tabs.forEach(tab => {
              tab.isActive = false;
            });

            this.currentTab = index;
            this.tabs[index].isActive = true;

            this.progress = ((this.currentTab + 1) / this.totalTabs * 100);
        },

        _validateCurrentTab(){
            if(!this.isValidationSupport)  //Check if user wants to validate 
                return true;

            this.storeState.v.$touch();

            if (this.storeState.v.$invalid) {
                this.tabs[this.currentTab].isValidated = false;
                return false;
            }

            this.tabs[this.currentTab].isValidated = true;

            return true;
        },

        mergeClassesWith(classes) {
            const resultClasses = this.wizardClasses;

            for (let key in classes) {
                resultClasses[key] = classes[key];
            }

            return resultClasses;
        },

        getClassesFor(element) {
            return this.wizardClasses[element];
        },

        getClassesWithExtraFor(element, extra) {
          const baseClasses = this.getClassesFor(element).split(' ');
          const classes = {};

          for (const className of baseClasses) {
            classes[className] = true;
          }

          for (const className in extra) {
              classes[className] = extra[className];
          }

          return classes;
        },

        getTabClasses(tab) {
            return this.getClassesWithExtraFor('tab_single', {
              'active': tab.isActive && !tab.disabled,
              'validated': tab.isValidated,
              'disabled': tab.disabled
            });
        }

    },
    watch:{
       currentTab(){
          store.setCurrentTab(this.currentTab);
       }
    },
    computed: {
        previousEnabled() {
          return this._findNextTab(-1) !== false && this.currentTab !== 0;
        },
        nextEnabled() {
            return this.currentTab < this.totalTabs - 1 || this._findNextTab(1) !== false;
        },
        submitEnabled() {
            return this.currentTab === this.totalTabs - 1 || this._findNextTab(1) === false;
        }
    }
    
}
</script>
<style>

  .progressbar {
    transition: width 1s ease;
  } 

  .vue-step-wizard{
    background-color: #F7F8FC;
    margin: auto;
    padding: 40px;
  }

  .step-progress{
    height: 1rem;
    background: white;
    border-radius: 1rem;
    margin: 1rem 0rem;
  }

  .step-progress .bar{
      content: '';
      height: 1rem;
      border-radius: 1rem;
      background-color: #4B8AEB;
  }

  .step-pills{
    display: flex;
    background-color: white;
    justify-content: space-between;
    padding: 1rem;
    border-radius: 1rem;
    box-shadow: 0 .5rem 1rem rgba(0,0,0,.15)!important;
  }

  .step-pills .step-item{
    background-color: #F5F5F5;
    border-radius: 10px;
    padding: 5px 20px;
    list-style-type: none;
    padding: .5rem 1.5rem;
  }


   .step-pills .step-item a{
     text-decoration: none;
     color: #7B7B7B;
   }

   .step-pills .step-item.active{
      border: 1px solid #4B8AEB;
   }

   .step-pills .step-item.validated{
      border: 1px solid #008011;
   }

   .step-body{
     background-color: white;
     margin-left: auto;
     padding: 1rem;
     border-radius: 1rem;
     box-shadow: 0 .5rem 1rem rgba(0,0,0,.15)!important;
   }

  .step-footer{
     margin-left: auto;
     padding: 1rem;
     border-radius: 1rem;
     margin: 1rem 0rem;
     text-align: center;
   }

  .step-button{
    font-weight: 700;
    line-height: 1;
    text-transform: uppercase;
    position: relative;
    max-width: 30rem;
    text-align: center;
    border: 1px solid;
    border-radius: 12px;
    color: #22292f;
    color: rgba(34,41,47,var(--text-opacity));
    padding: 0.5rem 1.25rem;
    font-size: .875rem;
    margin: 0.5rem;
    color: #fff;
    outline: none !important;
    box-shadow: none !important;
  }

  .step-button-next{
    background-color: #126fde;
  }

  .step-button-previous{
    background-color: #3deaba;
  }

  .step-button-submit{
    background-color: #4fa203;
  }

  .step-button-reset{
    background-color: #037da2;
  }

  /** Wizard Ends */
  .tabStatus{
      display: inline-block;
      width: 1.5rem;
      height: 1.5rem;
      margin-right: .5rem;
      line-height: 1.5rem;
      color: #fff;
      text-align: center;
      background: rgba(0,0,0,0.38);
      border-radius: 50%;
  }


</style>
