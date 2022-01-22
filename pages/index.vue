<template>
  <v-card min-height="800" min-width="1200" height="100%" fill-height>
    <v-container style="min-height: 800px;">
      <v-row style="min-height: 800px;">
        <v-flex xs3>
          <v-col style="max-height: 800px;" class="overflow-y-auto">
            <v-row>
              <v-col>
                <v-select v-model="selectedAppToBeAdded" :items="appItems" label="Add new app" @change="addNewApp(selectedAppToBeAdded)">
                  <template slot="item" slot-scope="data">
                    <v-row>
                      <v-icon class="ml-2 mr-4">
                        {{ data.item.icon }}
                      </v-icon> {{ data.item.text }}
                    </v-row>
                  </template>
                </v-select>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-list>
                  <v-list-item-group
                    v-model="selectedApp"
                    color="primary"
                  >
                    <v-list-item
                      v-for="appId in addedAppsIds"
                      :key="appId"
                    >
                      <v-list-item-icon>
                        <v-icon v-text="addedApps[appId].icon" />
                      </v-list-item-icon>
                      <v-list-item-content>
                        <v-list-item-title v-text="addedApps[appId].text" />
                      </v-list-item-content>
                      <v-list-item-action>
                        <v-flex>
                          <v-btn
                            v-show="appId !== addedAppsIds[0]"
                            small
                            color="accent"
                            icon
                            @click.stop="moveUp(appId)"
                          >
                            <v-icon>mdi-arrow-up-circle</v-icon>
                          </v-btn>
                          <v-btn
                            v-show="appId !== addedAppsIds[addedAppsIds.length - 1]"
                            small
                            color="accent"
                            icon
                            @click.stop="moveDown(appId)"
                          >
                            <v-icon>mdi-arrow-down-circle</v-icon>
                          </v-btn>
                          <v-btn
                            small
                            color="red"
                            icon
                            @click.stop="remove(appId)"
                          >
                            <v-icon>mdi-close-circle-outline</v-icon>
                          </v-btn>
                        </v-flex>
                      </v-list-item-action>
                    </v-list-item>
                  </v-list-item-group>
                </v-list>
              </v-col>
            </v-row>
          </v-col>
        </v-flex>
        <v-divider vertical />
        <v-flex>
          <v-col ref="appConfigForm">
            <span v-for="appIndex in addedAppsIds.length" :key="addedAppsIds[appIndex]">
              <keep-alive>
                <component
                  :is="appConfigFormComponent"
                  v-show="selectedApp !== null && selectedApp !== undefined && selectedApp === (appIndex - 1)"
                  v-if="Object.keys(addedApps)[appIndex] !== null && Object.keys(addedApps)[(appIndex - 1)] !== undefined"
                  :key="addedAppsIds[appIndex - 1]"
                  v-model="addedApps[Object.keys(addedApps)[(appIndex - 1)]].data"
                  :fields="apps[addedApps[Object.keys(addedApps)[(appIndex - 1)]].name].form"
                />
              </keep-alive>
            </span>
          </v-col>
        </v-flex>
      </v-row>
    </v-container>
    <v-divider />
    <v-card-actions>
      <v-col class="text-right" fill-width>
        <input ref="configFile" type="file" accept="application/json" style="display: none" @change="importConfig()">
        <v-btn color="secondary" @click="importConfigDialog()">
          Import config
        </v-btn>
        <a ref="downloadConfig" :href="exportConfigHref" :download="exportConfigFilename" style="display: none" />
        <v-btn color="primary" @click="exportConfig()">
          Export config
        </v-btn>
      </v-col>
    </v-card-actions>
  </v-card>
</template>

<script>

export default {
  name: 'IndexPage',
  data () {
    return {
      appConfigFormComponent: 'AppConfigForm',
      selectedAppToBeAdded: null,
      addedAppsIds: [],
      addedApps: {},
      selectedApp: null,
      exportConfigHref: null,
      exportConfigFilename: null,
      apps: {
        time: {
          label: 'Time',
          icon: 'mdi-clock',
          form: [{
            type: 'text',
            name: 'timezone',
            label: 'Timezone (Required)',
            rules: [v => !!v || 'Required']
          }, {
            type: 'boolean',
            name: 'show_seconds',
            label: 'Show seconds',
            defaultValue: false
          }]
        },
        crypto: {
          label: 'Crypto',
          icon: 'mdi-bitcoin',
          form: [{
            type: 'text',
            name: 'crypto',
            label: 'Crypto (Required)',
            rules: [v => !!v || 'Required']
          }, {
            type: 'text',
            name: 'base_currency',
            label: 'Base currency (Required)',
            rules: [v => !!v || 'Required']
          }, {
            type: 'number',
            name: 'decimals',
            label: 'Number of decimals'
          }, {
            type: 'text',
            name: 'thousands_separator',
            label: 'Thousands separator'
          }]
        },
        auto_contrast: {
          label: 'Auto Contrast',
          icon: 'mdi-brightness-6',
          form: [{
            type: 'text',
            name: 'latitude',
            label: 'Latitude (Required)',
            rules: [v => !!v || 'Required']
          }, {
            type: 'text',
            name: 'longitude',
            label: 'Longitude (Required)',
            rules: [v => !!v || 'Required']
          }, {
            type: 'number-slider',
            name: 'contrast_after_sunrise',
            label: 'Contrast after sunrise (Required)',
            rules: [v => !!v || 'Required'],
            max: 255,
            defaultValue: 200
          }, {
            type: 'number-slider',
            name: 'contrast_after_sunset',
            label: 'Contrast after sunset (Required)',
            rules: [v => !!v || 'Required'],
            max: 255,
            defaultValue: 0
          }]
        },
        blockheight: {
          label: 'Block height',
          icon: 'mdi-cube-outline',
          form: [{
            type: 'text',
            name: 'mempool_space_api',
            label: 'Mempool.space API endpoint'
          }]
        },
        halving: {
          label: 'Halving countdown',
          icon: 'mdi-fraction-one-half',
          form: [{
            type: 'text',
            name: 'mempool_space_api',
            label: 'Mempool.space API endpoint'
          }]
        },
        fees: {
          label: 'Bitcoin fees',
          icon: 'mdi-human-male-height',
          form: [{
            type: 'text',
            name: 'mempool_space_api',
            label: 'Mempool.space API endpoint'
          }]
        },
        text: {
          label: 'Text',
          icon: 'mdi-message-text',
          form: [{
            type: 'text',
            name: 'text',
            label: 'Your text (Required)',
            rules: [v => !!v || 'Required']
          }]
        },
        marketcap: {
          label: 'Market cap',
          icon: 'mdi-currency-usd',
          form: [{
            type: 'text',
            name: 'crypto',
            label: 'Crypto (Required)',
            rules: [v => !!v || 'Required']
          }, {
            type: 'text',
            name: 'base_currency',
            label: 'Base currency (Required)',
            rules: [v => !!v || 'Required']
          }]
        },
        moscow_time: {
          label: 'Moscow time',
          icon: 'mdi-mosque',
          form: []
        },
        difficulty: {
          label: 'Difficulty',
          icon: 'mdi-dumbbell',
          form: [{
            type: 'text',
            name: 'mempool_space_api',
            label: 'Mempool.space API endpoint'
          }]
        },
        temperature: {
          label: 'Temperature',
          icon: 'mdi-thermometer',
          form: [{
            type: 'text',
            name: 'city',
            label: 'City'
          }, {
            type: 'text',
            name: 'key',
            label: 'openweathermap.org API Key'
          }, {
            type: 'text',
            name: 'units',
            label: 'Units'
          }]
        },
        xpub: {
          label: 'XPub (Experimental)',
          icon: 'mdi-bitcoin',
          form: [{
            type: 'text',
            name: 'xpub',
            label: 'Your XPub'
          }, {
            type: 'number',
            name: 'limit',
            label: 'City'
          }, {
            type: 'number',
            name: 'offset',
            label: 'City'
          }, {
            type: 'number',
            name: 'step_addresses',
            label: 'Step addresses'
          }, {
            type: 'number',
            name: 'limit',
            label: 'End when unused'
          }, {
            type: 'text',
            name: 'btc_rpc_explorer_api',
            label: 'BTC RPC explorer API'
          }, {
            type: 'number',
            name: 'waittime',
            label: 'Waittime'
          }
          ]
        },
        lnbits_wallet_balance: {
          label: 'LNBits wallet balance (Exp.)',
          icon: 'mdi-flash',
          form: [{
            type: 'text',
            name: 'server',
            label: 'Server'
          }, {
            type: 'text',
            name: 'invoicereadkey',
            label: 'Invoice read key'
          }]
        }
      }
    }
  },
  computed: {
    appItems () {
      const result = []
      for (const [k, v] of Object.entries(this.apps)) {
        result.push({ text: v.label, value: k, icon: v.icon })
      }
      return result
    }
  },
  methods: {
    addNewApp (name) {
      this.$nextTick(() => {
        this.selectedAppToBeAdded = null
      })

      const id = Math.random().toString(16).slice(2)
      this.addedApps[id] = {
        id,
        name,
        text: this.apps[name].label,
        icon: this.apps[name].icon
      }
      this.addedAppsIds.push(id)

      return id
    },
    moveUp (appId) {
      const index = this.addedAppsIds.indexOf(appId)
      if (index !== -1) {
        this.addedAppsIds.splice(index, 1)
        this.addedAppsIds.splice(index - 1, 0, appId)
      }
    },
    moveDown (appId) {
      const index = this.addedAppsIds.indexOf(appId)
      if (index !== -1) {
        this.addedAppsIds.splice(index, 1)
        this.addedAppsIds.splice(index + 1, 0, appId)
      }
    },
    remove (appId) {
      const index = this.addedAppsIds.indexOf(appId)
      if (index !== -1) {
        this.addedAppsIds.splice(index, 1)
      }
      delete this.addedApps[appId]
    },
    importConfigDialog () {
      this.$refs.configFile.click()
    },
    importConfig () {
      const file = this.$refs.configFile.files[0]
      const reader = new FileReader()
      reader.onload = (res) => {
        const data = JSON.parse(res.target.result.toString())
        this.$refs.configFile.value = null

        this.addedApps = []
        this.addedAppsIds = []

        for (const appConfig of data.apps) {
          const id = this.addNewApp(appConfig.name)
          delete appConfig.name
          this.addedApps[id].data = appConfig
        }
      }
      reader.onerror = err => console.log(err)
      reader.readAsText(file)
    },
    exportConfig () {
      const result = { apps: [] }

      for (const v of Object.values(this.addedApps)) {
        if (v.data) {
          const appConfig = { name: v.name }
          Object.assign(appConfig, v.data)
          result.apps.push(appConfig)
        }
      }
      const jsonData = encodeURIComponent(JSON.stringify(result, null, 4))
      this.exportConfigHref = `data:text/plain;charset=utf-8,${jsonData}`
      this.exportConfigFilename = 'conf.json'

      this.$nextTick(() => {
        this.$refs.downloadConfig.click()
      })
    }
  }
}
</script>
