<template>
  <div id="app">
     <header-component />
     <page-component/>
  </div>
</template>

<script>
import Header from "./components/Header.vue"
import Page from "./components/Page.vue"

export default {

  name: 'App',
  components: {
    "header-component": Header,
    "page-component": Page
  }
}
</script>
