<script setup>
  import { ref } from 'vue';
  import { useAuthStore } from '@/stores';

  const authStore = useAuthStore();
  const showDropdown = ref(false);
  const showMobileMenu = ref(false);

  const toggleDropdown = () => {
    showDropdown.value = !showDropdown.value;
  };

  const toggleMobileMenu = () => {
    showMobileMenu.value = !showMobileMenu.value;
  };

  const closeMobileMenu = () => {
    showMobileMenu.value = false;
  };

  // Close dropdown when clicking outside
  const handleClickOutside = (event) => {
    const dropdown = document.querySelector('.profile-dropdown');
    const mobileMenu = document.querySelector('.mobile-menu');

    if (dropdown && !dropdown.contains(event.target)) {
      showDropdown.value = false;
    }

    if (
      mobileMenu &&
      !mobileMenu.contains(event.target) &&
      !event.target.closest('.hamburger-btn')
    ) {
      showMobileMenu.value = false;
    }
  };

  // Add event listener when component is mounted
  if (typeof window !== 'undefined') {
    window.addEventListener('click', handleClickOutside);
  }
</script>

<template>
  <nav v-show="authStore.user" class="navbar navbar-expand navbar-dark bg-dark">
    <div class="navbar-container">
      <!-- Logo/Brand (optional) -->
      <div class="navbar-brand">MyApp</div>

      <!-- Desktop Menu -->
      <div class="navbar-nav desktop-menu">
        <router-link to="/" class="nav-item nav-link">Home</router-link>
        <router-link to="/users" class="nav-item nav-link">Users</router-link>
      </div>

      <!-- Desktop Profile Section -->
      <div class="profile-dropdown desktop-profile" v-if="authStore.user">
        <div class="profile-trigger" @click.stop="toggleDropdown">
          <img
            :src="authStore.user.profile"
            :alt="authStore.user.name"
            class="profile-avatar"
          />
          <span class="profile-name">{{ authStore.user.name }}</span>
          <i class="dropdown-icon">▼</i>
        </div>

        <!-- Dropdown Menu -->
        <div v-show="showDropdown" class="dropdown-menu-custom">
          <router-link
            to="/profile"
            class="dropdown-item"
            @click="showDropdown = false"
          >
            <i class="icon">👤</i> Profile
          </router-link>
          <div class="dropdown-divider"></div>
          <button @click="authStore.logout()" class="dropdown-item logout-btn">
            <i class="icon">🚪</i> Logout
          </button>
        </div>
      </div>

      <!-- Hamburger Button -->
      <button class="hamburger-btn" @click.stop="toggleMobileMenu">
        <span class="hamburger-line" :class="{ active: showMobileMenu }"></span>
        <span class="hamburger-line" :class="{ active: showMobileMenu }"></span>
        <span class="hamburger-line" :class="{ active: showMobileMenu }"></span>
      </button>
    </div>

    <!-- Mobile Menu -->
    <div class="mobile-menu" :class="{ show: showMobileMenu }">
      <!-- Mobile Profile Header -->
      <div class="mobile-profile-header" v-if="authStore.user">
        <img
          :src="authStore.user.profile"
          :alt="authStore.user.name"
          class="profile-avatar-mobile"
        />
        <div class="profile-info">
          <span class="profile-name-mobile">{{ authStore.user.name }}</span>
          <span class="profile-email-mobile">{{ authStore.user.email }}</span>
        </div>
      </div>

      <div class="mobile-divider"></div>

      <!-- Mobile Navigation Links -->
      <router-link to="/" class="mobile-nav-item" @click="closeMobileMenu">
        <i class="icon">🏠</i> Home
      </router-link>
      <router-link to="/users" class="mobile-nav-item" @click="closeMobileMenu">
        <i class="icon">👥</i> Users
      </router-link>
      <router-link
        to="/profile"
        class="mobile-nav-item"
        @click="closeMobileMenu"
      >
        <i class="icon">👤</i> Profile
      </router-link>

      <div class="mobile-divider"></div>

      <button @click="authStore.logout()" class="mobile-nav-item logout-btn">
        <i class="icon">🚪</i> Logout
      </button>
    </div>
  </nav>
</template>

<style scoped>
  .navbar {
    position: relative;
    padding: 0.5rem 1rem;
  }

  .navbar-container {
    display: flex;
    align-items: center;
    width: 100%;
  }

  .navbar-brand {
    color: #fff;
    font-size: 1.25rem;
    font-weight: bold;
    margin-right: 1rem;
  }

  .desktop-menu {
    display: flex;
    gap: 0.5rem;
  }

  .profile-dropdown {
    position: relative;
    margin-left: auto;
    margin-right: 1rem;
  }

  .profile-trigger {
    display: flex;
    align-items: center;
    gap: 10px;
    cursor: pointer;
    padding: 5px 10px;
    border-radius: 5px;
    transition: background-color 0.3s;
  }

  .profile-trigger:hover {
    background-color: rgba(255, 255, 255, 0.1);
  }

  .profile-avatar {
    width: 35px;
    height: 35px;
    border-radius: 50%;
    object-fit: cover;
    border: 2px solid #fff;
  }

  .profile-name {
    color: #fff;
    font-size: 14px;
    font-weight: 500;
  }

  .dropdown-icon {
    color: #fff;
    font-size: 10px;
    transition: transform 0.3s;
  }

  .profile-trigger:hover .dropdown-icon {
    transform: translateY(2px);
  }

  .dropdown-menu-custom {
    position: absolute;
    top: 100%;
    right: 0;
    margin-top: 5px;
    background-color: #fff;
    border-radius: 5px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    min-width: 200px;
    z-index: 1000;
    overflow: hidden;
  }

  .dropdown-item {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 12px 16px;
    color: #333;
    text-decoration: none;
    transition: background-color 0.2s;
    border: none;
    background: none;
    width: 100%;
    text-align: left;
    cursor: pointer;
    font-size: 14px;
  }

  .dropdown-item:hover {
    background-color: #f8f9fa;
  }

  .logout-btn:hover {
    background-color: #ffe6e6;
    color: #dc3545;
  }

  .dropdown-divider {
    height: 1px;
    background-color: #e9ecef;
    margin: 0;
  }

  .icon {
    font-size: 16px;
  }

  /* Hamburger Menu */
  .hamburger-btn {
    display: none;
    flex-direction: column;
    gap: 4px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 8px;
    margin-left: auto;
  }

  .hamburger-line {
    width: 25px;
    height: 3px;
    background-color: #fff;
    transition: all 0.3s;
    border-radius: 3px;
  }

  .hamburger-line.active:nth-child(1) {
    transform: rotate(45deg) translate(5px, 5px);
  }

  .hamburger-line.active:nth-child(2) {
    opacity: 0;
  }

  .hamburger-line.active:nth-child(3) {
    transform: rotate(-45deg) translate(7px, -6px);
  }

  /* Mobile Menu */
  .mobile-menu {
    display: none;
    position: fixed;
    top: 56px;
    right: -100%;
    width: 280px;
    height: calc(100vh - 56px);
    background-color: #343a40;
    transition: right 0.3s ease-in-out;
    z-index: 999;
    overflow-y: auto;
    box-shadow: -2px 0 10px rgba(0, 0, 0, 0.3);
  }

  .mobile-menu.show {
    right: 0;
  }

  .mobile-profile-header {
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 20px;
    background-color: rgba(255, 255, 255, 0.05);
  }

  .profile-avatar-mobile {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    object-fit: cover;
    border: 2px solid #fff;
  }

  .profile-info {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .profile-name-mobile {
    color: #fff;
    font-size: 16px;
    font-weight: 600;
  }

  .profile-email-mobile {
    color: #adb5bd;
    font-size: 12px;
  }

  .mobile-divider {
    height: 1px;
    background-color: rgba(255, 255, 255, 0.1);
    margin: 10px 0;
  }

  .mobile-nav-item {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 15px 20px;
    color: #fff;
    text-decoration: none;
    transition: background-color 0.2s;
    border: none;
    background: none;
    width: 100%;
    text-align: left;
    cursor: pointer;
    font-size: 15px;
  }

  .mobile-nav-item:hover {
    background-color: rgba(255, 255, 255, 0.1);
  }

  .mobile-nav-item.logout-btn {
    color: #f8d7da;
  }

  .mobile-nav-item.logout-btn:hover {
    background-color: rgba(220, 53, 69, 0.2);
  }

  /* Responsive */
  @media (max-width: 768px) {
    .desktop-menu {
      display: none;
    }

    .desktop-profile {
      display: none;
    }

    .hamburger-btn {
      display: flex;
    }

    .mobile-menu {
      display: block;
    }

    .navbar-brand {
      font-size: 1.1rem;
    }
  }

  @media (max-width: 480px) {
    .mobile-menu {
      width: 100%;
      right: -100%;
    }

    .mobile-menu.show {
      right: 0;
    }
  }
</style>
