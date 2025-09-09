<script setup lang="ts">
import { fetchSchema, mineDatasourceObsProps, SchemaFieldProperty } from '@/lib/DatasourceUtils'
import { onMounted, ref, watch } from 'vue'
import { useVisualizationStore } from '@/stores/visualizationstore'
import { useUIStore } from '@/stores/uistore'
import TimePicker from '@/components/menus/TimePicker.vue'
import { useStartEndTimeSync, usePlaybackModeSync } from '@/composables/DataSourceOptions'
import { Mode } from 'osh-js/source/core/datasource/Mode.js'
import DataSourceSelector from './DataSourceDropDown.vue'

const visualizationStore = useVisualizationStore()
const markerDS = ref<any>(null)
const selectedLocation = ref<SchemaFieldProperty | null>(null)
const selectedLOB = ref<SchemaFieldProperty | null>(null)
const selectedColor = ref<SchemaFieldProperty | null>(null)
const selectedOpacity = ref<SchemaFieldProperty | null>(null)
const obsProps = ref<{ 'definition': string, 'label': string }[]>([])
const dsSchema = ref<any>(null)
const uiStore = useUIStore()

const emit = defineEmits(['update:selectedLocation', 'update:selectedLOB', 'update:selectedColor', 'update:selectedOpacity'])

const startTime = ref<string | null>(null)
const endTime = ref<string | null>(null)
const playbackMode = ref(Mode.REPLAY)
const playbackModes = Object.entries(Mode).map(([key, value]) => ({
  label: key.replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase()),
  value
}))

useStartEndTimeSync(startTime, endTime, visualizationStore)
usePlaybackModeSync(playbackMode, visualizationStore)

async function fetchProps() {
  const { ds, observedProps } = mineDatasourceObsProps()
  markerDS.value = ds
  obsProps.value = observedProps

  const schema = await fetchSchema(ds.datastream)
  dsSchema.value = schema
}

onMounted(async () => {
  fetchProps()
})

watch(selectedLocation, (val) => {
  emit('update:selectedLocation', val)
})

watch(selectedLOB, (val) => {
  emit('update:selectedLOB', val)
})

watch(selectedColor, (val) => {
  emit('update:selectedColor', val)
})

// watch(selectedOpacity, (val) => {
//   emit('update:selectedOpacity', val)
// })

</script>

<template>
  <v-card>
    <DataSourceSelector title="Location" v-model:selectedProperty="selectedLocation" />
    <DataSourceSelector title="Line of Bearing" v-model:selectedProperty="selectedLOB" />
    <v-card class="pa-2">
      <v-card-title>Select Color</v-card-title>
      <v-container>
        <v-color-picker 
          v-model="selectedColor"
          mode="rgb"          
        >
        </v-color-picker>
      </v-container>
    </v-card>
    <TimePicker title="Start Time" v-model:formattedDate="startTime" />
    <TimePicker title="End Time" v-model:formattedDate="endTime" />
    

    <v-combobox
      v-model="playbackMode"
      :items="playbackModes"
      item-title="label"
      item-value="value"
      label="Playback Mode"
      variant="solo"
      density="compact">
    </v-combobox>
  </v-card>
</template>

<style scoped>

</style>