<template>
    <div class="module column">
        <div class="ui container form">
            <template v-if="!loading">
              <div class="ui segment">
                <SettingsGroup :group="mainGroup"
                               :values="values"
                               @value-change="updateValue"
                />
                <SettingsGroup v-for="group in titledGroups"
                               :group="group"
                               :values="values"
                               :key="group.name"
                               @value-change="updateValue"
                />
                <div class="ui center aligned basic segment">
                    <button class="ui primary button" @click="save"><i class="save icon"></i>Save</button>
                </div>
              </div>
            </template>
            <div v-else class="ui placeholder segment">
                <div class="ui icon header">
                    <i class="notched circle loading icon"></i>
                    Loading...
                </div>
            </div>
        </div>
    </div>
</template>

<script>
  import Promise from 'bluebird'

  import SettingsGroup from './group'

  export default {
    name: 'module',
    props: ['module'],
    components: { SettingsGroup },
    data () {
      return {
        config: {},
        changedValues: {},
        loading: true
      }
    },
    computed: {
      mainGroup () {
        return this.config.find(g => g.name === undefined) || { fields: [] }
      },
      titledGroups () {
        return this.config.filter(g => g.name !== undefined)
      }
    },
    methods: {
      updateValue (fieldName, newValue) {
        this.values[fieldName] = newValue
        this.changedValues[fieldName] = newValue
      },
      save () {
        const changedValues = this.changedValues
        this.changedValues = {}

        Promise.fromCallback(cb => this.adapter.saveValues(this.module, changedValues, cb))
          .then(() => this.toastr.success('Settings saved'))
          .catch(err => {
            console.error(err)
            return this.toastr.error('Settings failed saving')
          })
      },
      fetchConfig (moudleName) {
        return Promise.all([
          Promise.fromCallback(cb => this.adapter.getModuleConfig(moudleName, cb))
            .then(config => { this.config = config }),
          Promise.fromCallback(cb => this.adapter.getModuleValues(moudleName, cb))
            .then(values => { this.values = values })
        ])
          .then(() => { this.loading = false })
      }
    },
    mounted () {
      this.adapter = this.$electron.remote.require('./main').settingsAdapter
    },
    beforeRouteEnter (to, from, next) {
      next(vm => vm.fetchConfig(vm.module)
        .catch(err => {
          console.error(err)
        }))
    },
    beforeRouteUpdate (to, from, next) {
      this.loading = true

      this.fetchConfig(to.params.module)
        .catch(err => {
          console.error(err)
        })

      next()
    }
  }
</script>

<style scoped>
/*.module {
  height: 100%;
}*/
</style>
