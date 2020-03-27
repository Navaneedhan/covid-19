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
      let confirmedCasesDailyCumm = dailyData.filter((data) => { return !isNaN(parseInt(data.totalconfirmed)) }).map(data => parseInt(data.totalconfirmed))

      let confirmedCasesDaily = dailyData.filter((data) => { return !isNaN(parseInt(data.dailyconfirmed)) }).map(data => parseInt(data.dailyconfirmed))
      this.chartOptionsCumm = {
        title: {
          text: 'Covid-19 cases Cummulative'
        },
        credits: {
          enabled: false
        },
        series: [
          {
            name: 'Covid-19 Cases',
            type: 'spline',
            data: confirmedCasesDailyCumm
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
