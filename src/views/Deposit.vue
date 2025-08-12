<template>
    <div class="card">
        <div class="flex justify-between items-center mb-4">
            <div>
                <h1 class="text-3xl font-bold text-surface-900 dark:text-surface-0 mb-2">Deposit Management</h1>
                <p class="text-surface-600 dark:text-surface-400">Submit new deposits and track your deposit history</p>
            </div>
        </div>

        <div v-if="loading" class="flex flex-col items-center justify-center py-12 mb-6">
            <ProgressSpinner style="width: 50px; height: 50px" strokeWidth="4" />
            <p class="text-surface-600 dark:text-surface-400 mt-4 text-sm">Loading deposit data...</p>
        </div>

        <!-- Deposit History Table -->
        <DataTable v-else :value="deposits" :paginator="true" :rows="10" dataKey="id" :rowHover="true" v-model:filters="filters" :globalFilterFields="['transaction_id', 'payment_method', 'status', 'amount']">
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
            <template #empty>
                <div class="flex justify-center">
                    <p>No deposits found.</p>
                </div>
            </template>

            <Column field="transaction_id" header="Transaction ID" style="min-width: 14rem">
                <template #body="{ data }">
                    <span>{{ data.transaction_id }}</span>
                </template>
            </Column>

            <Column field="amount" header="Amount" dataType="numeric" style="min-width: 10rem">
                <template #body="{ data }">
                    <span class="font-semibold">${{ data.amount }}</span>
                    <small class="block font-bold" :class="{ 'text-primary': data.payment_method == 1, 'text-red-400': data.payment_method == 2, 'text-blue-600': data.payment_method == 3 }">
                        {{ data.payment_method == 1 ? 'Bank' : data.payment_method == 2 ? 'Mobile Money' : data.payment_method == 3 ? 'Crypto' : '' }}
                    </small>
                </template>
            </Column>

            <Column field="status" header="Status" style="min-width: 10rem">
                <template #body="{ data }">
                    <Tag
                        :value="data.status == 1 ? 'Pending' : data.status == 2 ? 'Approved' : 'Rejected'"
                        :severity="data.status === 1 ? 'warning' : data.status === 2 ? 'success' : 'danger'"
                        :icon="data.status === 1 ? 'pi pi-clock' : data.status === 2 ? 'pi pi-check' : 'pi pi-times'"
                    />
                </template>
            </Column>

            <Column field="added_by" header="Added By" dataType="date" style="min-width: 10rem">
                <template #body="{ data }">
                    {{ data.added_by }}
                    <small class="block text-primary font-bold">{{ data.created_at }}</small>
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
import axios from 'axios';
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

const paymentMethods = ref([]);
const deposits = ref([]);
const loading = ref(false);
const filters = ref(null);

onBeforeMount(() => {
    loadDeposits();
    initFilters();
});

async function loadDeposits() {
    loading.value = true;
    // Simulate API call
    const response = await axios.get('/deposits');
    if (response.status === 200) {
        deposits.value = response.data.deposits;
        loading.value = false;
    } else {
        toast.add({
            severity: 'error',
            summary: 'Error Loading Deposits',
            detail: 'Failed to load deposit data',
            life: 3000
        });
        loading.value = false;
    }
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
