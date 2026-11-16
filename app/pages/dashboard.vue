
<template>
    <div class="dashboard-page">
        <v-container class="py-6 pb-12" style="max-width: 1100px">
            <!-- Welcome -->
            <section class="welcome pa-6 pa-md-8 mb-6">
                <div class="welcome-text">
                    <div class="text-body-2 welcome-date">{{ today }}</div>
                    <h1 class="welcome-title">{{ greeting }}</h1>
                    <p class="welcome-copy">
                        Welcome to System Integration, your workspace for scanning QR codes and
                        checking live weather in one place. Choose a tool below to get started.
                    </p>
                </div>
                <v-icon icon="mdi-view-dashboard-outline" size="120" class="welcome-icon" />
            </section>

            <!-- Tools -->
            <h2 class="text-subtitle-1 font-weight-bold mb-3 section-title">Your tools</h2>

            <v-row>
                <v-col v-for="tool in tools" :key="tool.title" cols="12" md="6">
                    <v-card :to="tool.to" variant="outlined" class="tool-card pa-5 h-100">
                        <div class="d-flex align-center ga-4 mb-3">
                            <v-avatar size="52" class="tool-avatar">
                                <v-icon :icon="tool.icon" size="28" />
                            </v-avatar>
                            <div class="text-h6 font-weight-bold">{{ tool.title }}</div>
                        </div>

                        <p class="tool-copy mb-4">{{ tool.description }}</p>

                        <div class="tool-action">
                            {{ tool.action }}
                            <v-icon icon="mdi-arrow-right" size="18" class="ml-1" />
                        </div>
                    </v-card>
                </v-col>
            </v-row>
        </v-container>
    </div>
</template>

<script setup>
// Replace this with the name of the signed-in Google account
const userName = ref('')

const firstName = computed(() => userName.value.trim().split(' ')[0])

const greeting = computed(() => {
    const hour = new Date().getHours()
    const part = hour < 12 ? 'Good morning' : hour < 18 ? 'Good afternoon' : 'Good evening'
    return firstName.value ? `${part}, ${firstName.value}` : part
})

const today = computed(() =>
    new Date().toLocaleDateString([], { weekday: 'long', month: 'long', day: 'numeric' })
)

const tools = [
    {
        title: 'QR Scanner',
        icon: 'mdi-qrcode-scan',
        description:
            'Scan a QR code with your camera. Start and stop the scanner whenever you need, and see the result as soon as a code is read.',
        action: 'Open scanner',
        to: '/qr_scanner',
    },
    {
        title: 'Weather App',
        icon: 'mdi-weather-partly-cloudy',
        description:
            'Check live conditions, the next 12 hours, and a 7-day forecast for your location or any city you search for.',
        action: 'View forecast',
        to: '/weather_app',
    },
]
</script>

<style>
/* White and red theme */
.dashboard-page {
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

.v-main:has(.dashboard-page) {
    background: #ffffff;
}

/* Welcome banner */
.dashboard-page .welcome {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 24px;
    border-radius: 12px;
    border: 1px solid #c62828;
    background: linear-gradient(115deg, #ffcdd2, #ef9a9a, #e57373);
    box-shadow: 0 5px 20px rgba(198, 40, 40, 0.12);
    color: #000000;
}

.welcome-text {
    max-width: 620px;
}

.welcome-date {
    opacity: 0.85;
    color: #000000;
}

.welcome-title {
    font-size: 2.4rem;
    font-weight: 600;
    line-height: 1.2;
    margin: 6px 0 12px;
    color: #000000;
}

.welcome-copy {
    font-size: 1.05rem;
    line-height: 1.6;
    margin: 0;
    color: #000000;
    opacity: 0.9;
}

.dashboard-page .welcome-icon {
    color: #b71c1c;
}

/* Section heading */
.dashboard-page .section-title {
    color: #000000;
}

/* Tool cards */
.dashboard-page .tool-card {
    background: #fff5f5;
    color: #000000;
    border: 1px solid #e57373;
    border-radius: 12px;
    transition: border-color 0.2s, background 0.2s;
}

.dashboard-page .tool-card:hover {
    background: #ffebee;
    border-color: #c62828;
}

/* Tool icons */
.dashboard-page .tool-avatar {
    background: #ffcdd2;
    border: 1px solid #e57373;
}

.dashboard-page .tool-avatar .v-icon {
    color: #b71c1c;
}

/* Tool descriptions */
.tool-copy {
    line-height: 1.6;
    color: #000000;
}

/* Tool action */
.tool-action {
    display: inline-flex;
    align-items: center;
    font-weight: 600;
    color: #c62828;
}

@media (max-width: 600px) {
    .dashboard-page .welcome {
        flex-direction: column;
        align-items: flex-start;
    }

    .welcome-title {
        font-size: 1.8rem;
    }

    .dashboard-page .welcome-icon {
        display: none;
    }
}
</style>
