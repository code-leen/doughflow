<template>
    <UContainer class="relative flex flex-col justify-center">
        <UCard class="text-center">
            <UContainer class="p-3">
                <h1 class="text-3xl font-bold text-wrap pb-4">Starting Now! 🍞</h1>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                    <UCard class="p-4">
                        <h3 class="font-semibold mb-2">Bulk Fermentation Time</h3>
                        <USelect v-model="bulkFermentationTime" :options="[8, 9, 10, 11, 12]" option-attribute="value"
                            placeholder="Select hours" class="mb-2" />
                        <p class="text-sm opacity-75">Recommended: 8-12 hours</p>
                    </UCard>

                    <UCard class="p-4">
                        <h3 class="font-semibold mb-2">Proofing Time</h3>
                        <USelect v-model="proofingTime" :options="[6, 7, 8]" option-attribute="value"
                            placeholder="Select hours" class="mb-2" />
                        <p class="text-sm opacity-75">Recommended: 6-8 hours</p>
                    </UCard>
                </div>

                <UCard class="min-h-32">
                    <div class="text-sm">
                        <div class="text-xl">
                            Your sourdough will be ready at
                            <div class="mt-5 text-2xl font-semibold">
                                {{ formatTime(endTime) }}
                            </div>
                        </div>
                        <p class="opacity-50 mt-5">Add {{ COOLING_TIME }} hour for cooling time</p>
                        <ol class="li-decimal text-center mt-4 space-y-2">
                            <li>
                                <span class="font-medium">Starter Rise Time:</span> {{ RISE_TIME }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(starterRiseTime) }}</div>
                            </li>
                            <li>
                                <span class="font-medium">Stretch n' Folds:</span> {{ FOLDS_TIME }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(stretchAndFoldsTime) }}</div>
                            </li>
                            <li>
                                <span class="font-medium">Bulk Fermentation:</span> {{ bulkFermentationTime }} hrs
                                <div class="text-sm opacity-75">{{ formatTime(bulkFermentationStartTime) }}</div>
                            </li>
                            <li>
                                <span class="font-medium">Proofing:</span> {{ SIT_OUT_TIME }} hr + {{ proofingTime }}
                                hrs in fridge
                                <div class="text-sm opacity-75">
                                    Room temp: {{ formatTime(sitOutTime) }}<br>
                                    Fridge: {{ formatTime(proofingStartTime) }}
                                </div>
                            </li>
                            <li>
                                <span class="font-medium">Baking:</span> {{ BAKING_MINUTES }} minutes
                                <div class="text-sm opacity-75">{{ formatTime(bakingStartTime) }}</div>
                            </li>
                        </ol>
                    </div>
                </UCard>
            </UContainer>
        </UCard>
    </UContainer>
</template>

<script setup>
const endTime = ref('');
const bulkFermentationTime = ref(8);
const proofingTime = ref(6);

const starterRiseTime = ref('');
const stretchAndFoldsTime = ref('');
const bulkFermentationStartTime = ref('');
const sitOutTime = ref('');
const proofingStartTime = ref('');
const bakingStartTime = ref('');

const RISE_TIME = 4;
const FOLDS_TIME = 2;
const SIT_OUT_TIME = 1;
const BAKING_MINUTES = 55;
const COOLING_TIME = 1;

watch([bulkFermentationTime, proofingTime], calculateTimes, { immediate: true });

function calculateTimes() {
    const currentTime = new Date();

    starterRiseTime.value = new Date(currentTime.getTime());

    const afterRise = new Date(currentTime.getTime() + (RISE_TIME * 60 * 60 * 1000));
    stretchAndFoldsTime.value = afterRise;

    const afterFolds = new Date(afterRise.getTime() + (FOLDS_TIME * 60 * 60 * 1000));
    bulkFermentationStartTime.value = afterFolds;

    const afterBulk = new Date(afterFolds.getTime() + (bulkFermentationTime.value * 60 * 60 * 1000));
    sitOutTime.value = afterBulk;

    const afterSitOut = new Date(afterBulk.getTime() + (SIT_OUT_TIME * 60 * 60 * 1000));
    proofingStartTime.value = afterSitOut;


    const afterProofing = new Date(afterSitOut.getTime() + (proofingTime.value * 60 * 60 * 1000));
    bakingStartTime.value = afterProofing;


    endTime.value = new Date(afterProofing.getTime() + (BAKING_MINUTES * 60 * 1000));
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
</script>