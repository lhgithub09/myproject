<template>
  <div id="app">
    <!--省-->
    <el-select v-model="selectProCode" placeholder="请选择省" @change="selectProChnage">
      <el-option
        v-for="(item, index) in arrData"
        :key="index"
        :label="item.name"
        :value="item.code">
      </el-option>
    </el-select>
    <!--市-->
    <el-select v-model="selectCityCode" placeholder="请选择市" @change="selectCityChange">
      <el-option
        v-for="(item, index) in arrCity"
        :key="index"
        :label="item.name"
        :value="item.code">
      </el-option>
    </el-select>
    <!--县-->
    <el-select v-model="selectCountyCode" placeholder="请选择县">
      <el-option
        v-for="(item, index) in arrCounty"
        :key="index"
        :label="item.name"
        :value="item.code">
      </el-option>
    </el-select>
  </div>
</template>

<script>
  import areaData from '../area'
export default {
  name: 'App',
  data () {
    return {
      arrData: [],    // 所有数据
      arrCity: [],
      arrCounty: [],

      selectProCode: '',
      selectCityCode: '',
      selectCountyCode: '',
      selectPro: {},
      selectCity: {},
      selectCountty: {}
    }
  },
  methods: {
    /* 省份选择发生改变时触发 */
    selectProChnage (val) {
      let obj = {}
      obj = this.arrData.find((item) => {
        return item.code === val
      })

      this.arrCity = []
      this.arrCounty = []

      this.selectPro = obj
      this.selectProCode = obj.code

      this.selectCity = {}
      this.selectCityCode = ''
      this.selectCountty = {}
      this.selectCountyCode = ''

      if (obj.children) {
        this.arrCity = obj.children
      }
    },

    selectCityChange (val) {
      let obj = {}
      obj = this.arrCity.find((item) => {
        return item.code === val
      })

      this.arrCounty = []
      this.selectCity = obj
      this.selectCityCode = obj.code

      this.selectCountty = {}
      this.selectCountyCode = ''

      if (obj.children) {
        this.arrCounty = obj.children
      }
    }
  },
  mounted () {
    this.arrData = areaData
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}


</style>
