<template>
  <div class="input-form">
    <div class="savings-section">
      <div class="form-group">
        <label for="savings">Eigenkapital (€)</label>
        <input
          type="number"
          id="savings"
          :value="savings"
          @input="handleInput('savings', $event)"
          min="0"
          step="1000"
        />
      </div>
    </div>

    <div class="loan-section">
      <div class="form-group">
        <label for="loanAmount">Benötigte Gesamtsumme (€)</label>
        <input
          type="number"
          id="loanAmount"
          :value="loanAmount"
          @input="handleInput('loanAmount', $event)"
          min="0"
          step="1000"
        />
      </div>

      <div class="form-group">
        <label for="interestRate">Zinssatz (% p.a.)</label>
        <input
          type="number"
          id="interestRate"
          :value="interestRate"
          @input="handleInput('interestRate', $event)"
          min="0"
          step="0.1"
        />
      </div>

      <div class="calculation-mode">
        <label>
          <input
            type="radio"
            :value="'term'"
            :checked="calculationMode === 'term'"
            @change="() => emit('update:calculationMode', 'term')"
          />
          Laufzeit festlegen
        </label>
        <label>
          <input
            type="radio"
            :value="'rate'"
            :checked="calculationMode === 'rate'"
            @change="() => emit('update:calculationMode', 'rate')"
          />
          Rate festlegen
        </label>
      </div>

      <div class="form-group" v-if="calculationMode === 'term'">
        <label for="loanTerm">Laufzeit (Jahre)</label>
        <input
          type="number"
          id="loanTerm"
          :value="loanTerm"
          @input="handleInput('loanTerm', $event)"
          min="1"
          step="1"
        />
      </div>

      <div class="form-group" v-if="calculationMode === 'rate'">
        <label for="monthlyRate">Monatliche Rate (€)</label>
        <input
          type="number"
          id="monthlyRate"
          :value="desiredMonthlyRate"
          @input="handleInput('desiredMonthlyRate', $event)"
          min="0"
          step="10"
        />
      </div>
    </div>

    <h3>Miet-Vergleich</h3>

    <div class="form-group">
      <label for="monthlyRent">Monatliche Miete (€)</label>
      <input
        type="number"
        id="monthlyRent"
        :value="monthlyRent"
        @input="handleInput('monthlyRent', $event)"
        min="0"
        step="10"
      />
    </div>

    <div class="form-group">
      <label for="rentIncrease">Jährliche Mietsteigerung (%)</label>
      <input
        type="number"
        id="rentIncrease"
        :value="rentIncrease"
        @input="handleInput('rentIncrease', $event)"
        min="0"
        step="0.1"
      />
    </div>

    <div class="form-group">
      <label for="investmentReturn">Anlagerendite (% p.a.)</label>
      <input
        type="number"
        id="investmentReturn"
        :value="investmentReturn"
        @input="handleInput('investmentReturn', $event)"
        min="0"
        step="0.1"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { watch } from 'vue'

  const props = defineProps<{
    loanAmount: number
    savings: number
    interestRate: number
    loanTerm: number
    desiredMonthlyRate: number
    calculationMode: string
    monthlyRent: number
    rentIncrease: number
    investmentReturn: number
  }>()

  const emit = defineEmits<{
    (e: 'update:loanAmount', value: number): void
    (e: 'update:savings', value: number): void
    (e: 'update:interestRate', value: number): void
    (e: 'update:loanTerm', value: number): void
    (e: 'update:desiredMonthlyRate', value: number): void
    (e: 'update:calculationMode', value: string): void
    (e: 'update:monthlyRent', value: number): void
    (e: 'update:rentIncrease', value: number): void
    (e: 'update:investmentReturn', value: number): void
    (e: 'calculate'): void
  }>()

  const handleInput = (field: string, event: Event) => {
    const target = event.target as HTMLInputElement
    const value = target.value

    if (field === 'calculationMode') {
      emit('update:calculationMode', value)
    } else {
      const numValue = Number(value)
      if (!isNaN(numValue)) {
        emit(`update:${field}` as keyof typeof emit, numValue)
      }
    }
    emit('calculate')
  }

  // Watch for changes in calculationMode to trigger recalculation
  watch(
    () => props.calculationMode,
    () => {
      emit('calculate')
    }
  )
</script>

<style scoped>
  .input-form {
    display: grid;
    gap: 20px;
  }

  .savings-section {
    background: #e9ecef;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 10px;
  }

  .loan-section {
    display: grid;
    gap: 20px;
  }

  .form-group {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .calculation-mode {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin: 10px 0;
  }

  h3 {
    margin: 20px 0 10px 0;
    color: #2c3e50;
    border-bottom: 1px solid #dee2e6;
    padding-bottom: 5px;
  }

  label {
    font-weight: bold;
    color: #2c3e50;
  }

  input[type='number'] {
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 16px;
    background: white;
  }

  input[type='radio'] {
    margin-right: 8px;
  }
</style>
