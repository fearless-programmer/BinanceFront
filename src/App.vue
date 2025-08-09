<script setup>
import axios from 'axios';

axios.defaults.baseURL = 'http://localhost:8000/api';
</script>

<template>
    <router-view />
</template>

<style scoped></style>
