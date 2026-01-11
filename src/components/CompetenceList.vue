<template>
  <div class="max-w-screen-xl mx-auto p-6 overflow-x-auto">
    <!-- FILTER ROW -->
    <div class="mb-6 flex items-start space-x-4">
      <!-- AREA SELECT -->
      <div class="flex items-center space-x-4">
        <label for="areaFilter" class="font-medium">Filter by Area:</label>
        <select
          id="areaFilter"
          v-model="selectedArea"
          class="border border-gray-300 rounded px-3 py-1"
        >
          <option value="">All Areas</option>
          <option v-for="area in uniqueAreas" :key="area" :value="area">
            {{ area }}
          </option>
        </select>
      </div>

      <!-- ORGANISATION SELECT -->
      <div class="flex items-center space-x-4">
        <label for="organisationFilter" class="font-medium"
          >Filter by Organisation:</label
        >
        <select
          id="organisationFilter"
          v-model="selectedOrganisation"
          class="border border-gray-300 rounded px-3 py-1"
        >
          <option value="">All Organisations</option>
          <option v-for="org in uniqueOrganisations" :key="org" :value="org">
            {{ org }}
          </option>
        </select>
      </div>

      <!-- COMPETENCE DROPDOWN -->
      <div class="relative" ref="dropdownRoot">
        <button
          @click="toggleCompetenceDropdown"
          :aria-expanded="showCompetenceDropdown"
          aria-haspopup="true"
          aria-label="Open competence filter"
          class="inline-flex items-center justify-center w-8 h-8 rounded-full border border-gray-300 bg-white text-slate-800 p-1 shadow z-50"
        >
          <!-- SVG uses 'fill-current' so it inherits the text color above -->
          <svg
            class="w-4 h-4 fill-current"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 20 20"
            aria-hidden="true"
          >
            <path
              fill-rule="evenodd"
              d="M5.23 7.21a.75.75 0 011.06.02L10 11.584l3.71-4.354a.75.75 0 111.14.98l-4.25 5a.75.75 0 01-1.14 0l-4.25-5a.75.75 0 01.02-1.06z"
              clip-rule="evenodd"
            />
          </svg>
        </button>

        <!-- Dropdown panel -->
        <transition name="fade">
          <div
            v-if="showCompetenceDropdown"
            ref="dropdownPanel"
            class="absolute mt-2 w-80 bg-white border rounded shadow-lg z-50 p-3"
            @click.stop
            role="dialog"
            aria-label="Filter by competence"
          >
            <!-- Quick actions -->
            <div class="flex items-center justify-between mb-2">
              <strong>Filter by Competence</strong>
              <div class="text-xs text-slate-500">
                <button @click="clearCompetenceFilters" class="underline">
                  Clear
                </button>
              </div>
            </div>

            <!-- Predefined keywords -->
            <div
              class="grid grid-cols-1 gap-2 max-h-36 overflow-auto pr-2 mb-3"
            >
              <label
                v-for="kw in competenceKeywords"
                :key="kw"
                class="inline-flex items-center space-x-2"
              >
                <input
                  type="checkbox"
                  :value="kw"
                  v-model="selectedCompetenceKeywords"
                  class="w-4 h-4"
                />
                <span class="text-sm">{{ kw }}</span>
              </label>
            </div>

            <!-- Free text search -->
            <div class="flex items-center space-x-2">
              <input
                v-model="competenceSearch"
                placeholder="Search in competence text..."
                class="flex-1 border px-2 py-1 rounded"
              />
              <button
                @click="applySearch"
                class="px-2 py-1 border rounded text-sm"
              >
                Apply
              </button>
            </div>

            <p class="mt-2 text-xs text-slate-500">
              Tip: use checkboxes for quick phrase filters or type any fragment
              and click Apply.
            </p>
          </div>
        </transition>
      </div>
    </div>

    <!-- ADMIN‐ONLY ADD FORM -->
    <div
      v-if="isAdmin"
      class="mb-8 p-4 border border-green-400 rounded bg-green-50"
    >
      <h3 class="font-semibold mb-2">Add New Partner</h3>
      <div class="grid grid-cols-2 gap-4">
        <input
          v-model="newItem.organization"
          placeholder="Organisation"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.area"
          placeholder="Area"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.competence"
          placeholder="Competence"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.serviceOffers"
          placeholder="Service Offers"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.institutionContact"
          placeholder="Contact"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.furtherInfo"
          placeholder="Further Info"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.city"
          placeholder="City"
          class="border px-2 py-1"
        />
      </div>
      <button
        @click="addItem"
        class="mt-4 bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
      >
        + Add Partner
      </button>
    </div>

    <!-- DATA TABLE -->
    <table class="table-fixed w-full text-lg">
      <thead>
        <tr>
          <th
            colspan="7"
            class="px-6 py-4 text-center font-bold table-header-associated"
          >
            Associated Partners
          </th>
        </tr>

        <tr class="table-header-columns">
          <th class="px-6 py-3">Organisation</th>
          <th class="px-6 py-3">Area</th>
          <th class="px-6 py-3">Competence</th>
          <th class="px-6 py-3">Service Offers</th>
          <th class="px-6 py-3">Institution/Contact</th>
          <th class="px-6 py-3">Further Information</th>
          <th class="px-6 py-3">City</th>
        </tr>
        <tr>
          <th
            colspan="7"
            class="px-6 py-4 text-center font-bold table-header-leipzig"
          >
            Leipzig
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(item, idx) in filteredList"
          :key="idx"
          class="bg-[var(--brand-secondary)]"
        >
          <td class="px-6 py-3">{{ item.organization }}</td>

          <!-- Area: bold for body cells; special extra-bold + color for Leipzig -->
          <td
            class="px-6 py-3"
            :class="{
              'font-semibold': true,
              'font-black text-[var(--brand-fourth)]': item.area === 'Leipzig',
            }"
          >
            {{ item.area }}
          </td>

          <td class="px-6 py-3" v-html="item.competence"></td>

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
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import rawData from "../data/CompetenceList.json";

// props
const props = defineProps({
  isAdmin: { type: Boolean, default: false },
});

// normalize incoming keys
const dataList = ref(
  rawData.map((item) => ({
    organization: item["Organisation"] ?? item.organization ?? "",
    area: item["Area"] ?? item.area ?? "",
    competence: item["Competence"] ?? item.competence ?? "",
    serviceOffers: item["Service Offers"] ?? item.serviceOffers ?? "",
    institutionContact:
      item["Institution/Contact"] ?? item.institutionContact ?? "",
    furtherInfo: item["Further Information"] ?? item.furtherInfo ?? "",
    city: item["City"] ?? item.city ?? "",
  }))
);

// AREA filter
const selectedArea = ref("");
const uniqueAreas = computed(() => {
  const areas = dataList.value.map((r) => r.area || "");
  return [...new Set(areas)].sort();
});

// ORGANISATION filter
const selectedOrganisation = ref("");
const uniqueOrganisations = computed(() => {
  const orgs = dataList.value.map((r) => r.organization || "");
  return [...new Set(orgs)].sort();
});

// --- COMPETENCE FILTER VARIABLES ---
const competenceKeywords = [
  "Development of ATMPs",
  "Target validation",
  "Optimization of CAR cells",
  "Generation and testing of genetically-modified immunocells",
  "Provision of blood cells from healthy donors as control for transfusion medicine",
  "Cell Therapy",
  // extend as needed
];

const selectedCompetenceKeywords = ref([]);
const competenceSearch = ref("");

// Dropdown UI state and refs
const showCompetenceDropdown = ref(false);
const dropdownRoot = ref(null);
const dropdownPanel = ref(null);

function toggleCompetenceDropdown() {
  showCompetenceDropdown.value = !showCompetenceDropdown.value;
}

function closeCompetenceDropdown() {
  showCompetenceDropdown.value = false;
}

function applySearch() {
  closeCompetenceDropdown();
}

function clearCompetenceFilters() {
  selectedCompetenceKeywords.value = [];
  competenceSearch.value = "";
}

// helper for normalized comparisons
function norm(s) {
  return (s || "").toString().toLowerCase();
}

// helper to strip HTML tags
function stripHTML(s) {
  return (s || "").replace(/<[^>]*>/g, "").toLowerCase();
}

// filteredList applies area + competence keywords + free text
const filteredList = computed(() => {
  const area = norm(selectedArea.value);
  const selectedKeys = selectedCompetenceKeywords.value.map(norm);
  const freeSearch = norm(competenceSearch.value).trim();

  return dataList.value.filter((item) => {
    // AREA filter
    if (area && norm(item.area) !== area) return false;

    const compText = stripHTML(item.competence);

    const organisation = norm(selectedOrganisation.value);

    // ORGANISATION filter
    if (organisation && norm(item.organization) !== organisation) return false;

    // predefined keywords: if any selected, require at least one match
    if (selectedKeys.length > 0) {
      const any = selectedKeys.some((kw) => compText.includes(kw));
      if (!any) return false;
    }

    // free-text search
    if (freeSearch && !compText.includes(freeSearch)) return false;

    return true;
  });
});

// Add item form state + add function
const newItem = ref({
  organization: "",
  area: "",
  competence: "",
  serviceOffers: "",
  institutionContact: "",
  furtherInfo: "",
  city: "",
});

function addItem() {
  if (!newItem.value.organization || !newItem.value.area) {
    return alert("Organisation and Area are required.");
  }
  dataList.value.push({ ...newItem.value });
  Object.keys(newItem.value).forEach((k) => (newItem.value[k] = ""));
}

// Close dropdown on outside click or Escape
function onDocumentClick(e) {
  const root = dropdownRoot.value;
  if (!root) return;
  if (!root.contains(e.target)) {
    closeCompetenceDropdown();
  }
}

function onDocumentKey(e) {
  if (e.key === "Escape") closeCompetenceDropdown();
}

onMounted(() => {
  document.addEventListener("click", onDocumentClick);
  document.addEventListener("keydown", onDocumentKey);
});

onBeforeUnmount(() => {
  document.removeEventListener("click", onDocumentClick);
  document.removeEventListener("keydown", onDocumentKey);
});
</script>

<style scoped>
/* base table */
table {
  border-collapse: collapse;
  border: 2px solid #ccc;
}

th,
td {
  border: 1px solid #ccc;
  text-align: center;
  padding: 0.75rem;
  white-space: pre-line; /* show \n as new lines */
}

/* Only the Area column body cells bold */
td:nth-child(2) {
  font-weight: 700 !important;
}

/* Force the top "Associated Partners" header bold */
th[colspan="7"] {
  font-weight: 700 !important;
}

/* Optional class to highlight Leipzig (use class binding in template) */
.area-leipzig {
  font-weight: 900 !important; /* extra bold */
  color: var(--brand-fourth);
}

/* IMPORTANT: target HTML injected by v-html using the deep selector so scoped styles apply */
td :deep(b),
td :deep(strong),
td :deep(.font-bold),
td :deep([style*="font-weight"]) {
  font-weight: 900 !important;
}

/* Simple fade transition for dropdown */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.12s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

td:nth-child(3),
th:nth-child(3) {
  min-width: 300px; /* adjust as needed */
  max-width: 500px; /* optional */
  word-break: break-word; /* wrap long text nicely */
}

/* Zebra striping for table body rows */
tbody tr:nth-child(odd) {
  background-color: #ffffff; /* white */
}

tbody tr:nth-child(even) {
  background-color: #cccccc; /* light gray */
}

/* Ensure text remains readable */
tbody tr td {
  color: #000000;
}
</style>
