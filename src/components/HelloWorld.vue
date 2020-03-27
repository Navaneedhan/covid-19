<template>
  <div id="chart">
    <h2>A live (unofficial) dashboard for covid-19 India</h2>
    <div id="chart-plot">
      <highcharts :options="chartOptionsCumm"></highcharts>
      <hr>
      <highcharts :options="chartOptionsDaily"></highcharts>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'HelloWorld',
  created () {
    axios.get('https://api.covid19india.org/data.json').then((resp) => {
      let dailyData = resp.data.cases_time_series
      let todayStats = resp.data.statewise
      let confirmedCasesDailyCumm = dailyData.filter((data) => { return !isNaN(parseInt(data.totalconfirmed)) }).map(data => parseInt(data.totalconfirmed))

      let confirmedCasesDaily = dailyData.filter((data) => { return !isNaN(parseInt(data.dailyconfirmed)) }).map(data => parseInt(data.dailyconfirmed))

      // Live data
      let todaysConfirmedData = todayStats.filter((data) => { return data.state === 'Total' })[0].confirmed

      // yesterday's data for prediction
      let yesDen = confirmedCasesDailyCumm[confirmedCasesDailyCumm.length - 2]
      let yesNum = confirmedCasesDailyCumm[confirmedCasesDailyCumm.length - 1]
      let yesterDaysGrowthRatio = yesNum / yesDen

      const oneDay = 24 * 60 * 60 * 1000
      const targetDate = new Date('march 31 2020')
      let yesDate = new Date()
      yesDate.setDate(yesDate.getDate() - 1)

      let toDate = new Date()

      let yesNoOfDays = Math.round(Math.abs((targetDate - yesDate) / oneDay))
      let toNoOfDays = Math.round(Math.abs((targetDate - toDate) / oneDay))

      let i
      let yesPredictions = []
      yesPredictions.push(confirmedCasesDailyCumm)
      yesPredictions = yesPredictions.flat()
      let a = yesNum
      for (i = 1; i <= yesNoOfDays; i++) {
        yesPredictions.push(a * Math.pow(yesterDaysGrowthRatio, i))
        a = yesPredictions[yesPredictions.length - 1]
      }

      console.log(confirmedCasesDailyCumm)
      console.log(yesPredictions)
      console.log(toNoOfDays)

      // today's data for prediction
      confirmedCasesDailyCumm.push(parseInt(todaysConfirmedData))
      let todayDen = confirmedCasesDailyCumm[confirmedCasesDailyCumm.length - 2]
      let todayNum = confirmedCasesDailyCumm[confirmedCasesDailyCumm.length - 1]
      let latestDaysGrowthRatio = todayNum / todayDen

      let toPredictions = []
      toPredictions.push(confirmedCasesDailyCumm)
      toPredictions = toPredictions.flat()
      a = todayNum
      for (i = 1; i <= toNoOfDays; i++) {
        toPredictions.push(a * Math.pow(latestDaysGrowthRatio, i))
        a = toPredictions[toPredictions.length - 1]
      }

      console.log(yesterDaysGrowthRatio)
      console.log(latestDaysGrowthRatio)
      this.chartOptionsCumm = {
        title: {
          text: 'Covid-19 cases Cummulative'
        },
        credits: {
          enabled: false
        },
        series: [
          {
            name: 'Last day prediction',
            type: 'spline',
            data: yesPredictions,
            color: 'black'
          },
          {
            name: 'Today\'s prediction',
            type: 'spline',
            data: toPredictions,
            color: 'red'
          },
          {
            name: 'Covid-19 Cases',
            type: 'spline',
            data: confirmedCasesDailyCumm,
            color: 'violet'
          }
        ],
        xAxis: {
          title: {
            text: 'Days'
          }
        },
        yAxis: {
          tickInterval: 10,
          min: 0,
          minRange: 0,
          title: {
            text: 'Cases'
          }
        }
      }
      this.chartOptionsDaily = {
        title: {
          text: 'Covid-19 cases Daily scenes'
        },
        credits: {
          enabled: false
        },
        series: [
          {
            name: 'Covid-19 Cases',
            type: 'spline',
            data: confirmedCasesDaily
          }
        ],
        xAxis: {
          title: {
            text: 'Days'
          }
        },
        yAxis: {
          title: {
            text: 'Cases'
          }
        }
      }
    })
  },
  data () {
    return {
      chartOptionsCumm: {},
      chartOptionsDaily: {}
    }
  }
}
</script>
<style>
  #chart {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }
  #chart-plot {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin: 60px 20px 0px 20px;
    width: 50%;
  }
</style>
