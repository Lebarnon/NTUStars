<template>
  <div class="">
    <q-stepper
      v-model="step"
      ref="stepper"
      color="primary"
      animated
      flat
    >
      <q-step
        :name="1"
        :color="settingsStore.darkMode ? 'white': 'primary'"
        :title="title"
        :done="step > 1"
      >
        <SelectSemBar :is-error="!isSemSelected"/>
      </q-step>
      <q-step
        :name="2"
        :color="settingsStore.darkMode ? 'white': 'primary'"
        title=""
        :done="step > 2"
      >
        <div class="row justify-center q-mb-md">
          <SearchBar/>
        </div>
        <q-separator spaced />
        <SelectedList />
      </q-step>
      <template v-slot:navigation>
        <q-stepper-navigation>
          <q-btn v-if="step == 1" @click="handleStepIncrease" color="primary" label="Continue" />
          <q-btn v-if="step > 1" flat :color="settingsStore.darkMode ? 'white': 'primary'" @click="handleStepDecrease" label="Back" class="q-ml-sm" />
        </q-stepper-navigation>
      </template>
    </q-stepper>
  </div>
</template>

<script setup>
import { ref ,computed, onMounted } from 'vue'
import SearchBar from './SearchBar.vue';
import SelectedList from './SelectedList.vue';
import SelectSemBar from './SelectSemBar.vue';
import { useTimetableStore } from '@/stores/timetable';
import { useSettingsStore } from '@/stores/settings';
import { Dialog } from 'quasar';


const isSemSelected = ref(true)
const timetableStore = useTimetableStore()
const settingsStore = useSettingsStore()
const step = ref(1)
const stepper = ref(null)
const title = computed(() => timetableStore.getSemShortName)
onMounted(() => {
  if(timetableStore.getSemester != null){
    isSemSelected.value = true
    stepper.value.next()
  }
})

function handleStepIncrease() {
  if(timetableStore.getSemester != null){
    isSemSelected.value = true
    stepper.value.next()

  }else{
    isSemSelected.value = false
  }
}
function handleStepDecrease() {
  Dialog.create({
    title: 'Reset Timetable',
    message: 'Going back will reset your timetable.',
    cancel: true,
    persistent: true
  }).onOk(() => {
    timetableStore.reset()
    stepper.value.previous()
  })
}

</script>