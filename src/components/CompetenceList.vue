<template>
  <!-- FULL WIDTH TITLE -->
  <div style="width: 100vw; display: flex; justify-content: center">
    <h1
      style="
        font-size: 2rem;
        font-weight: 800;
        text-align: center;
        color: var(--brand-fourth);
        white-space: nowrap;
        margin: 2rem 0;
      "
    >
      SaxoCell Competences Table
    </h1>
  </div>
  <div class="max-w-screen-xl mx-auto p-6">
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
        <input
          v-model="newItem.email"
          placeholder="Email"
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
    <p class="italic text-gray-700 mb-6">
      If you are interested in contacting any of the organizations for a
      specific competence or if you have any questions please contact us at
      <a href="mailto:info@saxocell.de" class="underline"> info@saxocell.de </a>
    </p>

    <!-- DATA TABLE -->
    <div class="overflow-x-auto">
      <table class="table-fixed w-full text-lg">
        <thead>
          <tr>
            <th
              :colspan="isAdmin ? 9 : 7"
              class="px-6 py-4 text-center font-extrabold text-white !text-white"
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
            <!-- email only visible to admins -->
            <th v-if="isAdmin" class="px-6 py-3">Email</th>
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
            <!-- Organisation -->
            <td class="px-6 py-3">
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.organization"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.organization }}</template>
            </td>

            <!-- Area -->
            <td
              class="px-6 py-3"
              :class="{
                'font-black text-[var(--brand-fourth)]':
                  item.area === 'Leipzig',
              }"
            >
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.area"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.area }}</template>
            </td>

            <!-- Competence -->
            <td class="px-6 py-3">
              <template v-if="editingId === item.id">
                <textarea
                  v-model="editForm.competence"
                  class="border px-2 py-1 w-full"
                ></textarea>
              </template>
              <template v-else v-html="item.competence"></template>
            </td>

            <!-- Service offers -->
            <td class="px-6 py-3">
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.serviceoffers"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.serviceoffers }}</template>
            </td>

            <!-- Institution contact -->
            <td class="px-6 py-3">
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.institutioncontact"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.institutioncontact }}</template>
            </td>

            <!-- Further info -->
            <td class="px-6 py-3">
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.furtherinfo"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.furtherinfo }}</template>
            </td>

            <!-- City -->
            <td class="px-6 py-3">
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.city"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.city }}</template>
            </td>

            <!-- Email (admin only) -->
            <td v-if="isAdmin" class="px-6 py-3">
              <template v-if="editingId === item.id">
                <input
                  v-model="editForm.email"
                  class="border px-2 py-1 w-full"
                />
              </template>
              <template v-else>{{ item.email }}</template>
            </td>

            <!-- Actions (admin only) -->
            <td v-if="isAdmin" class="px-6 py-3">
              <template v-if="editingId === item.id">
                <button
                  @click="saveEdit(item.id)"
                  class="mr-2 bg-blue-600 text-white px-2 py-1 rounded"
                >
                  Save
                </button>
                <button
                  @click="cancelEdit"
                  class="bg-gray-200 px-2 py-1 rounded"
                >
                  Cancel
                </button>
              </template>
              <template v-else>
                <button
                  @click="startEdit(item)"
                  class="mr-2 text-blue-600 underline"
                >
                  Edit
                </button>
                <button
                  @click="deleteItem(item)"
                  class="text-red-600 underline"
                >
                  Delete
                </button>
              </template>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
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

function startEdit(item) {
  editingId.value = item.id;
  // shallow copy the item into editForm
  editForm.value = {
    organization: item.organization ?? "",
    area: item.area ?? "",
    competence: item.competence ?? "",
    serviceoffers: item.serviceoffers ?? "",
    institutioncontact: item.institutioncontact ?? "",
    furtherinfo: item.furtherinfo ?? "",
    city: item.city ?? "",
    email: item.email ?? "",
  };
}

function cancelEdit() {
  editingId.value = null;
  editForm.value = {
    organization: "",
    area: "",
    competence: "",
    serviceoffers: "",
    institutioncontact: "",
    furtherinfo: "",
    city: "",
    email: "",
  };
}

async function saveEdit(id) {
  // send only the fields we want to update
  const { data, error } = await supabase
    .from("partners")
    .update({
      organization: editForm.value.organization,
      area: editForm.value.area,
      competence: editForm.value.competence,
      serviceoffers: editForm.value.serviceoffers,
      institutioncontact: editForm.value.institutioncontact,
      furtherinfo: editForm.value.furtherinfo,
      city: editForm.value.city,
      email: editForm.value.email,
    })
    .eq("id", id)
    .select();

  if (error) {
    console.error("Update failed:", error);
    alert("Update failed: " + error.message);
    return;
  }

  // reload and close editor
  await loadPartners();
  cancelEdit();
}

// editing state
const editingId = ref(null);
const editForm = ref({
  organization: "",
  area: "",
  competence: "",
  serviceoffers: "",
  institutioncontact: "",
  furtherinfo: "",
  city: "",
  email: "",
});

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

const globalSearch = ref("");

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
        norm(item.city) +
        norm(item.email);

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
  email: "",
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
    email: newItem.value.email,
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
</script>

<style scoped>
/* base table */
table {
  border-collapse: collapse;
  border: 2px solid #ccc;
}

th {
  text-align: center;
  border: 1px solid #ccc;
  padding: 0.75rem;
}

td {
  text-align: left;
  border: 1px solid #ccc;
  padding: 0.75rem;
  white-space: pre-line;
}

/* Only the Area column body cells bold */
td:nth-child(2) {
  font-weight: 700 !important;
}

/* Force the top "Associated Partners" header bold */
th[colspan] {
  font-weight: 800 !important;
  color: white !important;
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
/* Ensure the spanning header is always centered regardless of content */
th[colspan] {
  text-align: center !important;
  vertical-align: middle;
}

/* Ensure the title container doesn't have weird padding issues */
.w-full.text-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
