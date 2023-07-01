<template>
  <v-container class="w-25">
    <!-- The dropdowns can be vertically dragged i.e. rearranged -->
    <draggable v-model="draggableItems" :key="draggableItems.length" @update:modelValue="handleDragChange">
      <template v-slot:item="{item}">
        <div 
          class="dropdown-row d-flex align-center"
        >
          <div class="drag-icon pt-5">
            <v-icon class="text-h3" icon="mdi-drag-vertical" />
          </div>
          <v-autocomplete
            v-model="item.value"
            @change="handleAutocompleteChange(item)"
            item-text="title"
            item-value="value"
            :items="enableDropdownItems"
          ></v-autocomplete>
          <v-btn
            class="remove-btn"
            variant="text"
            :disabled="disableAllDeleteBtns || item.disableDelete || minLimitReached"
            @click="removeDropdown(item.id)"
          >
            <v-icon class="text-h4" icon="mdi-close" color="red" />
          </v-btn>
        </div>
        <!-- Give a warning when keys (i.e. values) are used that are not in the standard objects list of
        possible keys. -->
        <div v-if="!valueIsValid(item.value)" class="text-caption text-warning">
          Warning: There is no matching value in the dropdown list.
        </div>
      </template>
    </draggable>
    <v-btn 
      class="mt-4"
      variant="outlined"
      color="blue"
      @click="addDropdown"
      :disabled="disableAdd || maxLimitReached"
    >
      <v-icon icon="mdi-plus" color="blue" />
      Add
    </v-btn>
  </v-container>
</template>

<script setup lang="ts">
  import { ref, computed, onMounted, watch } from "vue"
  import { v4 as uuidv4 } from "uuid"
  import Draggable from "vue3-draggable"
  import { IDropdown } from "../App.vue"

  interface IDropdownItem{
    title: string
    value: string
  }

  // The dropdowns are dynamically added on creation, if defined in the props.
  // Make a property disableDelete (default false) to enable/disable deletion of a dropdown
  // Make a property disableAdd (default false) to disable adding (the +)
  const props = defineProps({
    modelValue: {
      type: Array as () => IDropdown[],
      default: []
    },
    selectedValues: {
      type: Array as () => string[],
      default: []
    },
    disableAllDeleteBtns: {
      type: Boolean,
      default: false
    },
    disableAdd: {
      type: Boolean,
      default: false
    },
    minDropdowns: {
      type: Number || null,
      default: null
    },
    maxDropdowns: {
      type: Number || null,
      default: null
    }
  })

  const emits = defineEmits(['update:modelValue'])

  // The standard object list with the fields key and text (value / title) for all dropdowns can be
  // set in the component interface with data list property. To this is the same for all dropdowns.
  const dropdownItems: IDropdownItem[] = [
    { title: 'ID', value: 'ID' },
    { title: 'Title', value: 'Title' },
    { title: 'Assigned to', value: 'Assigned to' },
    { title: 'State', value: 'State' },
    { title: 'Iteration Path', value: 'Iteration Path' },
    { title: 'Original Estimate', value: 'Original Estimate' },
    { title: 'Tags', value: 'Tags' },
    { title: 'Comment Count', value: 'Comment Count' },
    { title: 'Activity Date', value: 'Activity Date' },
  ];
  const draggableItems = ref(props.modelValue)

  // Give a console warning when the initial data is not conform minimum or maximum when set
  onMounted(() => {
    if (
      (props.minDropdowns && props.minDropdowns >  draggableItems.value.length) ||
      (props.maxDropdowns && props.maxDropdowns <  draggableItems.value.length)
    ) {
      console.warn('The initial data is not conform minimum or maximum')
    }
  });

  // On initial creation of the dropdowns filter out automatically all the selected options existing
  // in the other dropdowns to prevent selecting double values.
  const enableDropdownItems = computed(
    () => dropdownItems.filter((item) => !props.selectedValues.includes(item.value))
  );

  // Make properties in the component interface to set minimal dropdowns and maximum
  // dropdown and check this by hiding delete / add
  const minLimitReached = computed((): boolean => {
    return !!(props.minDropdowns && props.minDropdowns >=  draggableItems.value.length)
  })
  const maxLimitReached = computed((): boolean => {
    return !!(props.maxDropdowns && props.maxDropdowns <=  draggableItems.value.length)
  })

  // Give a console warning if minimum is set and disableDelete is true
  watch(minLimitReached, (value) => {
    if (value) console.warn('Minimum dropdowns limit reached')
  });

  // Give a console warning if maximum is set and disableAdd is true
  watch(maxLimitReached, (value) => {
    if (value) console.warn('Maximum dropdowns limit reached')
  });

  //A user can add an extra dropdown with the [+] at the bottom.
  const addDropdown = (): void => {
    const newItem: IDropdown = { id: uuidv4(), value: '' }
    draggableItems.value.push(newItem)
    emits('update:modelValue', draggableItems.value)
  }

  //A dropdown can also be deleted
  const removeDropdown = (id: number): void => {
    const newItems: IDropdown[] = draggableItems.value.filter((el: IDropdown) => el.id !== id)
    draggableItems.value = newItems
    emits('update:modelValue', newItems)
  }

  const handleAutocompleteChange = (item: IDropdownItem): void => {
    const index: number = draggableItems.value.findIndex((el: IDropdown) => el.value === item.value)

    if (index !== -1) {
      draggableItems.value[index].value = item.value
      emits('update:modelValue', draggableItems.value)
    }
  }

  //The dropdowns can be vertically dragged i.e. rearranged
  const handleDragChange = (): void => {
    emits('update:modelValue', draggableItems.value)
  }

  const valueIsValid = (value: string): boolean => {
    return !value || dropdownItems.some((item: IDropdownItem) => item.value === value)
  }
</script>

<style scoped>
  .dropdown-row {
    padding-bottom: 15px;
    padding-left: 48px;
    padding-right: 10px;
  }

  .drag-icon {
    display: none;
    cursor: grab;
  }

  .remove-btn {
    height: 50px !important;
    margin-top: 5px;
  }

  .dropdown-row:hover {
    background: #f3f3f3;
    border-radius: 5px;
    padding-left: 0
  }

  .dropdown-row:hover .drag-icon {
    display: block;
  }

  .input-item {
    cursor: pointer;
  }
</style>
