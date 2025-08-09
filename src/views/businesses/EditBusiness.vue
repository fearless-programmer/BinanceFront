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

const emit = defineEmits(['business-updated', 'dialog-closed']);

const toast = useToast();
const localVisible = ref(props.visible);
const loading = ref(false);

// Form data
const formData = ref({
    id: null,
    name: '',
    description: '',
    contact: '',
    email: '',
    address: ''
});

// Form validation
const errors = ref({});

// Watch for prop changes
watch(
    () => props.visible,
    (newVal) => {
        localVisible.value = newVal;
        if (newVal && props.business) {
            resetForm();
        }
    }
);

watch(
    () => props.business,
    (newBusiness) => {
        if (newBusiness) {
            formData.value = { ...newBusiness };
        }
    },
    { deep: true }
);

function resetForm() {
    if (props.business) {
        formData.value = { ...props.business };
    }
    errors.value = {};
}

function validateForm() {
    errors.value = {};

    if (!formData.value.name.trim()) {
        errors.value.name = 'Business name is required';
    }

    if (!formData.value.contact.trim()) {
        errors.value.contact = 'Contact is required';
    }

    if (!formData.value.email.trim()) {
        errors.value.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.value.email)) {
        errors.value.email = 'Please enter a valid email address';
    }

    return Object.keys(errors.value).length === 0;
}

async function saveBusiness() {
    if (!validateForm()) {
        return;
    }

    loading.value = true;

    try {
        // Simulate API call
        await new Promise((resolve) => setTimeout(resolve, 1000));

        // Emit the updated business
        emit('business-updated', { ...formData.value });

        toast.add({
            severity: 'success',
            summary: 'Success',
            detail: 'Business updated successfully',
            life: 3000
        });

        closeDialog();
    } catch (error) {
        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: 'Failed to update business',
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
    <Dialog v-model:visible="localVisible" :modal="true" header="Edit Business" :style="{ width: '600px' }" :breakpoints="{ '960px': '75vw', '640px': '90vw' }" @hide="onHide">
        <form @submit.prevent="saveBusiness" class="space-y-4">
            <!-- Business Name -->
            <div class="field">
                <label for="businessName" class="block text-sm font-medium mb-2"> Business Name <span class="text-red-500">*</span> </label>
                <InputText id="businessName" v-model="formData.name" :class="{ 'p-invalid': errors.name }" placeholder="Enter business name" class="w-full" />
                <small v-if="errors.name" class="p-error">{{ errors.name }}</small>
            </div>

            <!-- Description -->
            <div class="field">
                <label for="description" class="block text-sm font-medium mb-2"> Description </label>
                <Textarea id="description" v-model="formData.description" placeholder="Enter business description" rows="3" class="w-full" />
            </div>

            <!-- Contact -->
            <div class="field">
                <label for="contact" class="block text-sm font-medium mb-2"> Contact <span class="text-red-500">*</span> </label>
                <InputText id="contact" v-model="formData.contact" :class="{ 'p-invalid': errors.contact }" placeholder="Enter contact number" class="w-full" />
                <small v-if="errors.contact" class="p-error">{{ errors.contact }}</small>
            </div>

            <!-- Email -->
            <div class="field">
                <label for="email" class="block text-sm font-medium mb-2"> Email <span class="text-red-500">*</span> </label>
                <InputText id="email" v-model="formData.email" :class="{ 'p-invalid': errors.email }" placeholder="Enter email address" type="email" class="w-full" />
                <small v-if="errors.email" class="p-error">{{ errors.email }}</small>
            </div>

            <!-- Address -->
            <div class="field">
                <label for="address" class="block text-sm font-medium mb-2"> Address </label>
                <InputText id="address" v-model="formData.address" placeholder="Enter business address" class="w-full" />
            </div>
        </form>

        <template #footer>
            <div class="flex justify-end gap-2">
                <Button label="Cancel" icon="pi pi-times" outlined @click="closeDialog" :disabled="loading" />
                <Button label="Save" icon="pi pi-check" @click="saveBusiness" :loading="loading" />
            </div>
        </template>
    </Dialog>
</template>

<style scoped>
.field {
    margin-bottom: 1rem;
}

.p-error {
    color: #ef4444;
    font-size: 0.875rem;
}
</style>
