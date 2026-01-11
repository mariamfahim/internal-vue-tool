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

th.table-header-associated {
  background-color: #37ab9c !important;
  color: white;
}

th.table-header-leipzig {
  background-color: var(--brand-third) !important;
  color: white;
}

/* Header row for table columns */
tr.table-header-columns {
  background-color: var(--brand-primary); /* #37AB9C */
  color: white;
  position: sticky;
  top: 0;
  z-index: 50; /* make sure it stays above table rows */
}

html,
body,
#app {
  margin: 0;
  padding: 0;
  width: 100vw;
}
</style>
