<template>
    <UContainer class="relative h-dvh flex flex-col justify-center">
        <UCard class="text-center">
            <UContainer class="p-3">
                <h1 class="text-3xl font-bold text-wrap pb-4">When do you want sourdough? 🍞</h1>
                <div class="grid gap-4 max-w-sm mx-auto">
                    <UFormGroup label="Ready By">
                        <UInput v-model="desiredDate" type="date" :min="minDate" @update:model-value="updateMinTime" />
                    </UFormGroup>
                    <UFormGroup>
                        <div class="relative">
                            <USelect v-model="desiredTime" :options="availableTimeOptions" :disabled="!desiredDate"
                                placeholder="Select time" class="w-full" />
                        </div>
                    </UFormGroup>
                </div>
                <UCard class="min-h-32 mt-4">
                    <div v-if="isValidTime" class="text-sm">
                        <div class="text-xl">
                            Ready by
                            <div class="mt-5 text-2xl font-semibold">
                                {{ formatTime(endTimeDate) }}
                            </div>
                            <br>
                            Start at
                            <div class="mt-5 text-2xl font-semibold">
                                {{ formatTime(startTimeDate) }}
                            </div>
                        </div>
                        <p class="opacity-50 mt-5">Add {{ COOLING_TIME }} hour for cooling time</p>
                        <ol class="li-decimal text-center pl-8 mt-4 space-y-2">
                            <li>
                                <span class="font-medium">Starter Rise Time:</span> {{ RISE_TIME }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(starterRiseTime) }}</div>
                            </li>
                            <li>
                                <span class="font-medium">Stretch n' Folds:</span> {{ FOLDS_TIME }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(stretchAndFoldsTime) }}</div>
                            </li>
                            <li>
                                <span class="font-medium">Bulk Fermentation:</span> {{ BULK_FERMENTATION_TIME }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(bulkFermentationStartTime) }}</div>
                            </li>
                            <li>
                                <span class="font-medium">Proofing:</span> {{ SIT_OUT_TIME }} hr + {{ PROOFING_TIME }}
                                hrs in fridge
                                <div class="text-sm opacity-75">
                                    Room temp: {{ formatTime(sitOutTime) }}<br>
                                    Fridge: {{ formatTime(proofingStartTime) }}
                                </div>
                            </li>
                            <li>
                                <span class="font-medium">Baking:</span> {{ BAKING_TIME }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(bakingStartTime) }}</div>
                            </li>
                        </ol>
                    </div>
                    <div v-else class="text-sm p-4">
                        Select a date and time to see your baking schedule
                    </div>
                </UCard>
            </UContainer>
        </UCard>
    </UContainer>
</template>

<script setup>
const RISE_TIME = 4;
const FOLDS_TIME = 2;
const BULK_FERMENTATION_TIME = 8;
const SIT_OUT_TIME = 1;
const PROOFING_TIME = 6;
const BAKING_TIME = 0.92; // 55 minutes in hours
const COOLING_TIME = 1;

const totalPrepTimeHours = RISE_TIME + FOLDS_TIME + BULK_FERMENTATION_TIME + SIT_OUT_TIME + PROOFING_TIME + BAKING_TIME;

function isValidReadyTime(readyTime) {
    const startTime = new Date(readyTime.getTime() - totalPrepTimeHours * 60 * 60 * 1000);
    const now = new Date();
    return startTime > now;
}

function getFirstValidDate() {
    const now = new Date();
    let checkDate = new Date(now);
    checkDate.setHours(0, 0, 0, 0);

    for (let i = 0; i < 7; i++) {
        const testDate = new Date(checkDate);
        testDate.setDate(checkDate.getDate() + i);
        const testTime = new Date(testDate);
        testTime.setHours(23, 59, 59, 999);

        const startTime = new Date(testTime.getTime() - totalPrepTimeHours * 60 * 60 * 1000);
        if (startTime > now) {
            return testDate;
        }
    }

    const tomorrow = new Date(now);
    tomorrow.setDate(tomorrow.getDate() + 1);
    return tomorrow;
}

function createDateTime(dateStr, timeStr) {
    const [year, month, day] = dateStr.split('-').map(num => parseInt(num));
    const [hours, minutes] = timeStr.split(':').map(num => parseInt(num));
    const date = new Date(year, month - 1, day, hours, minutes, 0, 0);
    return date;
}

function roundToNearestThirtyMinutes(date) {
    const roundedDate = new Date(date);
    const minutes = roundedDate.getMinutes();
    const roundedMinutes = Math.round(minutes / 30) * 30;
    roundedDate.setMinutes(roundedMinutes, 0, 0);
    return roundedDate;
}

const desiredTime = ref('');
const desiredDate = ref('');
const startTimeDate = ref(null);
const endTimeDate = ref(null);

const starterRiseTime = ref('');
const stretchAndFoldsTime = ref('');
const bulkFermentationStartTime = ref('');
const sitOutTime = ref('');
const proofingStartTime = ref('');
const bakingStartTime = ref('');

const minDate = computed(() => {
    return formatDateForInput(getFirstValidDate());
});

const isToday = computed(() => {
    const selectedDate = new Date(desiredDate.value);
    const today = new Date();
    return selectedDate.toDateString() === today.toDateString();
});

const minTimeFormatted = computed(() => {
    if (!isToday.value) return '00:00';
    const now = new Date();
    const minTime = new Date(now.getTime() + totalPrepTimeHours * 60 * 60 * 1000);
    const hours = minTime.getHours();
    const minutes = minTime.getMinutes();
    return `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}`;
});

const availableTimeOptions = computed(() => {
    if (!desiredDate.value) return [];
    const options = [];

    for (let hour = 0; hour < 24; hour++) {
        for (let minute = 0; minute < 60; minute += 30) {
            const readyTime = createDateTime(
                desiredDate.value,
                `${String(hour).padStart(2, '0')}:${String(minute).padStart(2, '0')}`
            );

            if (!isValidReadyTime(readyTime)) continue;

            const formattedTime = readyTime.toLocaleTimeString([], {
                hour: '2-digit',
                minute: '2-digit',
                hour12: true
            });

            options.push({
                label: formattedTime,
                value: `${String(hour).padStart(2, '0')}:${String(minute).padStart(2, '0')}`
            });
        }
    }

    return options;
});

const isValidTime = computed(() => {
    if (!desiredTime.value || !desiredDate.value) return false;
    const readyTime = createDateTime(desiredDate.value, desiredTime.value);
    return isValidReadyTime(readyTime);
});

watch([desiredDate, desiredTime], calculateTimes);

function updateMinTime() {
    desiredTime.value = '';
}

function calculateTimes() {
    if (!isValidTime.value) {
        startTimeDate.value = null;
        endTimeDate.value = null;
        return;
    }

    const readyTime = createDateTime(desiredDate.value, desiredTime.value);
    const startTime = roundToNearestThirtyMinutes(
        new Date(readyTime.getTime() - totalPrepTimeHours * 60 * 60 * 1000)
    );

    startTimeDate.value = startTime;
    endTimeDate.value = readyTime;

    let currentTime = new Date(startTime);

    starterRiseTime.value = currentTime;
    currentTime = new Date(currentTime.getTime() + RISE_TIME * 60 * 60 * 1000);

    stretchAndFoldsTime.value = currentTime;
    currentTime = new Date(currentTime.getTime() + FOLDS_TIME * 60 * 60 * 1000);

    bulkFermentationStartTime.value = currentTime;
    currentTime = new Date(currentTime.getTime() + BULK_FERMENTATION_TIME * 60 * 60 * 1000);

    sitOutTime.value = currentTime;
    currentTime = new Date(currentTime.getTime() + SIT_OUT_TIME * 60 * 60 * 1000);

    proofingStartTime.value = currentTime;
    currentTime = new Date(currentTime.getTime() + PROOFING_TIME * 60 * 60 * 1000);

    bakingStartTime.value = currentTime;
}

function formatDateForInput(date) {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    return `${year}-${month}-${day}`;
}

function formatTimeOnly(date) {
    if (!date) return '';
    const locale = typeof navigator !== 'undefined' ? navigator.language : 'en-US';
    return date.toLocaleTimeString(locale, {
        hour: '2-digit',
        minute: '2-digit'
    });
}

function formatTime(date) {
    if (!(date instanceof Date)) return '';
    const locale = typeof navigator !== 'undefined' ? navigator.language : 'en-US';
    return date.toLocaleTimeString(locale, {
        hour: '2-digit',
        minute: '2-digit',
        weekday: 'short',
        month: 'short',
        day: 'numeric'
    });
}

onMounted(async () => {
    const validDate = getFirstValidDate();
    desiredDate.value = formatDateForInput(validDate);
    await nextTick();
    const timeOptions = availableTimeOptions.value;
    if (timeOptions.length > 0) {
        desiredTime.value = timeOptions[0].value;
    }
});
</script>