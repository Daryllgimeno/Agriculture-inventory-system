
<template>
  <div class="min-h-screen flex flex-col justify-center items-center bg-gray-100">
 
    <header class="w-full max-w-md mb-6 text-center">
      <slot name="header"></slot>
    </header>

    <main class="w-full max-w-md bg-white shadow-md rounded-lg p-8">
      <slot></slot>
    </main>

    <footer class="mt-6 text-center text-gray-500 text-sm">
      &copy; {{ new Date().getFullYear() }} Agriculture Inventory System. All rights reserved.
    </footer>
  </div>
</template>

<script setup>

</script>

<style scoped>

</style>
