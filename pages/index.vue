<template>
  <div class="map-component">
    <h3>Maps</h3>
    <p>
      Zoom: {{ zoom }}<br>
      Center: {{ center }}
    </p>
    <vl-map
      ref="map"
      data-projection="EPSG:4326"
      :load-tiles-while-animating="true"
      :load-tiles-while-interacting="true"
      :style="{cursor: mapCursor}"
      class="map"
      @click="clickCoordinate = $event.coordinate"
      @postcompose="onMapPostCompose"
      @pointermove="onMapPointerMove"
    >
      <vl-view :zoom.sync="zoom" :center.sync="center" :rotation.sync="rotation" />

      <!-- interactions -->
      <vl-interaction-select :features.sync="selectedFeatures">
        <template slot-scope="select">
          <!-- select styles -->
          <vl-style-box>
            <vl-style-stroke color="#423e9e" :width="7" />
            <vl-style-fill :color="[254, 178, 76, 0.7]" />
            <vl-style-circle :radius="5">
              <vl-style-stroke color="#423e9e" :width="7" />
              <vl-style-fill :color="[254, 178, 76, 0.7]" />
            </vl-style-circle>
          </vl-style-box>
          <vl-style-box :z-index="1">
            <vl-style-stroke color="#d43f45" :width="2" />
            <vl-style-circle :radius="5">
              <vl-style-stroke color="#d43f45" :width="2" />
            </vl-style-circle>
          </vl-style-box>
          <!--// select styles -->

          <!-- selected feature popup -->
          <vl-overlay
            v-for="feature in select.features"
            :id="feature.id"
            :key="feature.id"
            :position="pointOnSurface(feature.geometry)"
            :auto-pan="true"
            :auto-pan-animation="{ duration: 300 }"
            class="feature-popup"
          >
            <template>
              <section class="card">
                <header class="card-header">
                  <p class="card-header-title">
                    ID : {{ feature.id }}
                  </p>
                  <a
                    class="card-header-icon"
                    title="Close"
                    @click="selectedFeatures = selectedFeatures.filter(f => f.id !== feature.id)"
                  >
                    <span class="absolute top-0 bottom-0 right-0 px-4 py-3">
                      <svg class="fill-current h-6 w-6 text-blue-900" role="button" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><title>Close</title><path d="M14.348 14.849a1.2 1.2 0 0 1-1.697 0L10 11.819l-2.651 3.029a1.2 1.2 0 1 1-1.697-1.697l2.758-3.15-2.759-3.152a1.2 1.2 0 1 1 1.697-1.697L10 8.183l2.651-3.031a1.2 1.2 0 1 1 1.697 1.697l-2.758 3.152 2.758 3.15a1.2 1.2 0 0 1 0 1.698z" /></svg>
                    </span>
                  </a>
                </header>
                <div class="card-content">
                  <div class="content">
                    <table>
                      <tr>
                        <td>Date</td>
                        <td>:</td>
                        <td>{{ feature.properties.date }}</td>
                      </tr>
                      <tr>
                        <td>Borough</td>
                        <td>:</td>
                        <td>{{ feature.properties.borough }}</td>
                      </tr>
                      <tr>
                        <td>On Street Name</td>
                        <td>:</td>
                        <td>{{ feature.properties.onstreetName }}</td>
                      </tr>
                      <tr>
                        <td>Cross Street Name</td>
                        <td>:</td>
                        <td>{{ feature.properties.crossStreetName }}</td>
                      </tr>
                      <tr>
                        <td>Off Street Name</td>
                        <td>:</td>
                        <td>{{ feature.properties.offStreetName || '-' }}</td>
                      </tr>
                      <tr>
                        <td>Cyclist Injured</td>
                        <td>:</td>
                        <td>{{ feature.properties.cyclistInjured }}</td>
                      </tr>
                      <tr>
                        <td>Cyclist Killed</td>
                        <td>:</td>
                        <td>{{ feature.properties.cyclistKilled }}</td>
                      </tr>
                      <tr>
                        <td>Motorist Injured</td>
                        <td>:</td>
                        <td>{{ feature.properties.motoristInjured }}</td>
                      </tr>
                      <tr>
                        <td>Motorist Killed</td>
                        <td>:</td>
                        <td>{{ feature.properties.motoristKilled }}</td>
                      </tr>
                      <tr>
                        <td>Pedestrians Injured</td>
                        <td>:</td>
                        <td>{{ feature.properties.pedestriansInjured }}</td>
                      </tr>
                      <tr>
                        <td>Pedestrians Killed</td>
                        <td>:</td>
                        <td>{{ feature.properties.pedestriansKilled }}</td>
                      </tr>
                      <tr>
                        <td>Person Injured</td>
                        <td>:</td>
                        <td>{{ feature.properties.personsInjured }}</td>
                      </tr>
                      <tr>
                        <td>Person Killed</td>
                        <td>:</td>
                        <td>{{ feature.properties.personsKilled }}</td>
                      </tr>
                      <tr>
                        <td>Vehicle Factor</td>
                        <td>:</td>
                        <td>{{ feature.properties.vehicleFactor }}</td>
                      </tr>
                      <tr>
                        <td>Vehicle Type</td>
                        <td>:</td>
                        <td>{{ feature.properties.vechileType }}</td>
                      </tr>
                    </table>
                  </div>
                </div>
              </section>
            </template>
          </vl-overlay>
          <!--// selected popup -->
        </template>
      </vl-interaction-select>
      <!--// interactions -->

      <!-- base layers -->
      <vl-layer-tile
        :id="baseLayers.name"
        :key="baseLayers.name"
        :visible="baseLayers.visible"
      >
        <component :is="`vl-source-osm`" v-bind="baseLayers" />
      </vl-layer-tile>
      <!--// base layers -->

      <component :is="layer.cmp" :key="layer.id" v-bind="layer">
        <!-- add vl-source-* -->
        <component :is="layer.source.cmp" v-bind="layer.source">
          <!-- add static features to vl-source-vector if provided -->
          <div v-if="layer.source.staticFeatures && layer.source.staticFeatures.length">
            <vl-feature
              v-for="feature in layer.source.staticFeatures"
              :id="feature.id"
              :key="feature.id"
              :properties="feature.properties"
            >
              <vl-style-box>
                <vl-style-icon src="https://img.icons8.com/office/50/000000/place-marker.png" :anchor="[0.55, 0.3]" />
              </vl-style-box>
              <component :is="geometryTypeToCmpName(feature.geometry.type)" v-bind="feature.geometry" />
            </vl-feature>
          </div>

          <!-- add inner source if provided (like vl-source-vector inside vl-source-cluster) -->
          <component
            :is="layer.source.source.cmp"
            v-if="layer.source.source"
            v-bind="layer.source.source"
          >
            <!-- add static features to vl-source-vector if provided -->
            <div v-if="layer.source.source.staticFeatures && layer.source.source.staticFeatures.length">
              <vl-feature
                v-for="feature in layer.source.source.staticFeatures"
                :id="feature.id"
                :key="feature.id"
                :properties="feature.properties"
              >
                <vl-style-box>
                  <vl-style-icon src="https://img.icons8.com/office/50/000000/place-marker.png" :anchor="[0.5, 1]" />
                </vl-style-box>
                <component :is="geometryTypeToCmpName(feature.geometry.type)" v-bind="feature.geometry" />
              </vl-feature>
            </div>
          </component>
        </component>
        <!--// vl-source-* -->
      </component>

      <!-- add style components if provided -->
      <!-- create vl-style-box or vl-style-func -->
      <div v-if="layer.style">
        <component
          :is="style.cmp"
          v-for="(style, i) in layer.style"
          :key="i"
          v-bind="style"
        >
          <!-- create inner style components: vl-style-circle, vl-style-icon, vl-style-fill, vl-style-stroke & etc -->
          <div v-if="style.styles">
            <component
              :is="cmp"
              v-for="(st, cmp) in style.styles"
              :key="cmp"
              v-bind="st"
            >
              <!-- vl-style-fill, vl-style-stroke if provided -->
              <vl-style-fill v-if="st.fill" v-bind="st.fill" />
              <vl-style-stroke v-if="st.stroke" v-bind="st.stroke" />
            </component>
          </div>
        </component>
      </div>
    </vl-map>
  </div>
</template>

<script>
import { kebabCase } from 'lodash'
import { findPointOnSurface } from 'vuelayers/lib/ol-ext'
import mapData from '@/assets/exampledata.json'
const methods = {
  pointOnSurface: findPointOnSurface,
  geometryTypeToCmpName (type) {
    return 'vl-geom-' + kebabCase(type)
  },
  onMapPostCompose () {
    this.$refs.map.render()
  },
  onMapPointerMove ({ pixel }) {
    const hit = this.$refs.map.forEachFeatureAtPixel(pixel, () => true)
    if (hit) {
      this.mapCursor = 'pointer'
    } else {
      this.mapCursor = 'default'
    }
  }
}
export default {
  name: 'MapComponent',
  data () {
    return {
      zoom: 13,
      center: [-73.91857329322053, 40.65467255991021],
      clickCoordinate: undefined,
      selectedFeatures: [],
      mapCursor: 'default',
      rotation: 0,
      baseLayers: [
        {
          name: 'osm',
          title: 'OpenStreetMap',
          visible: true
        }
      ],
      layer: {
        id: 'countries',
        title: 'Countries',
        cmp: 'vl-layer-vector',
        visible: true,
        renderMode: 'image',
        source: {
          cmp: 'vl-source-vector',
          // url: 'https://openlayers.org/en/latest/examples/data/geojson/countries.geojson',
          staticFeatures: []
        },
        style: [
          {
            cmp: 'vl-style-box',
            styles: {
              'vl-style-fill': {
                color: [255, 255, 255, 0.5]
              },
              'vl-style-stroke': {
                color: '#219e46',
                width: 2
              },
              'vl-style-text': {
                text: '\uF041',
                font: '24px FontAwesome',
                fill: {
                  color: '#2355af'
                },
                stroke: {
                  color: 'white'
                }
              }
            }
          }
        ]
      }
    }
  },
  async mounted () {
    await mapData.forEach((data) => {
      if (data.LONGITUDE) {
        this.layer.source.staticFeatures.push({
          type: 'Feature',
          id: data['UNIQUE KEY'],
          properties: {
            borough: data.BOROUGH,
            crossStreetName: data['CROSS STREET NAME'],
            cyclistInjured: data['CYCLISTS INJURED'],
            cyclistKilled: data['CYCLISTS KILLED'],
            date: data.DATE,
            motoristInjured: data['MOTORISTS INJURED'],
            motoristKilled: data['MOTORISTS KILLED'],
            offStreetName: data['OFF STREET NAME'],
            onstreetName: data['ON STREET NAME'],
            pedestriansInjured: data['PEDESTRIANS INJURED'],
            pedestriansKilled: data['PEDESTRIANS KILLED'],
            personsInjured: data['PERSONS INJURED'],
            personsKilled: data['PERSONS KILLED'],
            time: data.TIME,
            vehicleFactor: data['VEHICLE 1 FACTOR'],
            vechileType: data['VEHICLE 1 TYPE'],
            zipCode: data['ZIP CODE']
          },
          geometry: {
            coordinates: [data.LONGITUDE, data.LATITUDE],
            type: 'Point'
          }
        })
      }
    })
  },
  methods
}
</script>
<style lang="scss">
.map-component {
  padding: 20px 50px 100px;
  min-width: 768px;
  h3 {
    text-align: center;
    font-weight: bold;
    font-size: 16px;
    margin-bottom: 16px;
  }
  p {
    margin-bottom: 12px;
  }
  .map {
    width: 100%;
    height: 80vh;
  }
  .feature-popup {
    position: absolute;
    left: -50px;
    bottom: 12px;
    width: 350px;
    max-width: none;
    background: #ffffff;
    padding: 12px;
    border-radius: 12px;
    &:after, &:before {
      top: 100%;
      border: solid transparent;
      content: ' ';
      height: 0;
      width: 0;
      position: absolute;
      pointer-events: none;
    }
    &:after {
      border-top-color: white;
      border-width: 10px;
      left: 48px;
      margin-left: -10px;
    }
    &:before {
      border-top-color: #cccccc;
      border-width: 11px;
      left: 48px;
      margin-left: -11px;
    }
    .card {
      &-header {
        padding-bottom: 10px;
        border-bottom: 1px solid lightgray;
        color: rgb(54, 54, 54);
        font-weight: bold;
        font-size: 16px;
      }
      &-content {
        padding-top: 10px;
        max-height: 20em;
        overflow: auto;
        .content {
          word-break: break-word;
        }
      }
    }
  }
}
</style>
