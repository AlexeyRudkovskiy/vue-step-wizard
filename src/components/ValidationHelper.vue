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
        hasError(fieldName){
          const { $formName = 'formData' } = this._data;
            return (fieldName in this.$v[$formName]) && (this.$v[$formName][fieldName].$error)
        }
    },

    validations() {
        const { $formName = 'formData' } = this._data;
        console.log($formName);
        return {
            [$formName]: this.rules
        };
    }

}
</script>
