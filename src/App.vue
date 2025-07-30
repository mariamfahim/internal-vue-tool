<template>
  <div id="app" class="w-screen">
    <CompetenceList />
  </div>
</template>

<script setup>
import CompetenceList from "./components/CompetenceList.vue";
</script>

<style>
:root {
  --brand-primary: #37ab9c;
  --brand-secondary: #b6dcd3;
  --brand-third: #9085ba;
  --brand-fourth: #108886;
  --brand-fifth: #2e9c8a;
}

html,
body,
#app {
  margin: 0;
  padding: 0;
  width: 100vw;
}
</style>
