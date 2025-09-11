<script setup lang="ts">
import { fetchSchema, mineDatasourceObsProps, SchemaFieldProperty } from '@/lib/DatasourceUtils'
import { onMounted, ref, watch } from 'vue'
import { useVisualizationStore } from '@/stores/visualizationstore'
import { useUIStore } from '@/stores/uistore'
import TimePicker from '@/components/menus/TimePicker.vue'
import { useStartEndTimeSync, usePlaybackModeSync } from '@/composables/DataSourceOptions'
import { Mode } from 'osh-js/source/core/datasource/Mode.js'
import DataSourceDropDown from './DataSourceDropDown.vue'
import LOBLinePicker from './LobLinePicker.vue'

const visualizationStore = useVisualizationStore()
const markerDS = ref<any>(null)
const selectedLocation = ref<SchemaFieldProperty | null>(null)
const selectedLOB = ref<SchemaFieldProperty | null>(null)
const selectedLobProps = ref<SchemaFieldProperty | null>(null)
const selectedOpacity = ref<SchemaFieldProperty | null>(null)
const obsProps = ref<{ 'definition': string, 'label': string }[]>([])
const dsSchema = ref<any>(null)
const uiStore = useUIStore()

const emit = defineEmits(['update:selectedLocation', 'update:selectedLOB', 'update:selectedLobProps', 'update:selectedOpacity'])

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

watch(selectedLobProps, (val) => {
  emit('update:selectedLobProps', val)
})

</script>

<template>
  <v-card>
    <DataSourceDropDown title="Location" v-model:selectedProperty="selectedLocation" />
    <DataSourceDropDown title="Line of Bearing" v-model:selectedProperty="selectedLOB" />
    <LOBLinePicker title="Select Color and Opacity for LOB" v-model:lobLineProperties="selectedLobProps" />
    <TimePicker title="Start Time" v-model:timeInstant="startTime" />
    <TimePicker title="End Time" v-model:timeInstant="endTime" />
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