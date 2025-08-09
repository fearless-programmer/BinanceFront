<script setup>
import { useToast } from 'primevue/usetoast';
import { defineEmits, defineProps, ref, watch } from 'vue';

const props = defineProps({
    visible: {
        type: Boolean,
        default: false
    },
    business: {
        type: Object,
        default: () => ({})
    }
});

const emit = defineEmits(['business-deleted', 'dialog-closed']);

const toast = useToast();
const localVisible = ref(props.visible);
const loading = ref(false);

// Watch for prop changes
watch(
    () => props.visible,
    (newVal) => {
        localVisible.value = newVal;
    }
);

async function deleteBusiness() {
    loading.value = true;

    try {
        // Simulate API call
        await new Promise((resolve) => setTimeout(resolve, 1000));

        // Emit the deleted business ID
        emit('business-deleted', props.business.id);

        toast.add({
            severity: 'success',
            summary: 'Success',
            detail: 'Business deleted successfully',
            life: 3000
        });

        closeDialog();
    } catch (error) {
        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: 'Failed to delete business',
            life: 3000
        });
    } finally {
        loading.value = false;
    }
}

function closeDialog() {
    localVisible.value = false;
    emit('dialog-closed');
}

function onHide() {
    emit('dialog-closed');
}
</script>

<template>
    <Dialog v-model:visible="localVisible" :modal="true" header="Delete Business" :style="{ width: '450px' }" :breakpoints="{ '960px': '75vw', '640px': '90vw' }" @hide="onHide">
        <div class="flex items-center justify-center mb-6">
            <i class="pi pi-exclamation-triangle text-red-500 text-4xl mr-4"></i>
            <div>
                <h3 class="text-lg font-semibold mb-2">Confirm Deletion</h3>
                <p class="text-gray-600">Are you sure you want to delete this business? This action cannot be undone.</p>
            </div>
        </div>

        <div class="bg-gray-50 p-4 rounded-lg mb-4" v-if="business">
            <div class="space-y-2">
                <div class="flex justify-between">
                    <span class="font-medium text-gray-700">Business Name:</span>
                    <span class="text-gray-900">{{ business.name }}</span>
                </div>
                <div class="flex justify-between">
                    <span class="font-medium text-gray-700">Email:</span>
                    <span class="text-gray-900">{{ business.email }}</span>
                </div>
                <div class="flex justify-between">
                    <span class="font-medium text-gray-700">Contact:</span>
                    <span class="text-gray-900">{{ business.contact }}</span>
                </div>
            </div>
        </div>

        <div class="flex items-center gap-2 p-4 bg-red-50 border border-red-200 rounded-lg">
            <i class="pi pi-info-circle text-red-600"></i>
            <span class="text-red-700 text-sm"> This will permanently delete the business and all associated data. </span>
        </div>

        <template #footer>
            <div class="flex justify-end gap-2">
                <Button label="Cancel" icon="pi pi-times" outlined severity="secondary" @click="closeDialog" :disabled="loading" />
                <Button label="Delete" icon="pi pi-trash" severity="danger" @click="deleteBusiness" :loading="loading" />
            </div>
        </template>
    </Dialog>
</template>

<style scoped>
.bg-gray-50 {
    background-color: #f9fafb;
}

.bg-red-50 {
    background-color: #fef2f2;
}

.border-red-200 {
    border-color: #fecaca;
}

.text-red-600 {
    color: #dc2626;
}

.text-red-700 {
    color: #b91c1c;
}

.text-gray-600 {
    color: #4b5563;
}

.text-gray-700 {
    color: #374151;
}

.text-gray-900 {
    color: #111827;
}
</style>
