<template>
  <div class="container-fluid">
    <div class="columns">
      <div class="column is-half">
        <h2 class="title">Webcam Sweep</h2>
        <div class="label is-small">
          <label class="label">Webcam</label>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">Model</label>
          </div>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <div class="select is-small is-fullwidth">
                  <select>
                    <option v-for="camera in camera.models" :key="camera.id">
                      {{ camera.name }}
                    </option>
                  </select>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">IP Address</label>
          </div>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <input class="input is-small" v-model="camera.ip" v-validate="'required|ip'" name="IP" type="text" placeholder="192.168.0.X">
              </div>
              <p class="help is-danger">
                {{ errors.first('IP') }}
              </p>
            </div>
          </div>
        </div>

        <div class="label is-small">
          <label class="label">Calibrate</label>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">Move</label>
          </div>
          <div class="field-body columns is-gapless">
            <div class="column is-one-third">
              <ul>
                <li><a class="noHover button is-small is-fullwidth" disabled></a></li>
                <li><a :disabled="!this.camera.ready" class="button is-small is-fullwidth is-success" @click="moveCamera('l', 100)">Left</a></li>
                <li><a class="noHover button is-small is-fullwidth" disabled></a></li>
              </ul>
            </div>
            <div class="column is-one-third">
              <ul>
                <li><a :disabled="!this.camera.ready" class="button is-small is-fullwidth is-success" @click="moveCamera('u', 100)">Up</a></li>
                <li><a class="button is-small is-fullwidth is-warning" @click="calibrateCamera()">Calibrate</a></li>
                <li><a :disabled="!this.camera.ready" class="button is-small is-fullwidth is-success" @click="moveCamera('d', 100)">Down</a></li>
              </ul>
            </div>
            <div class="column is-one-third">
              <ul>
                <li><a class="noHover button is-small is-fullwidth" disabled></a></li>
                <li><a :disabled="!this.camera.ready" class="button is-small is-fullwidth is-success" @click="moveCamera('r', 100)">Right</a></li>
                <li><a class="noHover button is-small is-fullwidth" disabled></a></li>
              </ul>
            </div>
          </div>          
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">Position</label>
          </div>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <p class="help">
                  x : {{ this.camera.position.x }} / y : {{ this.camera.position.y }}
                </p>
              </div>
            </div>
          </div>
        </div>

        <div class="label is-small">
          <label class="label">Animation</label>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">Range</label>
          </div>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <vb-slider type="success" size="large" :value="this.camera.sweep" :max="1300" :min="100" :step="50" is-fullwidth @change="updateAnimationSweep"></vb-slider>
                <p class="help">
                  {{ this.camera.sweep }}
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">Speed</label>
          </div>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <vb-slider type="success" size="large" :value="this.camera.speed" :max="900" :min="100" :step="50" is-fullwidth @change="updateAnimationSpeed"></vb-slider>
                <p class="help">
                  {{ this.camera.speed }}
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="field is-horizontal">
          <div class="field-label is-small">
            <label class="label">Start</label>
          </div>
          <div class="field-body">
            <div class="field">
              <div class="control">
                <vb-switch :disabled="!this.camera.ready && !this.camera.running" type="success" size="large" v-model="camera.running" @change="startAnimation"/>
              </div>
            </div>
          </div>
        </div>    
      </div>
    </div>    
  </div>
</template>

<script>
  import Vue from 'vue'
  import VeeValidate from 'vee-validate'
  import VbSwitch from './LandingPage/VueBulmaSwitch'
  import VbSlider from './LandingPage/VueBulmaSlider'
  import axios from 'axios'
  import querystring from 'querystring'

  Vue.use(VeeValidate)

  export default {
    components: {
      VbSwitch,
      VbSlider
    },
    name: 'landing-page',
    data: function () {
      return {
        camera: {
          model: 'Xiaomi Dafang',
          ready: false,
          running: false,
          speed: 800,
          sweep: 400,
          dir: 'l',
          models: [
            {id: 0, name: 'Xiaomi Dafang'},
            {id: 1, name: 'Heden IPCAM'}
          ],
          ip: '192.168.1.76',
          position: {
            x: 0,
            y: 0
          },
          range: {
            x: 0,
            y: 0
          }
        }
      }
    },
    methods: {
      startAnimation () {
        if (this.camera.ready) {
          this.moveCamera(this.camera.dir, this.camera.sweep)
        }
      },
      updateAnimationSpeed (val) {
        this.camera.speed = Number(val)
      },
      updateAnimationSweep (val) {
        this.camera.sweep = Number(val)
      },
      calibrateCamera () {
        var command = ''
        var vm = this
        switch (vm.camera.model) {
          case 'Xiaomi Dafang':
            command = 'http://' + this.camera.ip + ':5050/cgi-bin/action.cgi'
            break
          default:
            break
        }
        axios.post(command, querystring.stringify({
          cmd: 'motor_calibrate',
          speed: vm.camera.speed
        })).then(function (response) {
          var status = response.data.camera_status
          vm.camera.position.x = status.x
          vm.camera.position.y = status.y
          vm.camera.range.x = status.max_x
          vm.camera.range.y = status.max_y
          vm.camera.speed = status.speed
          vm.camera.ready = true
        }).catch(function (error) {
          console.log(error)
          vm.camera.ready = false
        })
      },
      moveCamera (dir, steps) {
        var command = ''
        var vm = this
        vm.camera.ready = false
        vm.camera.dir = dir
        switch (vm.camera.model) {
          case 'Xiaomi Dafang':
            command = 'http://' + this.camera.ip + ':5050/cgi-bin/action.cgi'
            break
          default:
            break
        }
        axios.post(command, querystring.stringify({
          cmd: 'motor_move',
          dir: dir,
          steps: steps,
          speed: vm.camera.speed
        })).then(function (response) {
          var status = response.data.camera_status
          vm.camera.position.x = status.x
          vm.camera.position.y = status.y
          vm.camera.speed = status.speed
          vm.camera.ready = true
          if (vm.camera.running) {
            vm.camera.dir = vm.camera.dir === 'l' ? 'r' : 'l'
            vm.moveCamera(vm.camera.dir, vm.camera.sweep * 2)
          }
        }).catch(function (error) {
          console.log(error)
          vm.camera.ready = true
        })
      }
    }
  }
</script>

<style>
body {
  overflow: hidden;
}
canvas {
  display:block;
}
.noHover {
  cursor: default !important;
  border-color: #FFF !important;
}
</style>
