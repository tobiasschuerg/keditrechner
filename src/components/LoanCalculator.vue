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

  const loanAmount = ref(300000)
  const savings = ref(80000)
  const interestRate = ref(3.5)
  const loanTerm = ref(15)
  const desiredMonthlyRate = ref(1000)
  const calculationMode = ref('term')
  const monthlyRent = ref(1000)
  const rentIncrease = ref(0.5)
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

  function calculateLoan() {
    const monthlyRate = interestRate.value / 100 / 12
    const monthlyInvestmentRate = investmentReturn.value / 100 / 12
    let numberOfPayments: number

    // Adjust loan amount by subtracting savings
    const actualLoanAmount = Math.max(0, loanAmount.value - savings.value)

    if (calculationMode.value === 'rate') {
      // Calculate term using the same formula as in LoanInputForm
      const monthlyPaymentAmount = desiredMonthlyRate.value

      // Handle edge cases
      if (actualLoanAmount <= 0 || monthlyPaymentAmount <= 0) {
        numberOfPayments = 0
      } else {
        const minimumPayment = actualLoanAmount * monthlyRate
        if (monthlyPaymentAmount <= minimumPayment) {
          // If payment doesn't cover interest, cap at 50 years
          numberOfPayments = 600 // 50 years * 12 months
        } else {
          const months =
            Math.log(
              monthlyPaymentAmount / (monthlyPaymentAmount - monthlyRate * actualLoanAmount)
            ) / Math.log(1 + monthlyRate)
          numberOfPayments = Math.min(600, Math.max(0, Math.ceil(months)))
        }
      }
      monthlyPayment.value = desiredMonthlyRate.value
    } else {
      numberOfPayments = loanTerm.value * 12
      monthlyPayment.value =
        actualLoanAmount > 0
          ? (actualLoanAmount * monthlyRate * Math.pow(1 + monthlyRate, numberOfPayments)) /
            (Math.pow(1 + monthlyRate, numberOfPayments) - 1)
          : 0
    }

    calculatedLoanTerm.value = numberOfPayments / 12

    // Loan calculation data
    const labels = []
    const remainingLoan = []
    const totalInterest = []
    const totalPrincipal = []
    const accumulatedRent = []
    const potentialInvestment = []

    // Rent and investment comparison data
    let currentRent = monthlyRent.value
    let currentLoan = actualLoanAmount
    let totalInterestPaid = 0
    let totalPrincipalPaid = 0
    let totalRentPaid = 0
    let investmentValue = savings.value // Start with initial savings

    // Add initial point
    labels.push('0 Jahre')
    remainingLoan.push(actualLoanAmount)
    totalInterest.push(0)
    totalPrincipal.push(0)
    accumulatedRent.push(0)
    potentialInvestment.push(savings.value)

    // Calculate data points for every 6 months
    const interval = 6
    let loanPaidOffMonth = null
    for (let month = 0; month < numberOfPayments; month += interval) {
      const currentInterval = Math.min(interval, numberOfPayments - month)

      // Calculate loan payments
      for (let i = 0; i < currentInterval; i++) {
        if (currentLoan > 0) {
          const interestPayment = currentLoan * monthlyRate
          const principalPayment = Math.min(monthlyPayment.value - interestPayment, currentLoan)

          currentLoan = Math.max(0, currentLoan - principalPayment)
          totalInterestPaid += interestPayment
          totalPrincipalPaid += principalPayment

          if (currentLoan === 0 && loanPaidOffMonth === null) {
            loanPaidOffMonth = month + i
          }
        }

        // Calculate rent and investment scenario - continue even after loan is paid off
        totalRentPaid += currentRent
        // If loan is paid off, entire monthly payment goes to investment
        const monthlyInvestmentAmount =
          currentLoan === 0 ? monthlyPayment.value : monthlyPayment.value - currentRent
        investmentValue = (investmentValue + monthlyInvestmentAmount) * (1 + monthlyInvestmentRate)

        // Update rent every 12 months
        if ((month + i + 1) % 12 === 0) {
          currentRent *= 1 + rentIncrease.value / 100
        }
      }

      if (month + interval <= numberOfPayments) {
        labels.push((month + interval) / 12 + ' Jahre')
        remainingLoan.push(currentLoan)
        totalInterest.push(totalInterestPaid)
        totalPrincipal.push(totalPrincipalPaid)
        accumulatedRent.push(totalRentPaid)
        potentialInvestment.push(investmentValue)
      }
    }

    // Always add the final point at exactly the loan term
    if (numberOfPayments % interval !== 0) {
      labels.push(numberOfPayments / 12 + ' Jahre')
      remainingLoan.push(currentLoan)
      totalInterest.push(totalInterestPaid)
      totalPrincipal.push(totalPrincipalPaid)
      accumulatedRent.push(totalRentPaid)
      potentialInvestment.push(investmentValue)
    }

    totalCost.value = totalInterestPaid + totalPrincipalPaid
    totalRentCost.value = totalRentPaid
    totalInvestmentValue.value = investmentValue

    // Define colors for consistency
    const colors = {
      principal: {
        border: 'rgb(75, 192, 192)',
        background: 'rgba(75, 192, 192, 0.1)',
      },
      loanPayments: {
        border: 'rgb(128, 128, 128)',
        background: 'rgba(128, 128, 128, 0.1)',
      },
      interest: {
        border: 'rgb(255, 99, 132)',
        background: 'rgba(255, 99, 132, 0.1)',
      },
      remainingLoan: {
        border: 'rgb(54, 162, 235)',
        background: 'rgba(54, 162, 235, 0.1)',
      },
      rent: {
        border: 'rgb(255, 159, 64)',
        background: 'rgba(255, 159, 64, 0.1)',
      },
      investment: {
        border: 'rgb(153, 102, 255)',
        background: 'rgba(153, 102, 255, 0.1)',
      },
    }

    // Update loan progress chart
    chartData.value = {
      labels,
      datasets: [
        {
          label: 'Kreditrate (kumuliert)',
          data: labels.map((_, index) => {
            if (index === 0) return 0
            const monthsPassed = Math.min(numberOfPayments, index * interval)
            return monthlyPayment.value * monthsPassed
          }),
          borderColor: colors.loanPayments.border,
          backgroundColor: colors.loanPayments.background,
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Restschuld',
          data: remainingLoan,
          borderColor: colors.remainingLoan.border,
          backgroundColor: colors.remainingLoan.background,
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Gezahlte Zinsen',
          data: totalInterest,
          borderColor: colors.interest.border,
          backgroundColor: colors.interest.background,
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Getilgte Summe',
          data: totalPrincipal,
          borderColor: colors.principal.border,
          backgroundColor: colors.principal.background,
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
          borderColor: colors.loanPayments.border,
          backgroundColor: colors.loanPayments.background,
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Miete (kumuliert)',
          data: accumulatedRent,
          borderColor: colors.rent.border,
          backgroundColor: colors.rent.background,
          tension: 0.1,
          fill: true,
        },
        {
          label: 'Investment der Differenz',
          data: potentialInvestment,
          borderColor: colors.investment.border,
          backgroundColor: colors.investment.background,
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
