<template>
  <div>
    <h2>Generic variables</h2>
    <v-container>
      <v-row>
        <v-col>
          <v-text-field v-model.number="data['duration']" type="number" label="Duration (Required)" :rules="[(v) => (v !== null && v !== undefined) || 'Required']" />
        </v-col>
        <v-col>
          <v-text-field v-model.number="data['update_frequency']" type="number" label="Update frequency" />
        </v-col>
        <v-col>
          <v-select v-model="data['align']" :items="alignmentItems" label="Alignment" />
        </v-col>
      </v-row>
    </v-container>
    <h2>App-specific variables</h2>
    <v-container>
      <v-row v-for="(field, i) in fields" :key="i">
        <v-col>
          <v-text-field v-if="field.type === 'text'" v-model="data[field.name]" :label="field.label" :rules="field.rules" />
          <v-select v-else-if="field.type === 'select'" v-model="data[field.name]" :label="field.label" :items="field.items" :rules="field.rules" />
          <v-checkbox v-else-if="field.type === 'boolean'" v-model="data[field.name]" :label="field.label" :items="field.items" />
          <v-text-field v-else-if="field.type === 'number'" v-model.number="data[field.name]" type="number" :label="field.label" :rules="field.rules" />
          <v-slider
            v-else-if="field.type === 'number-slider'"
            v-model="data[field.name]"
            :label="field.label"
            :min="field.min || 0"
            :max="field.max || 5"
            thumb-label
          />
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>
export default {
  props: {
    fields: {
      type: Array,
      default: () => []
    },
    value: {
      type: Object,
      default: () => {}
    }
  },
  data () {
    return {
      alignmentItems: [
        { text: 'Left', value: 'left' },
        { text: 'Center', value: 'center' },
        { text: 'Right', value: 'right' }
      ],
      data: {}
    }
  },
  watch: {
    data (val) {
      this.$emit('input', this.data)
    }
  },
  created () {
    for (const field of Object.values(this.fields)) {
      if (field.defaultValue !== null && field.defaultValue !== undefined) {
        this.data[field.name] = field.defaultValue
      }
    }

    if (this.value) {
      this.data = this.value
    }

    this.$emit('input', this.data)
  }
}
</script>
