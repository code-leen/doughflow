<template>
  <UContainer class="h-dvh flex flex-col justify-center">
    <UCard class="text-center">
      <UContainer class="p-3">
        <h1 class="text-3xl font-bold text-wrap">When do you want sourdough? 🍞</h1>
        <UFormGroup label="Time">
          <UInput v-model="desiredTime" type="time" />
        </UFormGroup>
        <UCard class="min-h-32">
          <div v-if="startTime" class="text-sm">
            <div class="text-xl">
              Start at
              <div class="mt-5 text-2xl font-semibold">
                {{ startTime }}
              </div>
            </div>
            <p class="opacity-50 mt-5">Does not factor in cooling time</p>
            <ol class="li-decimal">
              <li>Starter Rise Time: {{ RISE_TIME }} hrs</li>
              <li>Stretch n' Folds: {{ FOLDS_TIME }} hrs</li>
              <li>Bulk Fermentation: {{ BULK_FERMENTATION_TIME }} hrs</li>
              <li>Proofing: {{ SIT_OUT_TIME }} hr + {{ PROOFING_TIME }} hrs in fridge </li>
              <li>Baking: {{ BAKING_TIME }} hrs</li>
            </ol>
          </div>
        </UCard>
      </UContainer>
    </UCard>
  </UContainer>
</template>


<script setup>
const desiredTime = ref('');
const startTime = ref('');
const startTimeWithCooling = ref('');
const RISE_TIME = 4;
const FOLDS_TIME = 2;
const BULK_FERMENTATION_TIME = 8; // todo: maybe allow user to select between 8-12 hrs
const SIT_OUT_TIME = 1;
const PROOFING_TIME = 6; // todo: maybe allow user to select between 6-8 hrs
const BAKING_TIME = 55;
const COOLING_TIME = 1;

watch(desiredTime, getStartTime);

function getStartTime() {
  const currentTimeDate = new Date();

  // get desired time in ms
  const desiredTimeInMs = convertToMs(desiredTime.value);

  // get desired time to date obj
  const desiredTimeDate = new Date(currentTimeDate);
  desiredTimeDate.setHours(desiredTimeInMs.hours);
  desiredTimeDate.setMinutes(desiredTimeInMs.minutes);

  // get total prep time in ms
  const totalPrepTimeInMs = (RISE_TIME + FOLDS_TIME + BULK_FERMENTATION_TIME + SIT_OUT_TIME + PROOFING_TIME + BAKING_TIME) * 60 * 60 * 1000;

  // find time to start 
  const timeToStartInMs = desiredTimeDate.getTime() - totalPrepTimeInMs;
  startTime.value = new Date(timeToStartInMs).toLocaleTimeString(navigator.language, { hour: '2-digit', minute: '2-digit' });
  startTimeWithCooling.value = new Date(timeToStartInMs + COOLING_TIME * 60 * 60 * 1000).toLocaleTimeString(navigator.language, { hour: '2-digit', minute: '2-digit' });
}

function convertToMs(time) {
  const [hours, minutes] = time.split(':');
  return { hours: parseInt(hours), minutes: parseInt(minutes) };
}

</script>
