<template>
  <div class="container py-5" id="app">
<!--    <SimpleStepper></SimpleStepper>-->
    <BootstrapStepper></BootstrapStepper>
  </div>
</template>

<script>
import SimpleStepper from './demos/SimpleStepper.vue'
import BootstrapStepper from './demos/BootstrapStepper'

export default {
  name: 'app',
  components: {
    SimpleStepper, BootstrapStepper
  }
}
</script>

<style>

</style>
