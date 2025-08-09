<template>
    <div class="card">
        <div class="flex justify-between items-center mb-4">
            <div>
                <h1 class="text-3xl font-bold text-surface-900 dark:text-surface-0 mb-2">Deposit Management</h1>
                <p class="text-surface-600 dark:text-surface-400">Submit new deposits and track your deposit history</p>
            </div>
        </div>

        <!-- Deposit History Table -->
        <DataTable :value="deposits" :paginator="true" :rows="10" dataKey="id" :rowHover="true" v-model:filters="filters" :loading="loading" :globalFilterFields="['transaction_id', 'payment_method', 'status', 'amount']">
            <template #header>
                <div class="flex justify-between mb-2">
                    <IconField>
                        <InputIcon>
                            <i class="pi pi-search" />
                        </InputIcon>
                        <InputText v-model="filters['global'].value" placeholder="Search through deposits..." />
                    </IconField>
                    <Button label="New Deposit" icon="pi pi-plus" @click="openDepositDialog" class="px-6 py-2" />
                </div>
            </template>
            <template #empty> No deposits found. </template>
            <template #loading> Loading deposit data. Please wait. </template>

            <Column field="transaction_id" header="Transaction ID" style="min-width: 14rem">
                <template #body="{ data }">
                    <span>{{ data.transaction_id }}</span>
                </template>
            </Column>

            <Column field="amount" header="Amount" dataType="numeric" style="min-width: 10rem">
                <template #body="{ data }">
                    <span class="font-semibold">${{ data.amount }}</span>
                </template>
            </Column>

            <Column field="payment_method" header="Payment Method" style="min-width: 12rem">
                <template #body="{ data }">
                    <div class="flex items-center gap-2">
                        <i
                            class="pi"
                            :class="{
                                'pi-university': data.payment_method.includes('Bank') || data.payment_method.includes('Wire'),
                                'pi-credit-card': data.payment_method.includes('Credit Card'),
                                'pi-paypal': data.payment_method.includes('PayPal'),
                                'pi-bitcoin': data.payment_method.includes('Crypto')
                            }"
                        ></i>
                        <span>{{ data.payment_method }}</span>
                    </div>
                </template>
            </Column>

            <Column field="status" header="Status" style="min-width: 10rem">
                <template #body="{ data }">
                    <Tag :value="data.status" severity="success" :icon="data.status === 'approved' ? 'pi pi-check' : data.status === 'rejected' ? 'pi pi-times' : 'pi pi-clock'" />
                </template>
            </Column>

            <Column field="created_at" header="Date" dataType="date" style="min-width: 10rem">
                <template #body="{ data }">
                    {{ data.created_at }}
                </template>
            </Column>

            <Column header="Actions" style="min-width: 8rem">
                <template #body="{ data }">
                    <div class="flex gap-2">
                        <Button icon="pi pi-eye" size="small" text rounded severity="info" v-tooltip="'View Details'" />
                        <Button icon="pi pi-download" size="small" text rounded severity="secondary" v-tooltip="'Download Receipt'" v-if="data.status === 'approved'" />
                    </div>
                </template>
            </Column>
        </DataTable>

        <!-- Deposit Dialog -->
        <Dialog header="Submit New Deposit" v-model:visible="showDepositDialog" :breakpoints="{ '960px': '75vw' }" :style="{ width: '450px' }" :modal="true" class="p-fluid">
            <div class="flex flex-col gap-6">
                <div class="field">
                    <label for="transaction_id" class="font-semibold">Transaction ID *</label>
                    <InputText id="transaction_id" v-model="depositForm.transaction_id" placeholder="Enter transaction ID" :class="{ 'p-invalid': !depositForm.transaction_id && showDepositDialog }" />
                    <small class="text-surface-500">Unique identifier from your payment provider</small>
                </div>

                <div class="field">
                    <label for="amount" class="font-semibold">Amount *</label>
                    <InputNumber id="amount" v-model="depositForm.amount" mode="currency" currency="USD" locale="en-US" placeholder="0.00" :class="{ 'p-invalid': !depositForm.amount && showDepositDialog }" :min="1" :max="50000" />
                    <small class="text-surface-500">Minimum: $1.00, Maximum: $50,000.00</small>
                </div>

                <div class="field">
                    <label for="payment_method" class="font-semibold">Payment Method *</label>
                    <Select
                        id="payment_method"
                        v-model="depositForm.payment_method"
                        :options="paymentMethods"
                        optionLabel="label"
                        optionValue="value"
                        placeholder="Select payment method"
                        :class="{ 'p-invalid': !depositForm.payment_method && showDepositDialog }"
                    >
                        <template #option="slotProps">
                            <div class="flex items-center gap-2">
                                <i
                                    class="pi"
                                    :class="{
                                        'pi-university': slotProps.option.value.includes('bank') || slotProps.option.value.includes('wire'),
                                        'pi-credit-card': slotProps.option.value.includes('credit'),
                                        'pi-paypal': slotProps.option.value.includes('paypal'),
                                        'pi-bitcoin': slotProps.option.value.includes('crypto')
                                    }"
                                ></i>
                                <span>{{ slotProps.option.label }}</span>
                            </div>
                        </template>
                    </Select>
                </div>

                <div class="bg-blue-50 dark:bg-blue-900/20 p-4 rounded-lg border border-blue-200 dark:border-blue-800">
                    <div class="flex items-start gap-3">
                        <i class="pi pi-info-circle text-blue-600 dark:text-blue-400 mt-1"></i>
                        <div class="text-sm">
                            <p class="font-medium text-blue-900 dark:text-blue-100 mb-2">Important Notes:</p>
                            <ul class="text-blue-800 dark:text-blue-200 space-y-1">
                                <li>• All deposits are subject to review and approval</li>
                                <li>• Processing time may take 1-3 business days</li>
                                <li>• Ensure your transaction ID is correct</li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>

            <template #footer>
                <div class="flex justify-end gap-3">
                    <Button label="Cancel" severity="secondary" @click="closeDepositDialog" />
                    <Button label="Submit Deposit" @click="submitDeposit" />
                </div>
            </template>
        </Dialog>
    </div>
</template>

<script setup>
import { FilterMatchMode, FilterOperator } from '@primevue/core/api';
import { useToast } from 'primevue/usetoast';
import { onBeforeMount, ref } from 'vue';

const toast = useToast();

// Dialog state
const showDepositDialog = ref(false);

// Form data
const depositForm = ref({
    transaction_id: '',
    amount: null,
    payment_method: null
});

// Payment methods
const paymentMethods = ref([
    { label: 'Bank Transfer', value: 'bank_transfer' },
    { label: 'Credit Card', value: 'credit_card' },
    { label: 'PayPal', value: 'paypal' },
    { label: 'Crypto', value: 'crypto' },
    { label: 'Wire Transfer', value: 'wire_transfer' }
]);

// Deposit history data
const deposits = ref([]);
const loading = ref(false);
const filters = ref(null);

// Sample deposit data (replace with actual API call)
const sampleDeposits = [
    {
        id: 1,
        transaction_id: 'TXN001234567',
        amount: 1500.0,
        payment_method: 'Bank Transfer',
        status: 'approved',
        date: new Date('2025-08-08'),
        created_at: new Date('2025-08-08T10:30:00')
    },
    {
        id: 2,
        transaction_id: 'TXN001234568',
        amount: 750.5,
        payment_method: 'Credit Card',
        status: 'pending',
        date: new Date('2025-08-09'),
        created_at: new Date('2025-08-09T14:15:00')
    },
    {
        id: 3,
        transaction_id: 'TXN001234569',
        amount: 2000.0,
        payment_method: 'Crypto',
        status: 'rejected',
        date: new Date('2025-08-07'),
        created_at: new Date('2025-08-07T16:45:00')
    },
    {
        id: 4,
        transaction_id: 'TXN001234570',
        amount: 500.0,
        payment_method: 'PayPal',
        status: 'approved',
        date: new Date('2025-08-06'),
        created_at: new Date('2025-08-06T09:20:00')
    },
    {
        id: 5,
        transaction_id: 'TXN001234571',
        amount: 3000.0,
        payment_method: 'Wire Transfer',
        status: 'pending',
        date: new Date('2025-08-05'),
        created_at: new Date('2025-08-05T11:30:00')
    }
];

onBeforeMount(() => {
    loadDeposits();
    initFilters();
});

function loadDeposits() {
    loading.value = true;
    // Simulate API call
    setTimeout(() => {
        deposits.value = sampleDeposits;
        loading.value = false;
    }, 1000);
}

function initFilters() {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        transaction_id: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        payment_method: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
        status: { operator: FilterOperator.OR, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] },
        amount: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] },
        date: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.DATE_IS }] }
    };
}

function clearFilter() {
    initFilters();
}

function openDepositDialog() {
    depositForm.value = {
        transaction_id: '',
        amount: null,
        payment_method: null
    };
    showDepositDialog.value = true;
}

function closeDepositDialog() {
    showDepositDialog.value = false;
}

function submitDeposit() {
    // Validate form
    if (!depositForm.value.transaction_id || !depositForm.value.amount || !depositForm.value.payment_method) {
        toast.add({
            severity: 'error',
            summary: 'Validation Error',
            detail: 'Please fill in all required fields',
            life: 3000
        });
        return;
    }

    // Create new deposit entry
    const newDeposit = {
        id: deposits.value.length + 1,
        transaction_id: depositForm.value.transaction_id,
        amount: depositForm.value.amount,
        payment_method: getPaymentMethodLabel(depositForm.value.payment_method),
        status: 'pending',
        date: new Date(),
        created_at: new Date()
    };

    // Add to deposits array (in real app, this would be an API call)
    deposits.value.unshift(newDeposit);

    toast.add({
        severity: 'success',
        summary: 'Deposit Submitted',
        detail: 'Your deposit has been submitted for review',
        life: 3000
    });

    closeDepositDialog();
}

function getPaymentMethodLabel(value) {
    const method = paymentMethods.value.find((pm) => pm.value === value);
    return method ? method.label : value;
}

function formatDate(value) {
    return value.toLocaleDateString('en-US', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric'
    });
}

function formatDateTime(value) {
    return value.toLocaleString('en-US', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
    });
}

// Available statuses for filtering
const statuses = ['approved', 'pending', 'rejected'];
</script>

<style scoped lang="scss">
:deep(.p-datatable-frozen-tbody) {
    font-weight: bold;
}

:deep(.p-datatable-scrollable .p-frozen-column) {
    font-weight: bold;
}

.field {
    margin-bottom: 1rem;
}

.field label {
    display: block;
    margin-bottom: 0.5rem;
}
</style>
