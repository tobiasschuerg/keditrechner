`
<template>
  <div class="summary-info">
    <div class="info-item">
      <span>Monatliche Rate:</span>
      <strong>{{ monthlyPayment.toFixed(2) }}€</strong>
    </div>
    <div class="info-item">
      <span>Laufzeit:</span>
      <strong>{{ calculatedLoanTerm }} Jahre</strong>
    </div>
    <div class="info-item">
      <span>Gesamtkosten Kredit:</span>
      <strong>{{ totalCost.toFixed(2) }}€</strong>
    </div>
    <div class="info-item">
      <span>Gesamtkosten Miete:</span>
      <strong>{{ totalRentCost.toFixed(2) }}€</strong>
    </div>
    <div class="info-item">
      <span>Mögliches Investment:</span>
      <strong>{{ totalInvestmentValue.toFixed(2) }}€</strong>
    </div>
  </div>
</template>

<script setup lang="ts">
  defineProps<{
    monthlyPayment: number
    calculatedLoanTerm: number
    totalCost: number
    totalRentCost: number
    totalInvestmentValue: number
  }>()
</script>

<style scoped>
  .summary-info {
    margin-top: 20px;
    padding: 15px;
    background: white;
    border-radius: 4px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  }

  .info-item {
    display: flex;
    justify-content: space-between;
    margin: 8px 0;
    font-size: 14px;
  }

  .info-item strong {
    color: #2c3e50;
  }
</style>
`
