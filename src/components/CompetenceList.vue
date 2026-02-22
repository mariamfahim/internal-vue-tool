<template>
  <div class="max-w-screen-xl mx-auto p-6 overflow-x-auto">
    <h1 class="text-3xl font-bold text-center mb-8 text-[var(--brand-fourth)]">
      SaxoCell Competences Table
    </h1>

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
      <!-- GLOBAL SEARCH -->
      <div class="flex items-center space-x-4">
        <label class="font-medium">Search:</label>
        <input
          v-model="globalSearch"
          type="text"
          placeholder="Search in all columns..."
          class="border border-gray-300 rounded px-3 py-1 w-64"
        />
      </div>
    </div>

    <!-- ADMIN LOGIN -->
    <div v-if="!isAdmin" class="mb-6 p-4 border rounded bg-gray-50 max-w-md">
      <h3 class="font-semibold mb-2">Admin Login</h3>

      <div class="space-y-2">
        <input
          v-model="loginUser"
          placeholder="Username"
          class="border px-2 py-1 w-full"
        />

        <input
          v-model="loginPass"
          type="password"
          placeholder="Password"
          class="border px-2 py-1 w-full"
        />

        <button
          @click="login"
          class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        >
          Login
        </button>

        <p v-if="loginError" class="text-red-600 text-sm">
          {{ loginError }}
        </p>
      </div>
    </div>

    <!-- LOGOUT -->
    <div v-if="isAdmin" class="mb-4 flex justify-end">
      <button @click="logout" class="text-sm underline text-red-600">
        Logout admin
      </button>
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
          v-model="newItem.serviceoffers"
          placeholder="Service Offers"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.institutioncontact"
          placeholder="Contact"
          class="border px-2 py-1"
        />
        <input
          v-model="newItem.furtherinfo"
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
            :colspan="isAdmin ? 8 : 7"
            class="px-6 py-4 text-center font-bold text-white"
            style="background-color: #37ab9c"
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
          <th v-if="isAdmin" class="px-6 py-3">Actions</th>
        </tr>
        <tr></tr>
      </thead>
      <tbody>
        <tr
          v-for="(item, idx) in filteredList"
          :key="item.id"
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

          <td class="px-6 py-3">{{ item.serviceoffers }}</td>
          <td class="px-6 py-3">{{ item.institutioncontact }}</td>
          <td class="px-6 py-3">{{ item.furtherinfo }}</td>
          <td class="px-6 py-3">{{ item.city }}</td>

          <td v-if="isAdmin" class="px-6 py-3">
            <button @click="deleteItem(item)" class="text-red-600 underline">
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import rawData from "../data/CompetenceList.json";
import { supabase } from "../lib/supabase";

// --- ADMIN AUTH STATE ---
const isAdmin = ref(false);
const loginUser = ref("");
const loginPass = ref("");
const loginError = ref("");

// ⚠️ Demo credentials (NOT secure)

// Restore login from localStorage
onMounted(async () => {
  testSupabase();
  loadPartners();

  const { data } = await supabase.auth.getSession();
  isAdmin.value = !!data.session;
});

async function login() {
  const { error } = await supabase.auth.signInWithPassword({
    email: loginUser.value,
    password: loginPass.value,
  });

  if (error) {
    loginError.value = error.message;
  } else {
    isAdmin.value = true;
    loginError.value = "";
  }
}

async function logout() {
  await supabase.auth.signOut();
  isAdmin.value = false;
}

async function loadPartners() {
  const { data, error } = await supabase
    .from("partners")
    .select("*")
    .order("organization");

  if (error) {
    console.error("Load error:", error);
  } else {
    dataList.value = data;
    console.log("Loaded from Supabase:", data.length);
  }
}

async function testSupabase() {
  const { data, error } = await supabase.from("partners").select("*").limit(3);

  console.log("Supabase test result:", data);
  console.log("Supabase error:", error);
}

async function deleteItem(item) {
  const { error } = await supabase.from("partners").delete().eq("id", item.id);

  if (error) alert("Delete failed");
  else await loadPartners();
}

// normalize incoming keys
const dataList = ref([]);

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

const globalSearch = ref("");

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
  const organisation = norm(selectedOrganisation.value);
  const search = norm(globalSearch.value).trim();

  return dataList.value.filter((item) => {
    if (area && norm(item.area) !== area) return false;

    if (organisation && norm(item.organization) !== organisation) return false;

    if (search) {
      const combined =
        norm(item.organization) +
        norm(item.area) +
        norm(item.competence) +
        norm(item.serviceoffers) +
        norm(item.institutioncontact) +
        norm(item.furtherinfo) +
        norm(item.city);

      if (!combined.includes(search)) return false;
    }

    return true;
  });
});

// Add item form state + add function
const newItem = ref({
  organization: "",
  area: "",
  competence: "",
  serviceoffers: "",
  institutioncontact: "",
  furtherinfo: "",
  city: "",
});

async function addItem() {
  if (!newItem.value.organization || !newItem.value.area) {
    alert("Organisation and Area are required.");
    return;
  }

  const payload = {
    organization: newItem.value.organization,
    area: newItem.value.area,
    competence: newItem.value.competence,
    serviceoffers: newItem.value.serviceoffers,
    institutioncontact: newItem.value.institutioncontact,
    furtherinfo: newItem.value.furtherinfo,
    city: newItem.value.city,
  };

  const { data, error } = await supabase
    .from("partners")
    .insert([payload])
    .select();

  if (error) {
    console.error(error);
    alert(error.message);
    return;
  }

  await loadPartners();

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

/* Zebra striping */
tbody tr:nth-child(odd) {
  background-color: #ffffff;
}

tbody tr:nth-child(even) {
  background-color: #cccccc;
}

/* Restore visible grid lines on gray rows */
tbody tr:nth-child(even) td {
  border-color: #999999; /* darker than #ccc */
}
</style>
