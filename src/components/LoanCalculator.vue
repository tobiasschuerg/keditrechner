<template>
  <div class="loan-calculator">
    <div class="calculator-layout">
      <div class="input-panel">
        <h2>Kreditdetails</h2>

        <LoanInputForm
          :loanAmount="loanAmount"
          :savings="savings"
          :interestRate="interestRate"
          :loanTerm="loanTerm"
          :desiredMonthlyRate="desiredMonthlyRate"
          :calculationMode="calculationMode"
          :monthlyRent="monthlyRent"
          :rentIncrease="rentIncrease"
          :investmentReturn="investmentReturn"
          @update:loanAmount="loanAmount = $event"
          @update:savings="savings = $event"
          @update:interestRate="interestRate = $event"
          @update:loanTerm="loanTerm = $event"
          @update:desiredMonthlyRate="desiredMonthlyRate = $event"
          @update:calculationMode="calculationMode = $event"
          @update:monthlyRent="monthlyRent = $event"
          @update:rentIncrease="rentIncrease = $event"
          @update:investmentReturn="investmentReturn = $event"
          @calculate="calculateLoan"
        />

        <LoanSummary
          v-if="showResults"
          :monthlyPayment="monthlyPayment"
          :calculatedLoanTerm="calculatedLoanTerm"
          :totalCost="totalCost"
          :totalRentCost="totalRentCost"
          :totalInvestmentValue="totalInvestmentValue"
        />
      </div>

      <LoanCharts
        v-if="showResults"
        :chartData="chartData"
        :comparisonChartData="comparisonChartData"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, onMounted } from 'vue'
  import LoanInputForm from './LoanInputForm.vue'
  import LoanSummary from './LoanSummary.vue'
  import LoanCharts from './LoanCharts.vue'

  interface ChartDataset {
    label: string
    data: number[]
    borderColor: string
    backgroundColor: string
    tension: number
    fill: boolean
  }

  interface ChartData {
    labels: string[]
    datasets: ChartDataset[]
  }

  const loanAmount = ref(400000)
  const savings = ref(50000)
  const interestRate = ref(3.5)
  const loanTerm = ref(20)
  const desiredMonthlyRate = ref(1000)
  const calculationMode = ref('term')
  const monthlyRent = ref(1000)
  const rentIncrease = ref(1)
  const investmentReturn = ref(6)
  const showResults = ref(false)
  const monthlyPayment = ref(0)
  const calculatedLoanTerm = ref(0)
  const totalCost = ref(0)
  const totalRentCost = ref(0)
  const totalInvestmentValue = ref(0)
  const chartData = ref<ChartData>({
    labels: [],
    datasets: [],
  })
  const comparisonChartData = ref<ChartData>({
    labels: [],
    datasets: [],
  })

  function calculateTermFromRate(monthlyRate: number): number {
    const monthlyInterestRate = interestRate.value / 100 / 12
    const numerator =
      Math.log(monthlyRate) - Math.log(monthlyRate - loanAmount.value * monthlyInterestRate)
    const denominator = Math.log(1 + monthlyInterestRate)
    return Math.ceil(numerator / denominator / 12)
  }

  function calculateLoan() {
    const monthlyRate = interestRate.value / 100 / 12
    const monthlyInvestmentRate = investmentReturn.value / 100 / 12
    let numberOfPayments: number

    // Adjust loan amount by subtracting savings
    const actualLoanAmount = Math.max(0, loanAmount.value - savings.value)

    if (calculationMode.value === 'rate') {
      const calculatedTerm = calculateTermFromRate(desiredMonthlyRate.value)
      loanTerm.value = calculatedTerm
      numberOfPayments = calculatedTerm * 12
      monthlyPayment.value = desiredMonthlyRate.value
    } else {
      numberOfPayments = loanTerm.value * 12
      monthlyPayment.value =
        actualLoanAmount > 0
          ? (actualLoanAmount * monthlyRate * Math.pow(1 + monthlyRate, numberOfPayments)) /
            (Math.pow(1 + monthlyRate, numberOfPayments) - 1)
          : 0
    }

    calculatedLoanTerm.value = loanTerm.value

    // Loan calculation data
    const labels = ['0 Jahre']
    const remainingLoan = [actualLoanAmount]
    const totalInterest = [0]
    const totalPrincipal = [0]

    // Rent and investment comparison data
    const accumulatedRent = [0]
    const potentialInvestment = [savings.value] // Start with initial savings
    let currentRent = monthlyRent.value
    let currentLoan = actualLoanAmount
    let totalInterestPaid = 0
    let totalPrincipalPaid = 0
    let totalRentPaid = 0
    let investmentValue = savings.value // Start with initial savings

    // Calculate data points for every 6 months
    const interval = 6
    for (let month = interval; month <= numberOfPayments; month += interval) {
      // Calculate loan payments
      for (let i = 0; i < interval && month + i <= numberOfPayments; i++) {
        const interestPayment = currentLoan * monthlyRate
        const principalPayment = Math.min(monthlyPayment.value - interestPayment, currentLoan)

        currentLoan = Math.max(0, currentLoan - principalPayment)
        totalInterestPaid += interestPayment
        totalPrincipalPaid += principalPayment

        // Calculate rent and investment scenario
        totalRentPaid += currentRent
        // Take the fixed loan rate and subtract the current rent to get investment amount
        const monthlyInvestmentAmount = monthlyPayment.value - currentRent
        investmentValue = (investmentValue + monthlyInvestmentAmount) * (1 + monthlyInvestmentRate)

        // Update rent every 12 months
        if ((month + i + 1) % 12 === 0) {
          currentRent *= 1 + rentIncrease.value / 100
        }

        if (currentLoan === 0) break
      }

      labels.push(month / 12 + ' Jahre')
      remainingLoan.push(currentLoan)
      totalInterest.push(totalInterestPaid)
      totalPrincipal.push(totalPrincipalPaid)
      accumulatedRent.push(totalRentPaid)
      potentialInvestment.push(investmentValue)

      if (currentLoan === 0) break
    }

    // Add final point if not already added
    if (currentLoan > 0) {
      labels.push((numberOfPayments / 12).toFixed(1) + ' Jahre')
      remainingLoan.push(0)
      totalInterest.push(totalInterestPaid)
      totalPrincipal.push(totalPrincipalPaid + currentLoan)
      accumulatedRent.push(totalRentPaid)
      potentialInvestment.push(investmentValue)
    }

    totalCost.value = totalInterestPaid + totalPrincipalPaid
    totalRentCost.value = totalRentPaid
    totalInvestmentValue.value = investmentValue

    // Update loan progress chart
    chartData.value = {
      labels,
      datasets: [
        {
          label: 'Restschuld',
          data: remainingLoan,
          borderColor: 'rgb(75, 192, 192)',
          backgroundColor: 'rgba(75, 192, 192, 0.1)',
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Gezahlte Zinsen',
          data: totalInterest,
          borderColor: 'rgb(255, 99, 132)',
          backgroundColor: 'rgba(255, 99, 132, 0.1)',
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Getilgte Summe',
          data: totalPrincipal,
          borderColor: 'rgb(54, 162, 235)',
          backgroundColor: 'rgba(54, 162, 235, 0.1)',
          tension: 0.1,
          fill: true,
        },
      ],
    }

    // Calculate total loan payments for comparison
    const totalLoanPayments = labels.map((_, index) => {
      if (index === 0) return 0
      const monthsPassed = Math.min(numberOfPayments, index * interval)
      return monthlyPayment.value * monthsPassed
    })

    // Update comparison chart
    comparisonChartData.value = {
      labels,
      datasets: [
        {
          label: 'Kreditraten (kumuliert)',
          data: totalLoanPayments,
          borderColor: 'rgb(75, 192, 192)',
          backgroundColor: 'rgba(75, 192, 192, 0.1)',
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Miete (kumuliert)',
          data: accumulatedRent,
          borderColor: 'rgb(255, 159, 64)',
          backgroundColor: 'rgba(255, 159, 64, 0.1)',
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Investment der Differenz',
          data: potentialInvestment,
          borderColor: 'rgb(153, 102, 255)',
          backgroundColor: 'rgba(153, 102, 255, 0.1)',
          tension: 0.1,
          fill: true,
        },
      ],
    }

    showResults.value = true
  }

  // Trigger initial calculation
  onMounted(() => {
    calculateLoan()
  })
</script>

<style scoped>
  .loan-calculator {
    width: 100%;
    padding: 20px;
  }

  .calculator-layout {
    display: grid;
    grid-template-columns: 300px 1fr;
    gap: 30px;
    align-items: start;
  }

  .input-panel {
    background: #f8f9fa;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  h2 {
    margin: 0 0 20px 0;
    color: #2c3e50;
  }

  @media (max-width: 768px) {
    .calculator-layout {
      grid-template-columns: 1fr;
    }
  }
</style>
