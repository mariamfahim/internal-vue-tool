<template>
  <div class="p-6 overflow-x-auto full-bleed">
    <h2 class="mb-4 text-2xl font-semibold">
      ✅ Data loaded: {{ dataList.length }} items
    </h2>

    <table class="table-fixed w-full text-lg">
      <thead>
        <!-- Top‐level header w/ full span -->
        <tr>
          <th
            colspan="7"
            class="px-6 py-4 text-center font-bold text-white bg-[var(--brand-fourth)]"
          >
            Associated Partners
          </th>
        </tr>

        <!-- Column headers -->
        <tr class="bg-[var(--brand-fifth)] text-white">
          <th class="px-6 py-3">Organisation</th>
          <th class="px-6 py-3">Area</th>
          <th class="px-6 py-3">Competence</th>
          <th class="px-6 py-3">Service Offers</th>
          <th class="px-6 py-3">Institution/Contact</th>
          <th class="px-6 py-3">Further Information</th>
          <th class="px-6 py-3">City</th>
        </tr>
      </thead>

      <tbody>
        <tr
          v-for="(item, idx) in dataList"
          :key="idx"
          class="bg-[var(--brand-secondary)]"
        >
          <td class="px-6 py-3">{{ item.organization }}</td>
          <td class="px-6 py-3">{{ item.area }}</td>
          <td class="px-6 py-3">{{ item.competence }}</td>
          <td class="px-6 py-3">{{ item.serviceOffers }}</td>
          <td class="px-6 py-3">{{ item.institutionContact }}</td>
          <td class="px-6 py-3">{{ item.furtherInfo }}</td>
          <td class="px-6 py-3">{{ item.city }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import seedData from '../data/CompetenceList.json'

const dataList = ref([...seedData])
</script>

<style scoped>
.full-bleed {
  position: relative;
  width: 100vw;
  left: 50%;
  margin-left: -50vw;
}

table {
  border-collapse: collapse;
  border: 2px solid #ccc;
}

th,
td {
  border: 1px solid #ccc;
  text-align: center;
  padding: 0.75rem;
}
</style>
