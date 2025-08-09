<template>
    <div class="card">
        <div class="font-semibold text-xl mb-4">Business Management</div>
        <DataTable :value="businesses" :paginator="true" :rows="10" dataKey="id" :rowHover="true" :loading="loading">
            <template #header>
                <div class="flex justify-between">
                    <div class="font-semibold text-lg">Business Directory</div>
                    <Button type="button" icon="pi pi-plus" label="Add Business" @click="$router.push('/create-business')" severity="primary" />
                </div>
            </template>
            <template #empty> No businesses found. </template>
            <template #loading> Loading business data. Please wait. </template>

            <Column field="name" header="Business Name" style="min-width: 200px">
                <template #body="{ data }">
                    <div class="font-semibold">{{ data.name }}</div>
                    <div class="text-sm text-gray-600">
                        {{ data.description ? (data.description.length > 100 ? data.description.substring(0, 100) + '...' : data.description) : 'No description' }}
                    </div>
                </template>
            </Column>

            <Column field="contact" header="Contact" style="min-width: 150px">
                <template #body="{ data }">
                    <div class="items-center gap-2">
                        <i class="pi pi-phone text-blue-500 mx-2"></i>
                        <span>{{ data.contact }}</span>
                        <small class="block mt-2"><i class="pi pi-envelope text-blue-500 mx-2"></i> {{ data.email }}</small>
                    </div>
                </template>
            </Column>


            <Column field="address" header="Address" style="min-width: 250px">
                <template #body="{ data }">
                    <div class="flex items-center gap-2">
                        <i class="pi pi-map-marker text-red-500"></i>
                        <span>{{ data.address || 'No address provided' }}</span>
                    </div>
                </template>
            </Column>

            <Column header="Actions" style="min-width: 150px">
                <template #body="{ data }">
                    <div class="flex gap-2">
                        <Button icon="pi pi-pencil" severity="info" size="small" outlined @click="editBusiness(data)" v-tooltip="'Edit Business'" />
                        <Button icon="pi pi-trash" severity="danger" size="small" outlined @click="deleteBusiness(data)" v-tooltip="'Delete Business'" />
                    </div>
                </template>
            </Column>
        </DataTable>

        <!-- Edit Business Dialog -->
        <EditBusiness v-if="showEditDialog" :visible="showEditDialog" :business="selectedBusiness" @business-updated="onBusinessUpdated" @dialog-closed="onDialogClosed" />

        <!-- Delete Business Dialog -->
        <DeleteBusiness v-if="showDeleteDialog" :visible="showDeleteDialog" :business="selectedBusiness" @business-deleted="onBusinessDeleted" @dialog-closed="onDialogClosed" />
    </div>
</template>

<script setup>
import { onBeforeMount, ref } from 'vue';
import DeleteBusiness from './DeleteBusiness.vue';
import EditBusiness from './EditBusiness.vue';
import axios from 'axios'

const businesses = ref([]);
const loading = ref(true);
const selectedBusiness = ref(null);
const showEditDialog = ref(false);
const showDeleteDialog = ref(false);

onBeforeMount(() => {
    fetchBusinesses();
});

async function fetchBusinesses(){
    loading.value = true;
    const response = await axios.get('businesses');
    loading.value = false
    businesses.value = response.data.businesses;
}

function editBusiness(business) {
    selectedBusiness.value = { ...business };
    showEditDialog.value = true;
}

function deleteBusiness(business) {
    selectedBusiness.value = business;
    showDeleteDialog.value = true;
}

function onBusinessUpdated(updatedBusiness) {
    showEditDialog.value = false;
    selectedBusiness.value = null;
}

function onBusinessDeleted(deletedBusinessId) {
    showDeleteDialog.value = false;
    selectedBusiness.value = null;
}

function onDialogClosed() {
    showEditDialog.value = false;
    showDeleteDialog.value = false;
    selectedBusiness.value = null;
}
</script>


<style scoped lang="scss">
:deep(.p-datatable-frozen-tbody) {
    font-weight: bold;
}

:deep(.p-datatable-scrollable .p-frozen-column) {
    font-weight: bold;
}
</style>
