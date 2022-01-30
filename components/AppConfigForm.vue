<template>
  <div>
    <h2>Generic variables</h2>
    <v-container>
      <v-row>
        <v-col>
          <v-text-field
            v-model.number="data['duration']"
            type="number"
            label="Duration (Required)"
            :rules="[(v) => (v !== null && v !== undefined) || 'Required']"
            :disabled="formConfig.disabledDuration || false"
          />
        </v-col>
        <v-col>
          <v-text-field
            v-model.number="data['update_frequency']"
            type="number"
            label="Update frequency"
            :disabled="formConfig.disabledUpdateFrequency || false"
          />
        </v-col>
        <v-col>
          <v-select
            v-model="data['align']"
            :items="alignmentItems"
            label="Alignment"
            :disabled="formConfig.disabledAlignment || false"
          />
        </v-col>
        <v-btn icon color="green" @click="setDefault('align')">
          <v-icon>mdi-refresh</v-icon>
        </v-btn>
      </v-row>
    </v-container>
    <h2>App-specific variables</h2>
    <v-container>
      <v-col>
        <v-row v-for="(field, i) in fields" :key="i">
          <v-text-field
            v-if="field.type === 'text' && callVif(field.name)"
            v-model="data[field.name]"
            :label="field.label"
            :rules="field.rules"
          />
          <v-select
            v-else-if="field.type === 'select' && callVif(field.name)"
            v-model="data[field.name]"
            :label="field.label"
            :items="field.items"
            :rules="field.rules"
          />
          <v-checkbox
            v-else-if="field.type === 'boolean' && callVif(field.name)"
            v-model="data[field.name]"
            :label="field.label"
            :items="field.items"
          />
          <v-text-field
            v-else-if="field.type === 'number' && callVif(field.name)"
            v-model.number="data[field.name]"
            type="number"
            :label="field.label"
            :rules="field.rules"
          />
          <v-slider
            v-else-if="field.type === 'number-slider' && callVif(field.name)"
            v-model.number="data[field.name]"
            :label="field.label"
            :min="field.min || 0"
            :max="field.max || 5"
            thumb-label
          />
          <div v-else-if="field.type === 'map' && callVif(field.name)" style="height: 400px; width: 95%;" class="mb-12">
            <client-only>
              <label :for="'map' + i.toString()">{{ field.label }}</label>
              <l-map
                :id="'map' + i.toString()"
                ref="map"
                :zoom="7"
                :center="data[field.name]"
                @ready="setLocation($event, i, field.name)"
                @click="setLocation($event, i, field.name)"
              >
                <l-tile-layer url="http://{s}.tile.osm.org/{z}/{x}/{y}.png" />
                <l-marker :ref="'marker' + i.toString()" :lat-lng="data[field.name]" />
              </l-map>
            </client-only>
          </div>
          <v-btn v-if="callVif(field.name)" icon color="green" @click="setDefault(field.name)">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
        </v-row>
      </v-col>
    </v-container>
  </div>
</template>

<script>
export default {
  props: {
    formConfig: {
      type: Object,
      default: () => { return {} }
    },
    fields: {
      type: Array,
      default: () => []
    },
    value: {
      type: Object,
      default: () => {
      }
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
    this.data.duration = this.formConfig.defaultDuration

    for (const field of this.fields) {
      if (field.defaultValue !== null && field.defaultValue !== undefined) {
        this.$set(this.data, field.name, field.defaultValue)
      }
    }

    this.$emit('input', this.data)
  },
  mounted () {
    this.$nextTick(() => {
      if (this.$refs.map) {
        for (const m of this.$refs.map) {
          m.mapObject.addEventListener('resize', () => m.mapObject.invalidateSize())
        }
      }
    })
  },
  methods: {
    displayed () {
      this.$nextTick(() => {
        if (this.$refs.map) {
          for (const m of this.$refs.map) {
            m.mapObject.invalidateSize()
          }
        }
      })
    },
    setDefault (name) {
      for (const field of this.fields) {
        if (field.name === name) {
          this.$set(this.data, field.name, field.defaultValue)
          this.$emit('input', this.data)
          return
        }
      }

      // We don't have default value, let's delete it from data
      this.$set(this.data, name, undefined)
      this.$emit('input', this.data)
    },
    setLocation (e, index, name) {
      if (e.latlng) {
        this.$set(this.data, name, Object.values(e.latlng))
      } else {
        this.$set(this.data, name, Object.values(this.$refs['marker' + index.toString()][0].latLng))
      }

      this.$refs['marker' + index.toString()][0].setLatLng({ lat: this.data[name][0], lng: this.data[name][1] })
      this.$emit('input', this.data)
    },
    callVif (name) {
      for (const field of this.fields) {
        if (field.name === name) {
          if (!field.vif) {
            return true
          } else {
            return field.vif(this)
          }
        }
      }

      return true
    }
  }
}
</script>
