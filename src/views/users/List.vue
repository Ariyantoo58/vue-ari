<script setup>
  import { ref } from 'vue';
  import { storeToRefs } from 'pinia';

  import { useUsersStore } from '@/stores';

  const usersStore = useUsersStore();
  const { users } = storeToRefs(usersStore);

  const showDeleteModal = ref(false);
  const userToDelete = ref(null);

  usersStore.getAll();

  function confirmDelete(user) {
    userToDelete.value = user;
    showDeleteModal.value = true;
  }

  function cancelDelete() {
    showDeleteModal.value = false;
    userToDelete.value = null;
  }

  async function executeDelete() {
    if (userToDelete.value) {
      await usersStore.delete(userToDelete.value.id);
      showDeleteModal.value = false;
      userToDelete.value = null;
    }
  }
</script>

<template>
  <h1>Users</h1>
  <router-link to="/users/add" class="btn btn-sm btn-success mb-2"
    >Add User</router-link
  >

  <div class="table-responsive">
    <table class="table table-striped">
      <thead>
        <tr>
          <th style="width: 30%">Name</th>
          <th style="width: 30%">Phone</th>
          <th style="width: 30%">email</th>
          <th style="width: 10%"></th>
        </tr>
      </thead>
      <tbody>
        <template v-if="users.length">
          <tr v-for="user in users" :key="user.id">
            <td>{{ user.name }}</td>
            <td>{{ user.phone }}</td>
            <td>{{ user.email }}</td>
            <td style="white-space: nowrap">
              <router-link
                :to="`/users/edit/${user.id}`"
                class="btn btn-sm btn-primary mr-1"
                >Edit</router-link
              >
              <button
                @click="confirmDelete(user)"
                class="btn btn-sm btn-danger btn-delete-user"
                :disabled="user.isDeleting"
              >
                <span
                  v-if="user.isDeleting"
                  class="spinner-border spinner-border-sm"
                ></span>
                <span v-else>Delete</span>
              </button>
            </td>
          </tr>
        </template>
        <tr v-if="users.loading">
          <td colspan="4" class="text-center">
            <span class="spinner-border spinner-border-lg align-center"></span>
          </td>
        </tr>
        <tr v-if="users.error">
          <td colspan="4">
            <div class="text-danger">
              Error loading users: {{ users.error }}
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Delete Confirmation Modal -->
  <div
    v-if="showDeleteModal"
    class="modal fade show d-block"
    tabindex="-1"
    style="background-color: rgba(0, 0, 0, 0.5)"
  >
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Konfirmasi Hapus</h5>
          <button
            type="button"
            class="close"
            @click="cancelDelete"
            aria-label="Close"
          >
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <p>
            Apakah Anda yakin ingin menghapus user
            <strong
              >{{ userToDelete?.firstName }}
              {{ userToDelete?.lastName }}</strong
            >?
          </p>
          <p class="text-muted mb-0">Tindakan ini tidak dapat dibatalkan.</p>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="cancelDelete">
            Batal
          </button>
          <button type="button" class="btn btn-danger" @click="executeDelete">
            Hapus
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .modal.show {
    display: block;
  }
</style>
