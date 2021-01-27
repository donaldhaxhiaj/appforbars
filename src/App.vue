<template>
  <div id="app">
    <b-navbar type="dark" variant="info">
      <b-navbar-brand>Bar</b-navbar-brand>
    </b-navbar>
      <b-container fluid>
          <home></home>
      </b-container>
    </div>
</template>

<script>

import Home from '@/components/Home'

export default {
  name: 'app',
  components: {
    Home
  }

}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Lato:400,700');

body {
  background: #EEF1F4 !important;
  font-family: 'Lato', sans-serif !important;
}

.nav-background {
  background: #353535;
}
</style>
