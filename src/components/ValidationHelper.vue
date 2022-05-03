<template>
    <div>
    </div>
</template>

<script>
import { validationMixin } from 'vuelidate';
import { store } from "./store.js";
export default {
    name: 'ValidationHelper',
    mixins: [validationMixin],
    data(){
        return{
            storeState: store.state,
        }
    },
    mounted(){
        store.setValidation(this.$v);
    },
    computed:{
        rules() {
            if(this.validationRules)
                return this.validationRules[this.storeState.currentTab] ? this.validationRules[this.storeState.currentTab] : {}
            else
                return {};
        },
    },
    methods:{
        hasError(...path){
          const { $formName = 'formData' } = this._data;

          const lastSegment = path.pop();
          let rules = null;

          try {
            rules = path.reduce((acc, field) => {
              if (!(field in acc)) {
                throw "not-exist";
              }
              return acc[field];
            }, this.$v[$formName]);
          } catch (err) {
            if (err === 'not-exist') {
              return false;
            }
          }

          return (lastSegment in rules) && (rules[lastSegment].$error)
        }
    },

    validations() {
        const { $formName = 'formData' } = this._data;
        return {
            [$formName]: this.rules
        };
    }

}
</script>
