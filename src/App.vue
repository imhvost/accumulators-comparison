<template>
  <div class="page">
    <ul class="btns">
      <li>
        <label class="file-label">
          <input
            type="file"
            accept="application/json, .json"
            @change="importFromJSON($event)"
          >
          <span>Import from JSON file</span>
        </label>
      </li>
      <li>
        <button @click="getDemoData()">Get demo data</button>
      </li>
      <li v-if="accumulators.length || batteries.length">
        <button @click="saveAsJSON()">Save as JSON file</button>
      </li>
    </ul>
    <div class="batteries">
      <ul
        v-if="batteries.length"
        class="batteries-list">
        <li 
          v-for="(battery, index) in batteries"
          :key="index"
          class="battery"
        >
          <b>({{index + 1}})</b>
          <label>
            <input
              type="number"
              v-model="battery.s"
              placeholder="Successively"
              @input="updateStorage()"
            >
            <span>s</span>
          </label>
          <label>
            <input
              type="number"
              v-model="battery.p"
              placeholder="Parallel"
              @input="updateStorage()"
            >
            <span>p</span>
          </label>
          <button @click="copyBattery(index)">Copy</button>
          <button @click="removeBattery(index)">Remove</button>
        </li>
      </ul>
      <button @click="addBattery()">Add battery</button>
    </div>
    <div class="accumulators">
      <div
        v-if="totalАccumulators"
        class="accumulators-total"
      >
        <span>Total accumulators need: <b>{{totalАccumulators}} pieces</b></span>
        <button
          v-if="accumulators.length> 1"
          @click="sortAccumulatorsByBenefit()"
        >Sort by benefit</button>
        <button
          v-if="accumulators.length> 1"
          @click="sortAccumulatorsByCapacity()"
        >Sort by capacity</button>
      </div>
      <div
        v-if="accumulators.length"
        class="accumulators-table-wrapp"
      >
        <table class="accumulators-table">
          <tbody 
            v-for="(accumulator, index) in accumulators"
            :key="index"
            :class="accumulatorBenefitIndex === index ? 'benefit' : ''"
          >
            <tr>
              <td>
                <label>
                  <span>Model name</span>
                  <input
                    type="text"
                    v-model="accumulator.name"
                    placeholder="Model name"
                    @input="updateStorage()"
                  >
                </label>
              </td>
              <td>
                <label>
                  <a
                    v-if="accumulator.link"
                    :href="accumulator.link"
                    target="_blank"
                  >Link to shop</a>
                  <span v-else>Link to shop</span>
                  <input
                    type="url"
                    v-model="accumulator.link"
                    placeholder="Link to shop"
                    @input="updateStorage()"
                  >
                </label>
              </td>
              <td>
                <label>
                  <span>Voltage, <b>V</b></span>
                  <input
                    type="number"
                    v-model="accumulator.voltage"
                    placeholder="Voltage"
                    @input="updateStorage()"
                  >
                </label>
              </td>
              <td>
                <label>
                  <span>Amperage, <b>A</b></span>
                  <input
                    type="number"
                    v-model="accumulator.amperage"
                    placeholder="Amperage"
                    @input="updateStorage()"
                  >
                </label>
              </td>
              <td>
                <label>
                  <span>Capacity, <b>mAh</b></span>
                  <input
                    type="number"
                    v-model="accumulator.capacity"
                    placeholder="Capacity"
                    @input="updateStorage()"
                  >
                </label>
              </td>
              <td>
                <label>
                  <span>Price, <b>$</b></span>
                  <input
                    type="number"
                    v-model="accumulator.price"
                    placeholder="Price"
                    @input="updateStorage()"
                  >
                </label>
              </td>
              <td>
                <button @click="copyAccumulator(index)">Copy</button>
                <button @click="removeAccumulator(index)">Remove</button>
              </td>
            </tr>
            <tr v-if="batteries.length && (accumulator.voltage || accumulator.amperage || accumulator.capacity)">
              <td colspan="7">
                <table
                  v-if="accumulator.capacity || accumulator.amperage || accumulator.voltage"
                  class="accumulator-total"
                >
                  <thead>
                    <td>Battery</td>
                    <td>Pieces</td>
                    <td>Voltage</td>
                    <td>Amperage</td>
                    <td>Capacity</td>
                    <td>Total Cost</td>
                    <td>Cost 1 Ah</td>
                  </thead>
                  <tbody>
                    <tr
                      v-for="(battery, index) in batteries"
                      :key="index"
                    >
                      <td>
                        <b>({{index + 1}})</b>
                      </td>
                      <td>
                        <span v-if="battery.p && battery.s">
                          <b>{{getBatteryPieces(battery.p, battery.s)}}</b> pieces
                        </span>
                      </td>
                      <td>
                        <span v-if="battery.s && accumulator.voltage">
                          <b>{{getBatteryVoltage(battery.s, accumulator.voltage)}}</b> V
                        </span>
                      </td>
                      <td>
                        <span v-if="battery.p && accumulator.capacity">
                          <b>{{getBatteryAmperage(battery.p, accumulator.amperage)}}</b> A
                        </span>
                      </td>
                      <td>
                        <span v-if="battery.p && accumulator.capacity">
                          <b>{{getBatteryCapacity(battery.p, accumulator.capacity)}}</b> Ah
                        </span>
                      </td>
                      <td>
                        <span v-if="battery.p && battery.s  && accumulator.price">
                          <b>{{getBatteryCost(battery.p, battery.s, accumulator.price)}}</b> $
                        </span>
                      </td>
                      <td>
                        <span v-if="battery.p && battery.s && accumulator.price && accumulator.capacity">
                          <b>{{costBatteriesCapacity(battery.p, battery.s, accumulator.price, accumulator.capacity)}}</b> $
                        </span>
                      </td>
                    </tr>
                  </tbody>
                  <tfoot v-if="batteries.length > 1">
                    <tr>
                      <td>
                        <b>Total</b>
                      </td>
                      <td>
                        <span v-if="totalАccumulators">
                          <b>{{totalАccumulators}}</b> pieces
                        </span>
                      </td>
                      <td></td>
                      <td></td>
                      <td>
                        <span v-if="accumulator.capacity">
                          <b>{{getTotalBatteriesCapacity(accumulator.capacity)}}</b> Ah
                        </span>
                      </td>
                      <td>
                        <span v-if="accumulator.price">
                          <b>{{getTotalBatteriesCost(accumulator.price)}}</b> $
                        </span>
                      </td>
                      <td>
                        <span v-if="accumulator.price && accumulator.capacity">
                          <b>{{costTotalBatteriesCapacity(accumulator.price, accumulator.capacity)}}</b> $
                        </span>
                      </td>
                    </tr>
                  </tfoot>
                </table>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <ul class="btns">
        <li>
          <button @click="addAccumulator()">Add accumulator model</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue"
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    // HelloWorld,
  },
  setup() {
    const storage = localStorage.getItem('data') ? JSON.parse(localStorage.getItem('data')) : {}
    const updateStorage = () => {
      const data = {
        batteries: batteries.value,
        accumulators: accumulators.value
      }
      localStorage.setItem('data', JSON.stringify(data))
    }
    const batteries = ref(storage.batteries || [])
    const addBattery = () => {
      batteries.value.push({
        s: 0,
        p: 0
      })
      updateStorage()
    }
    const copyBattery = index => {
      batteries.value.splice(index, 0, {...batteries.value[index]})
      updateStorage()
    }
    const removeBattery = index => {
      batteries.value.splice(index, 1)
      updateStorage()
    }
    const getBatteryPieces = (p = 0, s = 0) => Number(p) * Number(s)
    const getBatteryVoltage = (s = 0, voltage = 0) => Math.round((Number(voltage) * Number(s)))
    const getBatteryAmperage = (p = 0, amperage = 0) => Math.round((Number(amperage) * Number(p)))
    const getBatteryCapacity = (p = 0, capacity = 0) => (Number(capacity) / 1000 * Number(p)).toFixed(1)
    const getBatteryCost = (p = 0, s = 0, price = 0) => Math.round(Number(p) * Number(s) * Number(price))
    const costBatteriesCapacity = (p = 0, s = 0, price = 0, capacity = 0) => Math.round( getBatteryCost(p, s, price) / getBatteryCapacity(p, capacity) )
    const accumulators = ref(storage.accumulators || [])
    const addAccumulator = () => {
      accumulators.value.push({
        name: '',
        link: '',
        voltage: '',
        amperage: '',
        capacity: '',
        price: '',
      })
      updateStorage()
    }
    const copyAccumulator = index => {
      accumulators.value.splice(index, 0, {...accumulators.value[index]})
      updateStorage()
    }
    const removeAccumulator = index => {
      accumulators.value.splice(index, 1)
      updateStorage()
    }
    const totalАccumulators = computed(() => {
      let count = 0;
      batteries.value.forEach(battery => count += Number(battery.s) * Number(battery.p))
      return count
    })
    const batteriesP = computed(() => {
      let p = 0;
      batteries.value.forEach(battery => p += Number(battery.p))
      return p;
    })
    const accumulatorBenefitIndex = computed(() =>{
      let benefitIndex
      let benefit = Infinity
      accumulators.value.forEach((accumulator, index) => {
        if(!accumulator.benefit){
          accumulator.benefit = costTotalBatteriesCapacity(accumulator.price, accumulator.capacity)
        }
        if(Number(accumulator.benefit) < benefit){
          benefit = Number(accumulator.benefit)
          benefitIndex = index
        }
      })
      return benefitIndex
    })
    const costTotalBatteriesCapacity = (price = 0, capacity = 0) => Math.round( totalАccumulators.value * Number(price) / (Number(capacity) / 1000 * Number(batteriesP.value)) )
    const getTotalBatteriesCapacity = (capacity = 0) => (Number(capacity) / 1000 * batteriesP.value).toFixed(1)
    const getTotalBatteriesCost = (price = 0) => Math.round(Number(price) * totalАccumulators.value)
    const sortAccumulatorsByBenefit = () => {
      accumulators.value.forEach(accumulator => {
        if(accumulator.benefit) return
        accumulator.benefit = costTotalBatteriesCapacity(accumulator.price, accumulator.capacity)
      })
      accumulators.value.sort((a, b) => Number(a.benefit) - Number(b.benefit))
      updateStorage()
    }
    const sortAccumulatorsByCapacity = () => {
      accumulators.value.sort((a, b) => Number(b.capacity) - Number(a.capacity))
      updateStorage()
    }
    const saveAsJSON = () => {
      const a = document.createElement('a')
      const file = new Blob([localStorage.getItem('data')], {type: 'application/json'})
      a.href = URL.createObjectURL(file)
      a.download = Date.now() + '.json'
      a.click()
    }
    const importFromJSON = (event) => {
      const files = event.target.files
      if(!files) return
      const file = files[0]
      if(file.name.split('.')[1] !== 'json'){
        alert('Not correct file')
        return
      }
      const reader = new FileReader()
      reader.readAsText(file)
      reader.onload = event => {
        let data
        try{
            data = JSON.parse(event.target.result)
        }catch(e){
          alert('Not correct file')
          return
        }
        if(data.batteries && Array.isArray(data.batteries)){
          batteries.value = data.batteries
        }
        if(data.accumulators && Array.isArray(data.accumulators)){
          accumulators.value = data.accumulators
        }
        updateStorage()
      }
    }
    const getDemoData = () => {
      async function postData() {
        const response = await fetch('http://accumulators-comparison.imhvost.top/demo.json', {
          mode: 'cors',
        })
        return await response.json()
      }
      postData()
        .then((data) => {
           batteries.value = data.batteries
           accumulators.value = data.accumulators
           updateStorage()
        })
    }
    return {
      batteries,
      addBattery,
      copyBattery,
      removeBattery,
      getBatteryPieces,
      getBatteryVoltage,
      getBatteryAmperage,
      getBatteryCapacity,
      getBatteryCost,
      costBatteriesCapacity,
      accumulators,
      addAccumulator,
      copyAccumulator,
      removeAccumulator,
      totalАccumulators,
      getTotalBatteriesCapacity,
      costTotalBatteriesCapacity,
      getTotalBatteriesCost,
      sortAccumulatorsByBenefit,
      sortAccumulatorsByCapacity,
      accumulatorBenefitIndex,
      saveAsJSON,
      importFromJSON,
      getDemoData,
      updateStorage
    }
  },
}
</script>

<style lang="less" scoped>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
}
.page{
  font-family: 'Franklin Gothic', 'Arial Narrow', Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding:40px;
  font-size:14px;
}
.batteries{
  margin-bottom:40px;
  padding:15px;
  background-color:#fafafa;
  box-shadow:-2px 2px 4px rgba(0,0,0,.1);
}
.batteries-list{
  list-style:none;
  li{
    display:flex;
    margin-bottom:15px;
    align-items:center;
    &>b{
      margin-right:10px;
    }
  }
  label{
    display:block;
    margin-right:10px;
    display:flex;
    align-items:center;
  }
  input{
    margin-right:5px;
    width:60px;
  }
  span{
    text-transform:uppercase;
    font-weight:700;
  }
}
input{
  height:30px;
  padding:0 5px;
  border:solid 1px #ccc;
}
button{
  border:solid 1px #ccc;
  background-color:#f0f0f0;
  cursor:pointer;
  padding:5px 10px;
  height:30px;
  &+&{
    margin-left:-1px;
  }
}
.accumulators-total{
  margin:30px 0 20px;
  display:flex;
  align-items:center;
  &>*:not(:last-child){
    margin-right:15px;
  }
}
.accumulators-table-wrapp{
  margin:0 -10px 20px;
}
.accumulators-table{
  border-collapse:collapse;
  &>tbody{
    &>tr{
      &>td{
        padding:15px 15px 15px 0;
        vertical-align:bottom;
        &:first-child{
          padding-left:10px;
          border-left:solid 1px #ccc;
          input{
            width:250px;
          }
        }
        &:last-child{
          padding-right:10px;
          border-right:solid 1px #ccc;
        }
        &:nth-child(2){
          input{
            width:200px;
          }
        }
      }
      &:first-child{
        &>td{
          border-top:solid 1px #ccc;
        }
      }
      &:last-child{
        &>td{
          border-bottom:solid 1px #ccc;
        }
      }
    }
    label{
      &>*{
        display:block;
        &:not(:last-child){
          margin-bottom:5px;
        }
      }
    }
    input{
      width:90px;
      text-overflow:ellipsis;
    }
  }
  .benefit{
    background-color:#caffca;
  }
}
.accumulator-total{
  border-collapse:collapse;
  margin-top:-15px;
  background-color:#fafafa;
  width:100%;
  td{
    padding:0 20px 10px 20px;
  }
  thead{
    font-style:italic;
    td{
      padding-top:10px;
    }
  }
  tbody{
    tr{
      &:first-child{
        td{
          padding-top:10px;
          border-top:solid 1px #ccc;
        }
      }
    }
  }
  tfoot{
    td{
      padding-top:10px;
      border-top:solid 1px #ccc;
    }
  }
}
.btns{
  display:flex;
  margin:0 -10px 20px;
  list-style:none;
  li{
    padding:0 10px;
  }
}
.file-label{
  display:flex;
  flex-direction:column;
  justify-content:space-between;
  align-items:center;
  border:solid 1px #ccc;
  background-color:#f0f0f0;
  cursor:pointer;
  padding:5px 10px;
  height:30px;
  position:relative;
  overflow:hidden;
  input{
    font-size:0;
    cursor:pointer;
    position:absolute;
    right:0;
    bottom:0;
    width:100%;
    height:100%;
    padding:0;
    border:0;
    &::-webkit-file-upload-button{
      display:none;
    }
  }
}
</style>
