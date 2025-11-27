<template>
  <div class="container">
    <div class="columns">
      <div class="column is-3" style="padding-bottom: 2rem;">
        <div class="content">
          <h3 class="subtitle">
            Selection
          </h3>
          <b-field style="margin-bottom: 0.25rem;">
              <b-input v-model="searchStore" placeholder="search store"></b-input>
          </b-field>
          <p style="font-size: 70%; color: #999;">
            {{ store_list.length }} stores selected
          </p>
          <h4 class="subtitle">
            Extent
          </h4>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showGlobal">Global</b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showRegional">Regional</b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showLocal">Local</b-switch>
            </div>
            <h4 class="subtitle">
              Sampling Rate
            </h4>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showSampling1Hz">&le; 1 Hz</b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showSampling2Hz">&le; 2 Hz</b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showSamplingGt2Hz">&gt; 2 Hz</b-switch>
            </div>
            <h4 class="subtitle">
              Backends
            </h4>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showAhfullgreen">
                <b-tooltip label="AHFULLGREEN is a code to calculate synthetic seismograms in elorec homogeneous fullspace." position="is-right" multilined dashed>
                  AHFULLGREEN
                </b-tooltip>
              </b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showQSEIS">
                <b-tooltip label="QSEIS is a code to calculate synthetic seismograms based on a layered viscoelastic half-space model." position="is-right" multilined dashed>
                  QSEIS
                </b-tooltip>
              </b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showQSSP">
                <b-tooltip label="QSSP is a code to calculate complete synthetic seismograms of a layered, self-gravitating spherical Earth" position="is-right" multilined dashed>
                  QSSP
                </b-tooltip>
              </b-switch>
            </div>
            <div class="field">
              <b-switch size="is-small" type="is-light" v-model="showPSGRNPSCMP">
                <b-tooltip label="PSGRN-PSCMP is a code to calculate synthetic surface displacements (stress/strain/tilt/gravitational) on a layered viscoelastic halfspace." position="is-right" multilined dashed>
                  PSGRN-PSCMP
                </b-tooltip>
              </b-switch>
            </div>
            <div class="field has-text-centered">
              <input class="button is-info is-outlined" type="reset" value="Reset" @click="resetForm()">
            </div>
        </div>
      </div>
      <div class="column" style="padding-bottom: 2rem;">
        <b-table
          ref="table"
          :data="isEmpty ? [] : store_list"
          :bordered="false"
          :striped="false"
          :hoverable="true"
          :is-loading="isEmpty"
          :loading="isEmpty"
          :mobile-cards="true"

          detailed
          detail-key="uuid"
          :opened-detailed="[getStoreUUID($route.params.id ? $route.params.id : 'ah_store')]"
          @click="openDetails">

          <template slot-scope="store">
            <b-table-column field="id" label="ID" style="font-family: monospace;">
              {{ store.row.id }}
            </b-table-column>
            <b-table-column field="type" label="Type">
              <b-tag>{{ store.row.short_type }}</b-tag>
            </b-table-column>
            <b-table-column field="hor_extent" label="↔ Extent" class="has-text-right">
              {{ store.row.distance_min | as_km }} - {{ store.row.distance_max | as_km }}
            </b-table-column>
            <b-table-column field="hor_spacing">
              @ {{ store.row.distance_delta | as_km }} km
            </b-table-column>
            <b-table-column field="ver_extent" label="↕ Depth" class="has-text-right">
              {{ store.row.source_depth_min | as_km }} - {{ store.row.source_depth_max | as_km }}
            </b-table-column>
            <b-table-column field="ver_spacing" label=" ">
              @ {{ store.row.source_depth_delta | as_km }} km
            </b-table-column>
            <b-table-column field="sample_rate" label="Sample Rate" class="has-text-right" sortable>
              {{ store.row.sample_rate | sample_rate }}
            </b-table-column>
            <b-table-column field="uuid" label="UUID" class="has-text-right" style="font-family: monospace;">
              <a :href="store.row.reference" style="color: #aaa">{{ store.row.uuid.slice(0, 6) }}</a>
            </b-table-column>
          </template>

          <template slot="detail" slot-scope="store">
            <div class="column content">
              <h1 :id="store.row.uuid">{{ store.row.id }} <a :href="store.row.reference" style="font-size: 11pt; font-weight: normal; padding-left: .75rem;">Link</a></h1>
              <p>
                Download this Green's Function store with
                <code class="select-all">fomosto download pyrocko {{store.row.id}}</code>
              </p>

              <table style="max-width: 650px">
                <tr>
                  <td>Modelling Code</td>
                  <td><span class="tag is-success is-uppercase">{{ store.row.modelling_code_id }}</span></td>
                </tr>
                <tr>
                  <td>Horizontal extent</td>
                  <td>{{ store.row.distance_min | as_km }} - {{ store.row.distance_max | as_km }} @ {{ store.row.distance_delta | as_km }} km</td>
                </tr>
                <tr>
                  <td>Source depths</td>
                  <td>{{ store.row.source_depth_min | as_km }} - {{ store.row.source_depth_max | as_km }} @ {{ store.row.source_depth_delta | as_km }} km</td>
                </tr>
                <tr>
                  <td>Store size</td>
                  <td>{{ store.row.size | prettyBytes }}</td>
                </tr>
                <tr>
                  <td>UUID</td>
                  <td><a :href="store.row.uuid" style="font-family: monospace;">{{ store.row.uuid }}</a></td>
                </tr>
              </table>
            </div>
            <div class="column content" style="padding-top: 0px; padding-bottom: 0px;">
              <p>
                <button class="button" @click="showConfig(store)">Show configuration</button>
                <button class="button" @click="goToStore(store.row.id)">Show store directory</button>
              </p>
            </div>
            <div class="column content" style="padding-bottom: 0px">
              <h3>Velocity Model</h3>
              <img class="image profile" :src="`${api_endpoint}${store.row.id}/profile`" />
            </div>
          </template>

        </b-table>
      </div>
    </div>
    <div class="modal" v-bind:class="{ 'is-active': modalData }" v-if="modalData">
      <div class="modal-background" @click="closeConfig()"></div>
      <div class="modal-card" style="width: 70%;">
         <header class="modal-card-head">
           <p class="modal-card-title"><span style="font-family: monospace;">{{ modalData.id }}/config.yml</span></p>
           <button class="delete" aria-label="close" @click="closeConfig()"></button>
         </header>
         <section class="modal-card-body">
           <pre>{{ modalData.config }}</pre>
         </section>
       </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

const km = 1000.0

export default {
  name: 'StoreList',

  data () {
    return {
      api_endpoint: 'https://greens-mill.pyrocko.org/api/',
      static_endpoint: 'https://gf.pyrocko.org/gfws/static/',
      isEmpty: true,
      data: undefined,
      modalData: undefined,

      searchStore: '',
      showGlobal: true,
      showRegional: true,
      showLocal: true,
      showSampling1Hz: true,
      showSampling2Hz: true,
      showSamplingGt2Hz: true,

      showAhfullgreen: true,
      showQSEIS: true,
      showQSSP: true,
      showPSGRNPSCMP: true
    }
  },
  methods: {
    openDetails (row) {
      this.$refs.table.toggleDetails(row)
    },
    getStoreUUID (ref) {
      for (var is = 0; is < this.store_list.length; is++) {
        let store = this.data.stores[is]
        if (store.reference === ref || store.uuid === ref) {
          return store.uuid
        }
      }
    },
    showConfig (store) {
      axios
        .get(`${this.api_endpoint}${store.row.id}`)
        .then(response => {
          this.modalData = response.data
        })
        .catch(error => {
          console.log(error)
        })
    },
    goToStore (storeId) {
      window.location.assign(`${this.static_endpoint}stores/${storeId}/`)
    },
    closeConfig () {
      this.modalData = undefined
    },
    resetForm () {
      this.searchStore = ''
      this.showGlobal = true
      this.showRegional = true
      this.showLocal = true
      this.showSampling1Hz = true
      this.showSampling2Hz = true
      this.showSamplingGt2Hz = true
      this.showAhfullgreen = true
      this.showQSEIS = true
      this.showQSSP = true
      this.showPSGRNPSCMP = true
    },
    scrollToStore () {
      if (this.$route.params.id !== undefined) {
        var storeRef = this.getStoreUUID(this.$route.params.id)
        window.setTimeout(function () {
          let el = document.getElementById(storeRef)
          el.scrollIntoView(true)
        }, 200)
      }
    }

  },
  mounted () {
    axios
      .get(this.api_endpoint)
      .then(response => {
        this.data = response.data
        this.isEmpty = false
        this.scrollToStore()
      })
      .catch(error => {
        console.log(error)
      })
  },
  computed: {
    store_list: function () {
      if (!this.data) {
        return []
      }

      var stores = []
      var search = new RegExp(this.searchStore, 'i')

      for (var is = 0; is < this.data.stores.length; is++) {
        var store = this.data.stores[is]
        if (!store.uuid) {
          store.uuid = 'none-' + is.toString()
        }
        if (!store.reference) {
          store.reference = 'none'
        }
        if (!store.modelling_code_id) {
          store.modelling_code_id = 'unknown'
        }

        // Extent
        var extentKm = store.distance_max / km
        if (!this.showGlobal && extentKm > 1000) {
          continue
        }
        if (!this.showRegional && extentKm >= 100 && extentKm <= 1000) {
          continue
        }
        if (!this.showLocal && extentKm < 100) {
          continue
        }

        // Sampling Rate
        if (!this.showSampling1Hz && store.sample_rate <= 1.0) {
          continue
        }
        if (!this.showSampling2Hz && store.sample_rate <= 2.0 && store.sample_rate > 1.0) {
          continue
        }
        if (!this.showSamplingGt2Hz && store.sample_rate > 2.0) {
          continue
        }

        // Backend
        if (!this.showAhfullgreen && store.modelling_code_id.search('ahfullgreen') >= 0) {
          continue
        }
        if (!this.showQSEIS && store.modelling_code_id.search('qseis') >= 0) {
          continue
        }
        if (!this.showQSSP && store.modelling_code_id.search('qssp') >= 0) {
          continue
        }
        if (!this.showPSGRNPSCMP && store.modelling_code_id.search('psgrn_pscmp') >= 0) {
          continue
        }

        if (this.searchStore !== '') {
          if (store.id.search(search) < 0) {
            continue
          }
        }

        stores.push(store)
      }
      return stores
    }
  },
  filters: {
    as_km: function (value) {
      return value / 1e3
    },
    sample_rate: function (value) {
      if (value === 0.000011574074074074073) {
        return '1 Day'
      } else {
        return value + ' Hz'
      }
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
table tr {
  cursor: pointer;
  font-size: 11pt;
}

table tr.detail {
  cursor: initial;
}

img.profile {
  width: 50%;
  -webkit-box-shadow: 0px 0px 2px 0px rgba(0,0,0,0.75);
  -moz-box-shadow: 0px 0px 2px 0px rgba(0,0,0,0.75);
  box-shadow: 0px 0px 2px 0px rgba(0,0,0,0.75);
}

h2 {
  font-size: 13pt !important;
}

code.select-all {
  -webkit-user-select: all;
  -moz-user-select: all;
  -ms-user-select: all;
  user-select: all;
}
</style>
