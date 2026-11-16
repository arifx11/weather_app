<template>
  <div class="login-page">
    <v-container class="d-flex align-center justify-center" style="min-height: 100vh">
      <v-card width="400" class="login-card pa-6 rounded-xl">
        <v-card-title class="text-center text-h5 login-title">
          Login
        </v-card-title>

        <v-card-subtitle class="text-center login-subtitle mb-4">
          Sign in to System Integration
        </v-card-subtitle>

        <v-card-text>
          <v-btn block variant="tonal" size="large" class="google-btn" prepend-icon="mdi-google"
            @click="loginWithGoogle">
            Sign in with Google
          </v-btn>
        </v-card-text>
      </v-card>
    </v-container>
  </div>
</template>

<script setup lang="ts">
// @ts-nocheck

const config = useRuntimeConfig()

definePageMeta({ layout: false })

declare global {
  interface Window {
    google: any
  }
}

console.log('Google Client ID:', config.public.googleClientId)

const loginWithGoogle = () => {
  console.log('Using Client ID:', config.public.googleClientId)

  const client = window.google.accounts.oauth2.initTokenClient({
    client_id: config.public.googleClientId,
    scope: 'openid email profile',

    callback: async (response: any) => {
      const userInfo = await $fetch(
        'https://www.googleapis.com/oauth2/v3/userinfo',
        {
          headers: {
            Authorization: `Bearer ${response.access_token}`
          }
        }
      )

      localStorage.setItem(
        'google_user',
        JSON.stringify(userInfo)
      )

      localStorage.setItem(
        'google_token',
        response.access_token
      )

      navigateTo('/')
    }
  })

  client.requestAccessToken()
}
</script>

<!-- Not scoped on purpose: Vuetify's inner elements live outside this component's scope ID. -->
<style>
/* ---------- Login Page: White and Red Theme ---------- */

.login-page {
  --v-theme-primary: 198, 40, 40;
  --v-theme-surface: 255, 255, 255;
  --v-theme-on-surface: 0, 0, 0;
  --v-theme-background: 255, 255, 255;
  --v-theme-on-background: 0, 0, 0;
  --v-border-color: 198, 40, 40;
  --v-border-opacity: 0.35;

  min-height: 100vh;
  color: #000000;

  background:
    radial-gradient(circle at 50% 35%,
      rgba(198, 40, 40, 0.12),
      transparent 55%),
    #ffffff;
}

/* Application background */
.v-application:has(.login-page) {
  background: #ffffff !important;
}

/* Login card */
.login-page .login-card {
  background: #fff5f5;
  color: #000000;
  border: 1px solid #ef9a9a;
  box-shadow: 0 5px 24px rgba(198, 40, 40, 0.15);
  border-radius: 16px !important;
}

/* Login title */
.login-page .login-title {
  color: #000000;
  font-weight: 700;
}

/* Subtitle */
.login-page .login-subtitle {
  color: #555555;
  opacity: 1;
}

/* Google button */
.login-page .google-btn {
  background: #ffebee;
  color: #000000;
  border: 1px solid #ef9a9a;
  border-radius: 10px;
  font-weight: 600;
  transition: all 0.25s ease;
}

/* Google icon */
.login-page .google-btn :deep(.v-icon) {
  color: #c62828;
}

/* Button hover */
.login-page .google-btn:hover {
  background: #ffcdd2;
  border-color: #c62828;
  box-shadow: 0 4px 12px rgba(198, 40, 40, 0.15);
}

/* Button click */
.login-page .google-btn:active {
  transform: scale(0.98);
}
</style>
