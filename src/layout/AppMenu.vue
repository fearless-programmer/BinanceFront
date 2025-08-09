<script setup>
import { ref } from 'vue';

import AppMenuItem from './AppMenuItem.vue';

const model = ref([
    {

        items: [
            { label: 'Dashboard', icon: 'pi pi-fw pi-home', to: '/' },
        ]
    },
    {
        label: 'Invest',
        items: [
            { label: 'Deposit', icon: 'pi pi-fw pi-plus-circle', to: '/deposits' },
            { label: 'Withdraw', icon: 'pi pi-fw pi-minus-circle', to: '/withdraws' },
            { label: 'Transactions', icon: 'pi pi-fw pi-list', to: '/transactions' },
            { label: 'Returns', icon: 'pi pi-fw pi-refresh', to: '/returns' }
        ]
    },
    {
        label: 'Settings',
        items: [
            { label: 'Profile', icon: 'pi pi-fw pi-user', to: '/profile' },
            { label: 'Users', icon: 'pi pi-fw pi-users', to: '/system-users' },
            { label: 'Settings', icon: 'pi pi-fw pi-cog', to: '/settings' },
        ]
    }
]);
</script>

<template>
    <ul class="layout-menu">
        <template v-for="(item, i) in model" :key="item">
            <app-menu-item v-if="!item.separator" :item="item" :index="i"></app-menu-item>
            <li v-if="item.separator" class="menu-separator"></li>
        </template>
    </ul>
</template>

<style lang="scss" scoped></style>
