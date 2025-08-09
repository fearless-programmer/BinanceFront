<script setup>
import { useToast } from 'primevue/usetoast';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const toast = useToast();
const loading = ref(false);

// Form data
const formData = ref({
    name: '',
    description: '',
    contact: '',
    email: '',
    address: ''
});

// Form validation
const errors = ref({});

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

async function createBusiness() {
    if (!validateForm()) {
        return;
    }

    loading.value = true;

    try {
        // Simulate API call
        const obj = { ...formData.value };
        const response = await axios.post('/create-business', obj);

        if (response.status == 201) {
            toast.add({
                severity: 'success',
                summary: 'Success',
                detail: 'Business created successfully',
                life: 3000
            });

            // Navigate back to businesses list
            router.push('/businesses');
        } else {
            toast.add({
                severity: 'error',
                summary: 'Error',
                detail: 'Business creation failed',
                life: 3000
            });
        }
    } catch (error) {
        toast.add({
            severity: 'error',
            summary: 'Error',
            detail: 'Failed to create business',
            life: 3000
        });
    } finally {
        loading.value = false;
    }
}

function goBack() {
    router.push('/businesses');
}

function resetForm() {
    formData.value = {
        name: '',
        description: '',
        contact: '',
        email: '',
        address: ''
    };
    errors.value = {};
}
</script>

<template>
    <div class="card">
        <div class="flex items-center justify-between mb-6">
            <div>
                <h1 class="text-2xl font-semibold text-gray-900">Create New Business</h1>
                <p class="text-gray-600 mt-1">Add a new business to your directory</p>
            </div>
            <Button icon="pi pi-arrow-left" label="Back to Businesses" outlined @click="goBack" />
        </div>

        <form @submit.prevent="createBusiness" class="max-w-2xl">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- Business Name -->
                <div class="field md:col-span-2">
                    <label for="businessName" class="block text-sm font-medium mb-2"> Business Name <span class="text-red-500">*</span> </label>
                    <InputText id="businessName" v-model="formData.name" :class="{ 'p-invalid': errors.name }" placeholder="Enter business name" class="w-full" />
                    <small v-if="errors.name" class="p-error">{{ errors.name }}</small>
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
                <div class="field md:col-span-2">
                    <label for="address" class="block text-sm font-medium mb-2"> Address </label>
                    <InputText id="address" v-model="formData.address" placeholder="Enter business address" class="w-full" />
                </div>

                <!-- Description -->
                <div class="field md:col-span-2">
                    <label for="description" class="block text-sm font-medium mb-2"> Description </label>
                    <Textarea id="description" v-model="formData.description" placeholder="Enter business description" rows="4" class="w-full" />
                </div>
            </div>

            <div class="flex justify-end gap-3 mt-8">
                <Button type="button" label="Reset" icon="pi pi-refresh" outlined @click="resetForm" :disabled="loading" />
                <Button type="button" label="Cancel" icon="pi pi-times" severity="secondary" outlined @click="goBack" :disabled="loading" />
                <Button type="submit" label="Create Business" icon="pi pi-check" :loading="loading" />
            </div>
        </form>
    </div>
</template>

<style scoped>
.field {
    margin-bottom: 1rem;
}

.p-error {
    color: #ef4444;
    font-size: 0.875rem;
}

.text-gray-600 {
    color: #4b5563;
}

.text-gray-900 {
    color: #111827;
}

.text-red-500 {
    color: #ef4444;
}
</style>
