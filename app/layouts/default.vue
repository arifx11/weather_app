<template>
    <v-layout class="main-layout">
        <!-- Navigation Drawer -->
        <v-navigation-drawer permanent :rail="rail" @mouseenter="rail = false" @mouseleave="rail = true"
            class="main-drawer">
            <v-list-item :height="headerHeight" to="/profile"
                prepend-avatar="https://scontent.fcrk3-3.fna.fbcdn.net/v/t39.30808-1/853576245_29423010473951350_3225309618017184530_n.avif?stp=c52.0.590.590a_dst-jpg_tt6&cstp=mx590x590&ctp=s200x200&_nc_cat=107&_nc_map=urlgen_bucketless&ccb=1-7&_nc_sid=e99d92&_nc_eui2=AeEMjxJFwgxeaQJ1EaljiBXmT51At_uVDKZPnUC3-5UMplm_gyDi8UJa6oNJDqiGyhlt9iJaanqmG1wf94ts2Ig3&_nc_ohc=0vTIZ6EFyw8Q7kNvwGiNMXE&_nc_oc=AdpRTd_1mnm2QePkU3TY4ZzoiXkHHayAPhRJEtzHqYDIgTvVnsiE9iNLn-c-tTBqSnF_Ip0p2W_ZOZ6Fxr1au02h&_nc_zt=24&_nc_ht=scontent.fcrk3-3.fna&_nc_gid=i_Dv5PGzwBxVQ1g3qqHRRw&_nc_ss=7b2a8&oh=00_AQS8jSXBM-JR0-eTNoahI6foFjMbFUoGhzbZ_fEwjmTwjQ&oe=6B00FF4F"
                subtitle="Open details..." title="Profile" />

            <v-divider />

            <!-- Navigation Items -->
            <v-list density="compact" nav>
                <v-list-item prepend-icon="mdi-view-dashboard" title="Dashboard" value="dashboard" to="/dashboard" />

                <v-list-item prepend-icon="mdi-qrcode-scan" title="QR Scanner" value="qr_scanner" to="/qr_scanner" />

                <v-list-item prepend-icon="mdi-weather-partly-cloudy" title="Weather App" value="weather_app"
                    to="/weather_app" />
            </v-list>
        </v-navigation-drawer>

        <!-- Top App Bar -->
        <v-app-bar class="app-bar-gradient" flat>
            <v-app-bar-title>System Integration</v-app-bar-title>
        </v-app-bar>

        <!-- Main Content -->
        <v-main class="main-content">
            <slot />
        </v-main>
    </v-layout>
</template>


<script setup>
const rail = ref(true)
const headerHeight = 64
</script>


<style scoped>
/* ---------- Main Layout ---------- */

.main-layout {
    min-height: 100vh;
    background: #ffffff;
    color: #000000;
}

/* ---------- Navigation Drawer ---------- */

.main-drawer {
    background: #fff5f5 !important;
    color: #000000 !important;
    border-right: 1px solid #ef9a9a !important;
}

/* Profile section */
.main-drawer :deep(.v-list-item) {
    color: #000000;
}

.main-drawer :deep(.v-list-item__prepend .v-avatar) {
    border: 2px solid #c62828;
}

/* Drawer divider */
.main-drawer :deep(.v-divider) {
    border-color: #ef9a9a;
    opacity: 1;
}

/* Navigation icons */
.main-drawer :deep(.v-list-item .v-icon) {
    color: #c62828;
    opacity: 1;
}

/* Navigation text */
.main-drawer :deep(.v-list-item-title) {
    color: #000000;
    font-weight: 500;
}

/* Hover effect */
.main-drawer :deep(.v-list-item:hover) {
    background: #ffcdd2;
}

/* Selected navigation item */
.main-drawer :deep(.v-list-item--active) {
    background: #ffcdd2 !important;
    color: #b71c1c !important;
    border-left: 4px solid #c62828;
}

/* Selected item text */
.main-drawer :deep(.v-list-item--active .v-list-item-title) {
    color: #b71c1c !important;
    font-weight: 700;
}

/* Selected item icon */
.main-drawer :deep(.v-list-item--active .v-icon) {
    color: #b71c1c !important;
}

/* ---------- App Bar ---------- */

.v-app-bar.app-bar-gradient {
    background: linear-gradient(90deg,
            #b71c1c 0%,
            #c62828 55%,
            #ef5350 100%) !important;

    color: #ffffff !important;
    border-bottom: 1px solid #ef9a9a;
    box-shadow: 0 2px 12px rgba(198, 40, 40, 0.15);
}

/* App bar title and icons */
.v-app-bar.app-bar-gradient :deep(.v-icon),
.v-app-bar.app-bar-gradient :deep(.v-btn) {
    color: #ffffff !important;
}

/* ---------- Main Content ---------- */

.main-content {
    background: #ffffff;
    color: #000000;
    min-height: calc(100vh - 64px);
}

/* ---------- Responsive ---------- */

@media (max-width: 600px) {
    .v-app-bar.app-bar-gradient {
        padding-inline: 4px;
    }
}
</style>
