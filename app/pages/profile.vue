<template>
  <div class="profile-page">
    <v-container class="py-8" style="max-width: 560px">
      <v-card v-if="user" class="profile-card pa-8 text-center">
        <v-avatar size="112" class="profile-avatar mb-4">
          <v-img v-if="user.picture" :src="user.picture" referrerpolicy="no-referrer" />
          <v-icon v-else icon="mdi-account" size="56" />
        </v-avatar>

        <h2 class="profile-name">{{ user.name }}</h2>
        <p class="profile-email">{{ user.email }}</p>

        <v-divider class="profile-divider my-6" />

        <v-btn block variant="tonal" class="logout-btn" prepend-icon="mdi-logout" @click="logout">
          Logout
        </v-btn>
      </v-card>
    </v-container>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

const user = ref<any>(null)

onMounted(() => {
  const savedUser = localStorage.getItem('google_user')
  if (savedUser) {
    user.value = JSON.parse(savedUser)
    console.log('user object:', user.value)
    console.log('picture url:', user.value.picture)
  }
})

const logout = () => {
  localStorage.removeItem('google_user')
  localStorage.removeItem('google_token')

  router.push('/login')
}
</script>

<style>
/* White and red theme */
.profile-page {
  --v-theme-primary: 198, 40, 40;
  --v-theme-surface: 255, 255, 255;
  --v-theme-on-surface: 0, 0, 0;
  --v-theme-background: 255, 255, 255;
  --v-theme-on-background: 0, 0, 0;
  --v-border-color: 198, 40, 40;
  --v-border-opacity: 0.35;

  min-height: calc(100vh - 64px);
  background: #ffffff;
  color: #000000;
}

.v-main:has(.profile-page) {
  background: #ffffff;
}

/* Profile card */
.profile-page .profile-card {
  background: #fff5f5;
  color: #000000;
  border: 1px solid #e57373;
  border-radius: 12px;
  box-shadow: 0 5px 20px rgba(198, 40, 40, 0.12);
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Avatar */
.profile-page .profile-avatar {
  background: #ffcdd2;
  border: 3px solid #c62828;
  box-shadow: 0 0 18px rgba(198, 40, 40, 0.2);
}

.profile-page .profile-avatar .v-icon {
  color: #b71c1c;
}

/* Name + email */
.profile-name {
  font-size: 1.6rem;
  font-weight: 600;
  line-height: 1.3;
  color: #000000;
}

.profile-email {
  margin: 4px 0 0;
  color: #000000;
  opacity: 0.75;
  word-break: break-all;
}

/* Divider */
.profile-page .profile-divider {
  width: 100%;
  border-color: #e57373;
  opacity: 1;
}

/* Logout button */
.profile-page .logout-btn {
  background: #ffcdd2;
  color: #000000;
  border: 1px solid #c62828;
  font-weight: 600;
}

.profile-page .logout-btn:hover {
  background: #ef9a9a;
  border-color: #b71c1c;
}
</style>
