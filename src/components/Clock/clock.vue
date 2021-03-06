<template>
  <section class="root">
    <div class="cont time" :class="{ small: panel, big: !panel }">
      <div class="clock">
        <div>{{ time.hour }}</div>
        <span>:</span>
        <div>{{ time.minute }}</div>
        <div class="indicator">
          {{ time.meridiem === 'Ante Meridiem' ? 'am' : 'pm' }}
        </div>
      </div>

      <div v-if="geo.info !== null" class="info">
        <div>
          {{ `In ${geo.info.city}, ${geo.info.country_code}` }}
        </div>
        <div>
          <button class="infoBtn" @click="changePanelStatus">
            <span>{{ panel ? 'LESS' : 'MORE' }}</span>
            <div class="iconCont">
              <span v-if="panel === true" class="icon fas fa-chevron-down">
              </span>
              <span v-if="panel === false" class="icon fas fa-chevron-up">
              </span>
            </div>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { reactive, computed } from 'vue'
import { useStore } from 'vuex'
import { geoip } from '../../services/geolocation-api/geoip.js'

export default {
  name: 'Clock',

  setup() {
    const store = useStore()
    const chronos = store.getters.getChronos

    const time = reactive({
      day: chronos.day(),
      numberDay: chronos.numberDay(),
      month: chronos.month(),
      year: chronos.year(),
      hour: chronos.hour(),
      minute: chronos.min(),
      second: chronos.sec(),
      meridiem: chronos.meridiem()
    })

    const update = () => {
      time.day = chronos.day()
      time.numberDay = chronos.numberDay()
      time.month = chronos.month()
      time.year = chronos.year()
      time.hour = chronos.hour()
      time.minute = chronos.min()
      time.second = chronos.sec()
      time.meridiem = chronos.meridiem()
    }

    setInterval(() => {
      update()
    }, 1000)

    let geo = reactive({
      info: null
    })

    const getGeoInfo = async () => {
      const info = (await geoip()).data
      geo.info = info
      store.dispatch('setLocation', info)
    }

    getGeoInfo()

    return {
      time,
      geo,
      changePanelStatus: () => store.dispatch('changePanelStatus'),
      panel: computed(() => store.getters.panelStatus)
    }
  }
}
</script>

<style scoped>
.time {
  height: 100vh;
  display: flex;
  flex-flow: column;
  justify-content: center;
}

.time.small {
  height: 50vh;
}

.time.big {
  height: 100vh;
}

.clock {
  display: flex;
  justify-content: flex-start;
  align-items: baseline;
  padding: 50px 15px 5px;
  max-width: 800px;
  width: 800px;
  font-weight: 900;
  font-size: 7rem;
}

.clock > .indicator {
  margin-left: 15px;
  font-weight: 100;
  font-size: 2rem;
}

.info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px 15px;
  font-weight: 600;
  font-size: 15px;
  text-transform: uppercase;
}

.infoBtn {
  display: flex;
  align-items: center;
  background: white;
  padding: 7px;
  border-radius: 20px;
  border: 1px solid black;
  outline: none;
}

.iconCont {
  display: flex;
  justify-content: center;
  align-items: center;
  background: black;
  border-radius: 50%;
  padding: 7px;
  margin-left: 5px;
}

.icon {
  color: white;
}

@media screen and (max-width: 800px) {
  .clock {
    width: 100vw;
    font-size: 25vw;
  }
}
</style>
