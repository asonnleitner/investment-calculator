<template>
  <div class="container">
    <form class="md:flex mt-3 shadow-sm rounded-xl border p-4">
      <div class="input-group md:mb-0 mb-4 mr-0 md:mr-4">
        <input
          id="startBalance"
          v-model.lazy="startBalance"
          placeholder="Starting Amount (4500)"
          class="form-control"
          inputmode="numeric"
          type="text"
        />
        <span class="input-group-text" v-text="'$'" />
      </div>

      <div class="input-group md:mb-0 mb-4 mr-0 md:mr-4">
        <input
          id="rateOfReturn"
          v-model.lazy="rateOfReturn"
          placeholder="Rate of Return (20)"
          class="form-control"
          inputmode="numeric"
          type="text"
        />
        <span class="input-group-text" v-text="'%'" />
      </div>

      <div class="input-group md:mb-0 mb-4 mr-0 md:mr-4">
        <input
          id="yearsToGrow"
          v-model.lazy="yearsToGrow"
          placeholder="Years to Grow"
          class="form-control"
          inputmode="numeric"
          type="text"
        />
        <span class="input-group-text" v-text="'Years'" />
      </div>
      <button
        type="button"
        class="btn btn-primary md:mb-0 mb-4 mr-0 md:mr-4"
        @click="onSubmit"
      >
        Calculate
      </button>
    </form>

    <ul v-if="store">
      <li v-for="result in store" :key="result.year">
        <div>
          <span v-text="`Year: ${result.year}`" />
          <span
            v-text="
              `Starting Amount: ${new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
              }).format(result.startBalance)}`
            "
          />
          <span
            v-text="
              `Interest Earned: ${new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
              }).format(result.interestEarned)}`
            "
          />
          <span
            v-text="
              `Total Interest Earned: ${new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
              }).format(result.totalInterestEarned)}`
            "
          />
          <span
            v-text="
              `End Balance: ${new Intl.NumberFormat('en-US', {
                style: 'currency',
                currency: 'USD',
              }).format(result.endBalance)}`
            "
          />
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Index',

  data() {
    return {
      startBalance: '',
      rateOfReturn: '',
      yearsToGrow: '',
      store: [],
      years: [],
    }
  },

  watch: {
    startBalance(value) {
      if (value && this.rateOfReturn) this.initializeStore()
    },

    rateOfReturn(value) {
      if (value && this.startBalance) this.initializeStore()
    },
  },

  methods: {
    initializeStore() {
      this.$emit(
        'store-initialized',
        this.calculate(
          parseFloat(this.startBalance),
          parseFloat(this.rateOfReturn)
        )
      )
    },

    generateYearsRange() {
      const startYear = new Date().getFullYear() + 1
      const endYear = startYear + parseInt(this.yearsToGrow) - 1

      for (let year = startYear; year < endYear; year++) {
        this.years.push(year)
      }
      this.$emit('years-generated', this.years.length)
    },

    calculate(amount, rate, interest, year = new Date().getFullYear()) {
      const interestEarned = (amount / 100) * this.rateOfReturn
      const endBalance = amount + interestEarned
      const totalInterestEarned = interest
        ? endBalance - amount + interest
        : endBalance - amount
      const startBalance = this.startBalance

      this.store.push({
        interest,
        previousEndBalance: amount,
        startBalance,
        interestEarned,
        totalInterestEarned,
        endBalance,
        year,
      })
    },

    onSubmit() {
      this.generateYearsRange()
      this.years.forEach((year, index) => {
        this.calculate(
          this.store[index].endBalance,
          this.rateOfReturn,
          this.store[index].totalInterestEarned,
          year
        )
      })
    },
  },
}
</script>
