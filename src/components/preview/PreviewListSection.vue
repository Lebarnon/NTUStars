<template>
  <div class="col-12 col-md-10 q-pa-md">
    <div class="text-h5">Indexes:</div>
    <div class="text-overline" v-if="timetableStore.previewIndexes && timetableStore.totalIndexCount > 0"> 
      Total Indexes: {{ timetableStore.totalIndexCount }} | 
      Showing {{ timetableStore.showingPreviewIndexCount }}
    </div>
    <div class="text-overline" v-else> 
      Click on a class in the timetable to see other possible indexes.
    </div>
    <div class="text-h6">All:</div>
    <div class="flex flex-start">
      <div class="q-gutter-xs">
        <q-chip 
          v-for="[index, value] of previewIndexes"
          :disable="showingIndex == index"
          @update:selected="(nextState) => onChipClicked(timetableStore.isIndexPreviewSaved(index), index)"
          :icon="timetableStore.isIndexPreviewSaved(index) ? 'cancel' : 'add_circle'"
          :selected="false"
          :key="index"
          color="primary" 
          text-color="white"
          size="small">
          {{index}}
        </q-chip>
      </div>
    <div class="text-h6 q-mt-sm" style="width: 100%;">Saved:</div>
    <div class="flex flex-start">
      <div class="q-gutter-xs">
        <q-chip 
          v-for="[index, value] of timetableStore.getSavedPreviewIndexes"
          :disable="showingIndex == index"
          @update:selected="(nextState) => onChipClicked(timetableStore.isIndexPreviewSaved(index), index, !(previewIndexesDict.has(index)))"
          :icon="timetableStore.isIndexPreviewSaved(index) ? 'cancel' : 'add_circle'"
          :selected="false"
          :key="index"
          color="primary" 
          text-color="white"
          size="small">
          {{index}}
        </q-chip>
      </div>
    </div>
   
      <div class="flex flex-center q-mt-md" style="width: 100%;">
        <q-pagination
          :text-color="settingsStore.darkMode ? 'white': 'primary'"
          v-model="currentPage"
          :max="numberOfPages"
          :max-pages="5"
          boundary-numbers
          direction-links
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { useTimetableStore } from '@/stores/timetable.js'
import { useSettingsStore } from '@/stores/settings.js'
import { computed, ref, watch } from 'vue'


const settingsStore = useSettingsStore()
const timetableStore = useTimetableStore()
const MAX_PER_PAGE = computed(() => timetableStore.maxShowingIndex)
const numberOfPages = computed(() => Math.ceil(timetableStore.getPreviewIndexes.length / MAX_PER_PAGE.value))
const currentPage = ref(1)
const previewCourseCode = computed(() => timetableStore.getPreviewCourseCode)
const showingIndex = computed(() => timetableStore.getShowingIndex(previewCourseCode.value))
const previewIndexes = computed(() => 
timetableStore.getPreviewIndexes.slice(
  (currentPage.value-1) * MAX_PER_PAGE.value,
  currentPage.value * MAX_PER_PAGE.value
  ))
const previewIndexesDict = ref(null)
  
watch(() => [currentPage.value, previewCourseCode.value], async ([newPage, newCourseCode], [oldPage, oldCourseCode]) => {
  console.log(newCourseCode, oldCourseCode)
  if(!newCourseCode && !oldCourseCode){
    return
  }
  const newIndexes = timetableStore.getPreviewIndexes.slice(
    (newPage-1) * MAX_PER_PAGE.value,
    newPage * MAX_PER_PAGE.value
  )
  previewIndexesDict.value = new Set(newIndexes.map(el => el[0]))
  
  
  if(newCourseCode != oldCourseCode){
    currentPage.value = 1
    return
  }
  const oldIndexes = timetableStore.getPreviewIndexes.slice(
    (oldPage-1) * MAX_PER_PAGE.value,
    oldPage * MAX_PER_PAGE.value
  )
    
  timetableStore.removeTempPreviews(oldIndexes.reduce((filtered, el) => {
    // not saved or current selected
    if (el[0] != showingIndex.value && !timetableStore.isIndexPreviewSaved(el[0])){
      filtered.push(el[0])
    }
    return filtered
  }, []))
  
  await timetableStore.addTempPreviews(newIndexes.reduce((filtered, el) => {
    // not saved or current selected
    if (el[0] != showingIndex.value && !timetableStore.isIndexPreviewSaved(el[0])){
      filtered.push(el[0])
    }
    return filtered
  }, []))
})

function onChipClicked(nextState, index, flag){
  console.log("flag", flag)
  if(!nextState){
    // add preview to saved preview indexes
    timetableStore.addToSavePreview(index)
  }else{
    // remove preview from saved preview indexes
    timetableStore.removeFromSavePreview(index, flag)
  }
}

function splice(object, lowerLimit, upperLimit) {
  var newObj = {};

  if (!upperLimit) { upperLimit = 1 }
  Object.entries(object).forEach(function([key, item], index) {

    if (index >= lowerLimit && index < upperLimit) {
      newObj[key] = item
    }
  })
  return newObj;
}
</script>
