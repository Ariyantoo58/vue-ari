<script setup>
  import { ref, watch } from 'vue';
  import { Form, Field } from 'vee-validate';
  import * as Yup from 'yup';
  import { useRoute } from 'vue-router';
  import { storeToRefs } from 'pinia';
  import * as L from 'leaflet';
  import 'leaflet/dist/leaflet.css';

  import { useUsersStore, useAlertStore } from '@/stores';
  import { router } from '@/router';

  const usersStore = useUsersStore();
  const alertStore = useAlertStore();
  const route = useRoute();
  const id = route.params.id;

  let title = 'Add User';
  let user = null;

  // Profile image preview
  const profilePreview = ref(null);
  const profileFile = ref(null);
  const profileBase64 = ref(null);

  // Map setup
  const mapContainer = ref(null);
  const map = ref(null);
  const marker = ref(null);
  const selectedLocation = ref({ lat: -6.9175, lng: 107.6191 }); // Default: Sukabumi

  if (id) {
    // edit mode
    title = 'Edit User';
    ({ user } = storeToRefs(usersStore));
    usersStore.getById(id);
  }

  const schema = Yup.object({
    profile: Yup.mixed()
      .test('fileType', 'Only image files are allowed', (value) => {
        if (!value) return true; // no new file → OK
        if (typeof value === 'string') return true; // base64 from edit → OK
        return ['image/jpeg', 'image/png', 'image/jpg', 'image/gif'].includes(
          value.type
        );
      })
      .test('fileSize', 'File size must be less than 1MB', (value) => {
        if (!value || typeof value === 'string') return true;
        return value.size <= 1000000;
      }),
    name: Yup.string().required(),
    phone: Yup.number().required(),
    email: Yup.string().email().required(),
    password: Yup.string()
      .transform((v) => (v === '' ? undefined : v))
      .min(6, 'Password must be at least 6 characters'),
  });

  // Handle file selectionmapContainer
  function handleFileChange(event, setFieldValue) {
    const file = event.target.files[0];
    if (file) {
      profileFile.value = file;
      setFieldValue('profile', file); // save FILE
      const reader = new FileReader();
      reader.onload = (e) => {
        profilePreview.value = e.target.result; // BASE64
        profileBase64.value = e.target.result;
      };
      reader.readAsDataURL(file);
    }
  }

  // Trigger file input click
  function triggerFileInput() {
    document.querySelector('input[name="profile"]').click();
  }

  // Remove profile image
  function removeProfileImage() {
    profilePreview.value = null;
    profileFile.value = null;
    const fileInput = document.querySelector('input[name="profile"]');
    if (fileInput) fileInput.value = '';
  }

  // Initialize map
  function initMap() {
    if (mapContainer.value && !map.value) {
      map.value = L.map(mapContainer.value).setView(
        [selectedLocation.value.lat, selectedLocation.value.lng],
        13
      );

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(map.value);

      console.log(selectedLocation, 'selectedLocation');

      marker.value = L.marker(
        [selectedLocation.value.lat, selectedLocation.value.lng],
        {
          draggable: true,
        }
      ).addTo(map.value);

      // Update location when marker is dragged
      marker.value.on('dragend', function (e) {
        const position = e.target.getLatLng();
        selectedLocation.value = { lat: position.lat, lng: position.lng };
      });

      // Add click event to map
      map.value.on('click', function (e) {
        selectedLocation.value = { lat: e.latlng.lat, lng: e.latlng.lng };
        marker.value.setLatLng(e.latlng);
      });
    }
  }

  // Watch for user data in edit mode
  watch(
    () => user?.value,
    (newUser) => {
      if (newUser) {
        if (newUser.profile) {
          profilePreview.value = newUser.profile;
        }
        if (newUser.loclat || newUser.loclng) {
          let location = { lat: newUser.loclat, lng: newUser.loclng };
          selectedLocation.value = location;
          if (map.value && marker.value) {
            map.value.setView([newUser.loclat, newUser.loclng], 13);
            marker.value.setLatLng([newUser.loclat, newUser.loclng]);
          }
        }
      }
    },
    { immediate: true }
  );

  // Initialize map after component mount
  watch(mapContainer, (val) => {
    if (val && !map.value) {
      initMap();
    }
  });

  async function onSubmit(values) {
    try {
      let message;

      values.loclat = selectedLocation.value.lat;
      values.loclng = selectedLocation.value.lng;

      if (profileFile.value) {
        values.profile = profileBase64.value; // base64 saved
      }

      if (user) {
        await usersStore.update(user.value.id, values);
        message = 'User updated';
      } else {
        await usersStore.register(values);
        message = 'User added';
      }
      await router.push('/users');
      alertStore.success(message);
    } catch (error) {
      alertStore.error(error);
    }
  }
</script>

<template>
  <h1 class="mb-4">{{ title }}</h1>
  <template v-if="!(user?.loading || user?.error)">
    <Form
      @submit="onSubmit"
      :validation-schema="schema"
      :initial-values="{ ...user }"
      v-slot="{ errors, isSubmitting, setFieldValue }"
    >
      <!-- Profile Upload Section with Modern Design -->
      <div class="form-group">
        <div class="profile-upload-container">
          <!-- Hidden file input -->
          <Field
            type="file"
            name="profile"
            accept="image/*"
            @change="(e) => handleFileChange(e, setFieldValue)"
            style="display: none"
          />

          <!-- Profile Preview Area -->
          <div class="profile-preview-wrapper">
            <div
              v-if="profilePreview"
              class="profile-preview-image"
              :style="{ backgroundImage: `url(${profilePreview})` }"
            >
              <button
                type="button"
                @click="removeProfileImage"
                class="btn btn-danger btn-sm profile-remove-btn"
                title="Remove image"
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
              <button
                type="button"
                @click="triggerFileInput"
                class="edit-photo-btn"
                title="Change image"
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
            </div>

            <!-- Upload placeholder when no image -->
            <div
              v-else
              class="profile-upload-placeholder"
              @click="triggerFileInput"
            >
              <i class="fa fa-camera fa-2x mb-2"></i>
              <p class="mb-0">Click to upload</p>
              <small class="text-muted">Image Only (max. 2MB)</small>
            </div>
          </div>

          <!-- Error message -->
          <div v-if="errors.profile" class="text-danger mt-2 small">
            {{ errors.profile }}
          </div>
        </div>
      </div>

      <!-- Form fields in responsive grid -->
      <div class="row">
        <div class="col-md-6">
          <div class="form-group">
            <label>Name</label>
            <Field
              name="name"
              type="text"
              class="form-control"
              :class="{ 'is-invalid': errors.name }"
            />
            <div class="invalid-feedback">{{ errors.name }}</div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="form-group">
            <label>Phone Number</label>
            <Field
              name="phone"
              type="text"
              class="form-control"
              :class="{ 'is-invalid': errors.phone }"
            />
            <div class="invalid-feedback">{{ errors.phone }}</div>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-md-6">
          <div class="form-group">
            <label>Email</label>
            <Field
              name="email"
              type="email"
              class="form-control"
              :class="{ 'is-invalid': errors.email }"
            />
            <div class="invalid-feedback">{{ errors.email }}</div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="form-group">
            <label>
              Password
              <em v-if="user" class="text-muted small"
                >(Leave blank to keep the same password)</em
              >
            </label>
            <Field
              name="password"
              type="password"
              class="form-control"
              :class="{ 'is-invalid': errors.password }"
            />
            <div class="invalid-feedback">{{ errors.password }}</div>
          </div>
        </div>
      </div>

      <!-- Map Section -->
      <div class="form-group">
        <label>Location</label>
        <div ref="mapContainer" class="map-container"></div>
        <small class="form-text text-muted">
          <i class="fa fa-map-marker"></i> Click on the map or drag the marker
          to select location. <strong>Selected:</strong>
          {{ selectedLocation.lat.toFixed(6) }},
          {{ selectedLocation.lng.toFixed(6) }}
        </small>
      </div>

      <div class="form-group mt-4">
        <button class="btn btn-primary btn-lg" :disabled="isSubmitting">
          <span
            v-show="isSubmitting"
            class="spinner-border spinner-border-sm mr-2"
          ></span>
          <i v-show="!isSubmitting" class="fa fa-save mr-2"></i>
          Save
        </button>
        <router-link to="/users" class="btn btn-link btn-lg"
          >Cancel</router-link
        >
      </div>
    </Form>
  </template>
  <template v-if="user?.loading">
    <div class="text-center m-5">
      <span class="spinner-border spinner-border-lg align-center"></span>
    </div>
  </template>
  <template v-if="user?.error">
    <div class="text-center m-5">
      <div class="text-danger">User not found</div>
    </div>
  </template>
</template>

<style scoped>
  /* Profile Upload Styling */
  .profile-upload-container {
    margin-bottom: 1.5rem;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .profile-preview-wrapper {
    display: inline-block;
  }

  .profile-preview-image {
    width: 150px;
    height: 150px;
    border-radius: 50%;
    background-size: cover;
    background-position: center;
    position: relative;
    border: 4px solid #f0f0f0;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .profile-remove-btn {
    position: absolute;
    top: 5px;
    right: 5px;
    width: 30px;
    height: 30px;
    border-radius: 50%;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s;
  }

  .profile-change-btn {
    position: absolute;
    bottom: 5px;
    right: 5px;
    width: 35px;
    height: 35px;
    border-radius: 50%;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s;
  }

  .profile-preview-image:hover .profile-remove-btn,
  .profile-preview-image:hover .profile-change-btn {
    opacity: 1;
  }

  .profile-upload-placeholder {
    width: 150px;
    height: 150px;
    border-radius: 50%;
    border: 3px dashed #ccc;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.3s;
    background-color: #f8f9fa;
  }

  .profile-upload-placeholder:hover {
    border-color: #007bff;
    background-color: #e7f3ff;
  }

  .profile-upload-placeholder i {
    color: #6c757d;
  }

  .profile-upload-placeholder:hover i {
    color: #007bff;
  }

  /* Map Styling */
  .map-container {
    height: 400px;
    width: 100%;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  /* Responsive adjustments */
  @media (max-width: 768px) {
    .map-container {
      height: 300px;
    }

    .profile-preview-image,
    .profile-upload-placeholder {
      width: 120px;
      height: 120px;
    }

    .btn-lg {
      font-size: 1rem;
      padding: 0.5rem 1rem;
    }
  }

  @media (max-width: 576px) {
    .map-container {
      height: 250px;
    }

    h1 {
      font-size: 1.75rem;
    }
  }

  /* Form styling improvements */
  .form-group label {
    font-weight: 500;
    color: #495057;
    margin-bottom: 0.5rem;
  }

  .form-control {
    border-radius: 6px;
    padding: 0.625rem 0.75rem;
  }

  .form-control:focus {
    border-color: #80bdff;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
  }

  .btn-primary {
    border-radius: 6px;
    padding: 0.625rem 1.5rem;
    font-weight: 500;
  }

  .btn-link {
    color: #6c757d;
    text-decoration: none;
  }

  .btn-link:hover {
    color: #495057;
    text-decoration: underline;
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
</style>
