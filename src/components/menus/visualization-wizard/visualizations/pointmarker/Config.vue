<script setup lang="ts">
// import { fetchSchema } from '@/lib/DatasourceUtils';
import DataStreamFilter from 'osh-js/source/core/sweapi/datastream/DataStreamFilter'
import { OSHDatastream } from '@/lib/OSHConnectDataStructs';
import { useVizWizStore } from '@/stores/vizwizstore';
import { debug } from 'console';
import { ref, computed, reactive, watch } from 'vue';
import { debugPort } from 'process';
import { useDataStreamStore } from '@/stores/datastreamstore';

// Retrieve datastreams
const dataStreamStore = useVizWizStore()
// const listDatastreams = computed(() => dataStreamStore.dataStreams)
const listDatastreams = computed(() => dataStreamStore.datastreams)

console.log(listDatastreams)


// LOCATION
const hasLocation = ref(false)
// Destructure all values for location
const {
  selectedDSId: selectedLocationDsId, 
  selectedDS: selectedLocationDs,
  schema: locationPropertySchema,
  selectedPropertyName: selectedLocationPropertyName,
  listProperties: listLocationProperties,
  selectedProperty: selectedLocationProperty
} = useDatastreamPropertySelector(listDatastreams)

// ORIENTATION
const hasOrientation = ref(false)
// Destructure all values for Orientation
const {
  selectedDSId: selectedOrientationDsId, 
  selectedDS: selectedOrientationDs,
  schema: orientationPropertySchema,
  selectedPropertyName: selectedOrientationPropertyName,
  listProperties: listOrientationProperties,
  selectedProperty: selectedOrientationProperty
} = useDatastreamPropertySelector(listDatastreams)

// MARKER ID
const hasMarkerId = ref(false)
// Destructure all values for MarkerId
const {
  selectedDSId: selectedMarkerIdDsId, 
  selectedDS: selectedMarkerIdDs,
  schema: markerIdPropertySchema,
  selectedPropertyName: selectedMarkerIdPropertyName,
  listProperties: listMarkerIdProperties,
  selectedProperty: selectedMarkerIdProperty
} = useDatastreamPropertySelector(listDatastreams)


//HELPER FUNCTIONS
function fetchSchema(datastream: any) {
    if (!datastream || !datastream.properties) {
    console.warn("[DatasourceUtils] Invalid datastream passed to fetchSchema:", datastream)
    return null
  }

  console.log('[DatasourceUtils] Fetching schema for datastream:', datastream)
  // debugger
  let checkedFormat = datastream.properties.formats.filter(
    (format: any) =>
      format.includes('application/swe+json') || format.includes('application/swe+binary'),
  )

  

  if (!checkedFormat) {
    checkedFormat = ['application/om+json'] // Fallback to om+json which should be available always
  }

  let filter = new DataStreamFilter({ obsFormat: checkedFormat[0] })
  return datastream
    .getSchema(filter)
    .then((schemaRes: any) => {
      if (schemaRes) {
        console.log('[DatasourceUtils] Schema fetched:', schemaRes)
        return schemaRes
      }
    })
    .catch((error: any) => {
      console.error('[DatasourceUtils] Error fetching schema:', error)
      return null
    })
}

function useDatastreamPropertySelector(listDatastreams: any) {
  // Create a reference for the selected datastream ID in the form
  const selectedDSId = ref<string | null>(null)
  
  // Compute the actual datastream using the datastream id thats selected in the first drop down
  const selectedDS = computed(() =>
    listDatastreams.value.find((ds: any) => ds.id === selectedDSId.value) || null
  )

  // Create a reference for the selected Property Name
  const selectedPropertyName = ref<string | null>(null)

  // Create a variable for a datastreams schema to create a list of field properties
  const schema = ref<any | null>(null)

  // Update the schema based on the selected datastream
  watch(selectedDS, async (ds) => {
    schema.value = null
    selectedPropertyName.value = null
    if (!ds.datastream) return
    schema.value = await fetchSchema(ds.datastream)
  })

  // Compute the available properties based on the schema
  const listProperties = computed(() => {
    if (!schema.value || !schema.value.recordSchema?.fields) return []
    // Ensure it's really an array
    return Array.isArray(schema.value.recordSchema.fields)
      ? schema.value.recordSchema.fields
      : []
  })

  // Create a varible for the selected property
  const selectedProperty = computed(() =>
    listProperties.value.find((p: any) => p.name === selectedPropertyName.value) || null
  )

  return {
    selectedDSId,
    selectedDS,
    schema,
    selectedPropertyName,
    listProperties,
    selectedProperty,
  }
}

</script>

<template>
  <!-- SELECT POINTMARKER PROPERTIES -->
  <v-checkbox v-model="hasLocation" label="Location" value="location"></v-checkbox>
  <v-select
    v-if="hasLocation"
    v-model="selectedLocationDsId"
    :items="listDatastreams"  
    item-title="name"
    item-value="id"
    label="Select DataStream"
  />
  <v-select
    v-if="hasLocation"
    v-model="selectedLocationPropertyName"
    :items="listLocationProperties"  
    item-title="label"
    item-value="name"
    label="Select Property"
  />
  <v-checkbox v-model="hasOrientation" label="Orientation" value="orientation"></v-checkbox>
    <v-select
    v-if="hasOrientation"
    v-model="selectedOrientationDsId"
    :items="listDatastreams"  
    item-title="name"
    item-value="id"
    label="Select DataStream"
  />
  <v-select
    v-if="hasOrientation"
    v-model="selectedOrientationPropertyName"
    :items="listOrientationProperties"  
    item-title="label"
    item-value="name"
    label="Select Property"
  />
  <v-checkbox v-model="hasMarkerId" label="Marker Id" value="markerId"></v-checkbox>
  <v-select
    v-if="hasMarkerId"
    v-model="selectedMarkerIdDsId"
    :items="listDatastreams"  
    item-title="name"
    item-value="id"
    label="Select DataStream"
  />
  <v-select
    v-if="hasMarkerId"
    v-model="selectedMarkerIdPropertyName"
    :items="listMarkerIdProperties"  
    item-title="label"
    item-value="name"
    label="Select Property"
  />
</template>