<script lang="ts" setup></script>

<template>
  <router-view />

  <router-link exact to="/login" class="link">
    <p>Admin</p>
  </router-link>
</template>

<style scoped>
.link p {
  color: var(--white);
}
</style>
