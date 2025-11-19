<script setup>
  import { ref, computed } from 'vue';
  import { storeToRefs } from 'pinia';
  import { useAuthStore } from '@/stores';

  const authStore = useAuthStore();
  const { user } = storeToRefs(authStore);
  console.log(user, 'userrr');

  // Form state
  const isEditing = ref(false);
  const formData = ref({
    name: user.value?.name || '',
    phone: user.value?.phone || '',
    email: user.value?.email || '',
    profile: user.value?.bio || '',
    location: user.value?.location || '',
  });

  // Profile image state
  const profileImage = ref(user.value?.profileImage || null);
  const previewImage = ref(null);
  const fileInput = ref(null);

  // Password change state
  const showPasswordChange = ref(false);
  const passwordData = ref({
    currentPassword: '',
    newPassword: '',
    confirmPassword: '',
  });

  // Get initials for avatar
  const initials = computed(() => {
    const name = formData.value.name?.charAt(0) || '';
    return name.toUpperCase();
  });

  // Handle file selection
  const handleFileSelect = (event) => {
    const file = event.target.files[0];
    if (file) {
      if (file.size > 5 * 1024 * 1024) {
        alert('File size must be less than 5MB');
        return;
      }

      if (!file.type.startsWith('image/')) {
        alert('Please select an image file');
        return;
      }

      const reader = new FileReader();
      reader.onload = (e) => {
        previewImage.value = e.target.result;
      };
      reader.readAsDataURL(file);
    }
  };

  // Trigger file input
  const triggerFileInput = () => {
    fileInput.value?.click();
  };

  // Remove profile image
  const removeImage = () => {
    previewImage.value = null;
    profileImage.value = null;
    if (fileInput.value) {
      fileInput.value.value = '';
    }
  };

  // Toggle edit mode
  const toggleEdit = () => {
    if (isEditing.value) {
      // Cancel - reset form
      formData.value = {
        name: user.value?.name || '',
        phone: user.value?.phone || '',
        email: user.value?.email || '',
        profile: user.value?.bio || '',
        location: user.value?.location || '',
      };
      previewImage.value = null;
    }
    isEditing.value = !isEditing.value;
  };

  // Save profile
  const saveProfile = () => {
    console.log('Saving profile:', formData.value);

    if (previewImage.value) {
      profileImage.value = previewImage.value;
    }

    isEditing.value = false;
    alert('Profile updated successfully!');
  };

  // Change password
  const changePassword = () => {
    if (passwordData.value.newPassword !== passwordData.value.confirmPassword) {
      alert('New passwords do not match!');
      return;
    }

    if (passwordData.value.newPassword.length < 6) {
      alert('Password must be at least 6 characters long!');
      return;
    }

    console.log('Changing password');

    passwordData.value = {
      currentPassword: '',
      newPassword: '',
      confirmPassword: '',
    };
    showPasswordChange.value = false;
    alert('Password changed successfully!');
  };
</script>

<template>
  <div class="profile-container">
    <div class="profile-wrapper">
      <!-- Header -->
      <div class="profile-header">
        <h1>My Profile</h1>
        <p>Manage your personal information and settings</p>
      </div>

      <!-- Profile Card -->
      <div class="profile-card">
        <!-- Cover Image -->
        <div class="cover-image"></div>

        <div class="card-content">
          <!-- Profile Image Section -->
          <div class="avatar-section">
            <div class="avatar-wrapper">
              <!-- Avatar -->
              <div class="avatar-container">
                <div class="avatar">
                  <img
                    v-if="previewImage || profileImage"
                    :src="previewImage || profileImage"
                    alt="Profile"
                  />
                  <span v-else class="avatar-initials">{{ initials }}</span>
                </div>

                <!-- Edit Photo Button -->
                <button
                  v-if="isEditing"
                  @click="triggerFileInput"
                  class="edit-photo-btn"
                  title="Change photo"
                >
                  <svg
                    width="20"
                    height="20"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"
                    />
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"
                    />
                  </svg>
                </button>

                <!-- Remove Photo Button -->
                <button
                  v-if="isEditing && (previewImage || profileImage)"
                  @click="removeImage"
                  class="remove-photo-btn"
                  title="Remove photo"
                >
                  <svg
                    width="16"
                    height="16"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M6 18L18 6M6 6l12 12"
                    />
                  </svg>
                </button>

                <input
                  ref="fileInput"
                  type="file"
                  accept="image/*"
                  @change="handleFileSelect"
                  style="display: none"
                />
              </div>

              <!-- Name and Edit Button -->
              <div class="user-info">
                <h2>{{ formData.name }}</h2>
                <p class="username">@{{ formData.name }}</p>
              </div>

              <!-- Edit/Save Button -->
              <div class="action-buttons">
                <button
                  v-if="!isEditing"
                  @click="toggleEdit"
                  class="btn btn-primary"
                >
                  <svg
                    width="20"
                    height="20"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
                    />
                  </svg>
                  <span>Edit Profile</span>
                </button>

                <div v-else class="button-group">
                  <button @click="toggleEdit" class="btn btn-secondary">
                    Cancel
                  </button>
                  <button @click="saveProfile" class="btn btn-success">
                    <svg
                      width="20"
                      height="20"
                      fill="none"
                      stroke="currentColor"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M5 13l4 4L19 7"
                      />
                    </svg>
                    <span>Save Changes</span>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- Profile Information -->
          <div class="form-grid">
            <!-- First Name -->
            <div class="form-group">
              <label>Name</label>
              <input
                v-model="formData.name"
                :disabled="!isEditing"
                type="text"
                class="form-input"
              />
            </div>

            <!-- Last Name -->
            <div class="form-group">
              <label>Phone Number</label>
              <input
                v-model="formData.phone"
                :disabled="!isEditing"
                type="text"
                class="form-input"
              />
            </div>

            <!-- EMAIL -->
            <div class="form-group">
              <label>Email</label>
              <input
                v-model="formData.email"
                :disabled="!isEditing"
                type="text"
                class="form-input"
              />
            </div>

            <!-- Email -->
            <div class="form-group">
              <label>Email</label>
              <input
                v-model="formData.email"
                :disabled="!isEditing"
                type="email"
                class="form-input"
              />
            </div>

            <!-- Bio (Full Width) -->
            <div class="form-group form-group-full">
              <label>Bio</label>
              <textarea
                v-model="formData.bio"
                :disabled="!isEditing"
                rows="4"
                placeholder="Tell us about yourself..."
                class="form-textarea"
              ></textarea>
            </div>
          </div>
        </div>
      </div>

      <!-- Security Section -->
      <div class="security-card">
        <div class="security-header">
          <div>
            <h3>Security</h3>
            <p>Manage your password and security settings</p>
          </div>
          <button
            @click="showPasswordChange = !showPasswordChange"
            class="btn btn-secondary"
          >
            {{ showPasswordChange ? 'Hide' : 'Change Password' }}
          </button>
        </div>

        <!-- Password Change Form -->
        <div v-if="showPasswordChange" class="password-form">
          <div class="form-group">
            <label>Current Password</label>
            <input
              v-model="passwordData.currentPassword"
              type="password"
              class="form-input"
            />
          </div>

          <div class="form-group">
            <label>New Password</label>
            <input
              v-model="passwordData.newPassword"
              type="password"
              class="form-input"
            />
          </div>

          <div class="form-group">
            <label>Confirm New Password</label>
            <input
              v-model="passwordData.confirmPassword"
              type="password"
              class="form-input"
            />
          </div>

          <button @click="changePassword" class="btn btn-primary btn-full">
            Update Password
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  .profile-container {
    min-height: 100vh;
    background: linear-gradient(135deg, #3d4249 0%, #2d59bf 100%);
    padding: 2rem 1rem;
  }

  .profile-wrapper {
    max-width: 1000px;
    margin: 0 auto;
  }

  .profile-header {
    margin-bottom: 2rem;
  }

  .profile-header h1 {
    font-size: 2rem;
    font-weight: bold;
    color: white;
    margin-bottom: 0.5rem;
  }

  .profile-header p {
    color: rgba(255, 255, 255, 0.9);
    font-size: 1rem;
  }

  .profile-card {
    background: white;
    border-radius: 1rem;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
    overflow: hidden;
    margin-bottom: 1.5rem;
  }

  .cover-image {
    height: 128px;
    background: linear-gradient(90deg, #3d4249 0%, #2d59bf 100%);
  }

  .card-content {
    padding: 0 1.5rem 1.5rem;
  }

  .avatar-section {
    margin-top: -64px;
    margin-bottom: 1.5rem;
  }

  .avatar-wrapper {
    display: flex;
    align-items: flex-end;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .avatar-container {
    position: relative;
  }

  .avatar {
    width: 128px;
    height: 128px;
    border-radius: 50%;
    border: 4px solid white;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
    overflow: hidden;
    background: linear-gradient(135deg, #3d4249 0%, #2d59bf 100%);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .avatar img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .avatar-initials {
    color: white;
    font-size: 2rem;
    font-weight: bold;
  }

  .edit-photo-btn {
    position: absolute;
    bottom: 0;
    right: 0;
    background: #667eea;
    color: white;
    padding: 0.5rem;
    border-radius: 50%;
    border: none;
    cursor: pointer;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: background 0.3s;
  }

  .edit-photo-btn:hover {
    background: #5568d3;
  }

  .remove-photo-btn {
    position: absolute;
    top: 0;
    right: 0;
    background: #ef4444;
    color: white;
    padding: 0.375rem;
    border-radius: 50%;
    border: none;
    cursor: pointer;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: background 0.3s;
  }

  .remove-photo-btn:hover {
    background: #dc2626;
  }

  .user-info {
    flex: 1;
    padding-bottom: 0.5rem;
  }

  .user-info h2 {
    font-size: 1.5rem;
    font-weight: bold;
    color: #1f2937;
    margin-bottom: 0.25rem;
  }

  .username {
    color: #6b7280;
    font-size: 1rem;
  }

  .action-buttons {
    padding-bottom: 0.5rem;
  }

  .button-group {
    display: flex;
    gap: 0.5rem;
  }

  .btn {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.625rem 1.5rem;
    border-radius: 0.5rem;
    border: none;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s;
    font-size: 0.95rem;
  }

  .btn-primary {
    background: #667eea;
    color: white;
  }

  .btn-primary:hover {
    background: #5568d3;
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
  }

  .btn-secondary {
    background: #e5e7eb;
    color: #374151;
  }

  .btn-secondary:hover {
    background: #d1d5db;
  }

  .btn-success {
    background: #10b981;
    color: white;
  }

  .btn-success:hover {
    background: #059669;
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
  }

  .btn-full {
    width: 100%;
    justify-content: center;
  }

  .form-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  @media (min-width: 768px) {
    .form-grid {
      grid-template-columns: 1fr 1fr;
    }
  }

  .form-group {
    display: flex;
    flex-direction: column;
  }

  .form-group-full {
    grid-column: 1 / -1;
  }

  .form-group label {
    font-size: 0.875rem;
    font-weight: 500;
    color: #374151;
    margin-bottom: 0.5rem;
  }

  .form-input {
    width: 100%;
    padding: 0.75rem 1rem;
    border: 1px solid #d1d5db;
    border-radius: 0.5rem;
    font-size: 1rem;
    transition: all 0.3s;
  }

  .form-input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }

  .form-input:disabled {
    background: #f9fafb;
    color: #6b7280;
    cursor: not-allowed;
  }

  .form-textarea {
    width: 100%;
    padding: 0.75rem 1rem;
    border: 1px solid #d1d5db;
    border-radius: 0.5rem;
    font-size: 1rem;
    resize: vertical;
    font-family: inherit;
    transition: all 0.3s;
  }

  .form-textarea:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }

  .form-textarea:disabled {
    background: #f9fafb;
    color: #6b7280;
    cursor: not-allowed;
  }

  .security-card {
    background: white;
    border-radius: 1rem;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
    padding: 1.5rem;
    margin-bottom: 1.5rem;
  }

  .security-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
    flex-wrap: wrap;
    gap: 1rem;
  }

  .security-header h3 {
    font-size: 1.25rem;
    font-weight: bold;
    color: #1f2937;
    margin-bottom: 0.25rem;
  }

  .security-header p {
    color: #6b7280;
    font-size: 0.875rem;
  }

  .password-form {
    margin-top: 1rem;
    padding-top: 1rem;
    border-top: 1px solid #e5e7eb;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .stats-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  @media (min-width: 768px) {
    .stats-grid {
      grid-template-columns: repeat(3, 1fr);
    }
  }

  .stat-card {
    background: white;
    border-radius: 0.75rem;
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
    padding: 1.5rem;
    text-align: center;
  }

  .stat-value {
    font-size: 1.875rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
  }

  .stat-label {
    color: #6b7280;
    font-size: 0.875rem;
  }

  .stat-primary .stat-value {
    color: #667eea;
  }

  .stat-success .stat-value {
    color: #10b981;
  }

  .stat-purple .stat-value {
    color: #8b5cf6;
  }

  /* Responsive */
  @media (max-width: 768px) {
    .profile-header h1 {
      font-size: 1.5rem;
    }

    .avatar {
      width: 96px;
      height: 96px;
    }

    .avatar-section {
      margin-top: -48px;
    }

    .user-info h2 {
      font-size: 1.25rem;
    }

    .avatar-wrapper {
      flex-direction: column;
      align-items: flex-start;
    }

    .action-buttons {
      width: 100%;
    }

    .button-group {
      width: 100%;
    }

    .btn {
      flex: 1;
    }
  }
</style>
