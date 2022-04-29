<template>
    <div class="tab-pane" :class="{active: isActive}" v-if="isActive">
        <slot></slot>
    </div>
</template>

<script>
export default {
    name: 'tab-content',
    props:{
        selected: {
            default: false,
        },
        title: {
            type: String,
            required: true
        }
    },
    data(){
        return{
            isActive: false,
            isValidated : false,
        }
    },
    created(){
        this.isActive = this.selected;
    }
}
</script>
