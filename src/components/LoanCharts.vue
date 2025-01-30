<template>
  <div class="graphs-panel">
    <div class="graph-container">
      <div class="title-container">
        <h3>Kreditverlauf</h3>
        <span class="tooltip-trigger"
          >?
          <span class="tooltip-text"
            >Zeigt die Entwicklung des Kreditbetrags über die gesamte Laufzeit, einschließlich der
            Zinszahlungen und Tilgung.</span
          >
        </span>
      </div>
      <Line :data="chartData" :options="chartOptions" />
    </div>

    <div class="graph-container">
      <div class="title-container">
        <h3>Mieten und Anlegen</h3>
        <span class="tooltip-trigger"
          >?
          <span class="tooltip-text"
            >Vergleicht die finanzielle Entwicklung zwischen dem Kauf mit Kredit und der Alternative
            von Mieten mit Geldanlage. Dabei wird der gleiche monatliche Betrag wie bei der
            Kreditrate angesetzt, wobei die Differenz zur Miete angelegt wird.</span
          >
        </span>
      </div>
      <Line :data="comparisonChartData" :options="chartOptions" />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { Line } from 'vue-chartjs'
  import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend,
  } from 'chart.js'

  ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend)

  defineProps<{
    chartData: {
      labels: string[]
      datasets: Array<{
        label: string
        data: number[]
        borderColor: string
        backgroundColor: string
        tension: number
        fill: boolean
      }>
    }
    comparisonChartData: {
      labels: string[]
      datasets: Array<{
        label: string
        data: number[]
        borderColor: string
        backgroundColor: string
        tension: number
        fill: boolean
      }>
    }
  }>()

  const chartOptions = {
    responsive: true,
    maintainAspectRatio: false,
    plugins: {
      legend: {
        position: 'top' as const,
      },
      tooltip: {
        callbacks: {
          label: function (context: any) {
            return context.dataset.label + ': ' + context.parsed.y.toFixed(2) + ' €'
          },
        },
      },
    },
    scales: {
      y: {
        beginAtZero: true,
        ticks: {
          callback: function (value: any) {
            return value.toLocaleString() + ' €'
          },
        },
      },
    },
  }
</script>

<style scoped>
  .graphs-panel {
    display: flex;
    flex-direction: column;
    gap: 50px;
  }

  .graph-container {
    background: white;
    padding: 20px 20px 60px 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    height: 400px;
  }

  .graph-container h3 {
    margin: 0;
    padding: 0;
  }

  .title-container {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-bottom: 20px;
  }

  .tooltip-trigger {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background-color: #e0e0e0;
    font-size: 12px;
    cursor: help;
    position: relative;
  }

  .tooltip-trigger:hover .tooltip-text {
    visibility: visible;
    opacity: 1;
  }

  .tooltip-text {
    visibility: hidden;
    opacity: 0;
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    padding: 8px 12px;
    background-color: #333;
    color: white;
    border-radius: 4px;
    font-size: 12px;
    white-space: normal;
    z-index: 1;
    transition: opacity 0.2s;
    width: 200px;
    text-align: center;
    margin-bottom: 8px;
  }

  .tooltip-text::after {
    content: '';
    position: absolute;
    top: 100%;
    left: 50%;
    margin-left: -5px;
    border-width: 5px;
    border-style: solid;
    border-color: #333 transparent transparent transparent;
  }

  @media (max-width: 768px) {
    .graph-container {
      height: 300px;
    }
  }
</style>
