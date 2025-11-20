<script setup>
  import { storeToRefs } from 'pinia';
  import { useAuthStore } from '@/stores';

  const authStore = useAuthStore();
  const { user } = storeToRefs(authStore);
</script>

<template>
  <div
    v-if="user"
    class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center px-4"
  >
    <div class="bg-white rounded-xl shadow-xl p-8 max-w-md w-full text-center">
      <div
        class="w-20 h-20 bg-indigo-600 rounded-full flex items-center justify-center mx-auto mb-6"
      >
        <span class="text-3xl text-white font-bold">{{
          user.name.charAt(0).toUpperCase()
        }}</span>
      </div>

      <h1 class="text-3xl font-bold text-gray-800 mb-3">
        Hi {{ user.name }}! 👋
      </h1>

      <p class="text-gray-600 mb-6">You're logged in vue-aridevs project</p>

      <router-link
        to="/users"
        class="inline-block px-6 py-3 bg-indigo-600 text-white rounded-lg font-medium hover:bg-indigo-700 transition-colors duration-200"
      >
        Manage Users
      </router-link>
    </div>
  </div>
</template>
