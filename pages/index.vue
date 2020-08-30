<template>
  <div id="app">
    <h2>Auto Loan Calculator</h2>
    <div class="form">
      <div class="form-group">
        <label>Vehicle Value: </label>
        <CalculationInput v-model="homeValue" :step="1000" />
      </div>
      <div class="form-group">
        <label>Down Payment: </label>
        <CalculationInput v-model="downpayment" :step="1000" />
      </div>
      <div class="form-group">
        <label>Loan Period: </label>
        <CalculationInput v-model="amortization" :step="1" type="years" />
      </div>
      <div class="form-group">
        <label>Interest Rate: </label>
        <CalculationInput
          v-model="interestRate"
          :step="0.001"
          type="percent"
          :decimals="3"
        />
      </div>
      <div class="form-group">
        <label>Payment Frequency: </label>
        <select v-model="paymentSelection" class="form-control">
          <option
            v-for="(paymen, key) in paymentPeriod"
            :key="key"
            :value="key"
          >
            {{ key }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label>Total cost of loan:</label>
        <span class="">{{ formatAsCurrency(totalCostOfMortgage) }}</span>
      </div>

      <div class="form-group">
        <label>Total Interest Paid:</label>
        <span class="">{{ formatAsCurrency(interestPayed) }}</span>
      </div>

      <div class="form-group">
        <label>{{ paymentSelection }} Payment:</label>
        <span class="out">{{ formatAsCurrency(payment, 2) }}</span>
      </div>

      <h3>Loan Schedule</h3>
      <svg viewBox="0 0 560 300">
        <g class="xaxis">
          <line x1="0" y1="1" x2="500" y2="1" />
          <g v-for="index in amortization" :key="index">
            <line
              y1="0"
              :y2="index % 5 === 0 ? 10 : 5"
              v-bind="{
                x1: (index * 500) / amortization,
                x2: (index * 500) / amortization,
              }"
            />
            <text
              v-if="index % (amortization < 55 ? 5 : 10) === 0"
              v-bind="{ x: ((index - 0.4) * 500) / amortization, y: 30 }"
            >
              {{ index }}
            </text>
          </g>
        </g>

        <g class="graph">
          <rect
            v-for="(p, idx) in amortizationGraphBars(500, 250)"
            :key="idx"
            v-bind="p"
            @mouseover="setHover(p, $event)"
            @mouseleave="graphSelection.visible = false"
          ></rect>
        </g>
      </svg>
      <div class="note">
        * amount of principal remaining at the end of a year
      </div>
    </div>

    <div
      v-if="graphSelection !== null"
      v-show="graphSelection.visible === true"
      class="hover"
      :style="graphSelection.style"
    >
      Year: {{ graphSelection.year }}<br />
      Principal: {{ graphSelection.principal }} <br />
      Remaining: {{ graphSelection.principalPercent }} <br />
    </div>
  </div>
</template>

<script>
// import CalculationInput from './components/CalculationInput.vue'
function formatAsCurrency(value, dec) {
  dec = dec || 0
  return 'R ' + value.toFixed(dec).replace(/(\d)(?=(\d\d\d)+(?!\d))/g, '$1,')
}
export default {
  name: 'App',
  components: {
    // CalculationInput,
  },
  data() {
    return {
      homeValue: 350000,
      downpayment: 0,
      interestRate: 0.055,
      amortization: 5,
      paymentPeriod: {
        Monthly: { npy: 12 },
        'Semi-Monthly': { npy: 12 * 2 },
        'Bi-Weekly': { npy: 365.25 / 7 / 2 },
        Weekly: { npy: 365.25 / 7 },
      },
      paymentSelection: 'Monthly',
      graphSelection: null,
    }
  },
  computed: {
    principal() {
      return this.homeValue - this.downpayment
    },
    numPayments() {
      return this.amortization * this.numPaymentsPerYear
    },
    numPaymentsPerYear() {
      return this.paymentPeriod[this.paymentSelection].npy
    },
    interestPerPayment() {
      // eslint-disable-next-line no-console
      console.log(this.interestRate / this.numPaymentsPerYear)
      return this.interestRate / this.numPaymentsPerYear
    },
    payment() {
      const temp = Math.pow(1 + this.interestPerPayment, this.numPayments)
      const p = (this.principal * this.interestPerPayment * temp) / (temp - 1)
      return p
    },
    amortizationPayments() {
      const yearEndPrincipal = []
      let principal = this.principal
      let interestPortion
      for (let y = 0; y < this.amortization; y++) {
        for (let p = 0; p < this.numPaymentsPerYear; p++) {
          interestPortion = principal * this.interestPerPayment // portion of payment paying down interest
          principal = principal - (this.payment - interestPortion)
        }
        principal = principal > 0 ? principal : 0
        yearEndPrincipal.push({
          principal,
          interestPortion,
        })
      }
      return yearEndPrincipal
    },
    totalCostOfMortgage() {
      return this.payment * this.numPayments
    },
    interestPayed() {
      return this.totalCostOfMortgage - this.principal
    },
  },
  methods: {
    formatAsCurrency,
    amortizationGraphBars(width, height) {
      const bars = []
      const spacing = 0
      let i, p, x, y, w, h
      for (i in this.amortizationPayments) {
        p = this.amortizationPayments[i].principal
        w = width / this.amortization - spacing
        x = parseInt(i) * (w + spacing)
        w -= 1
        h = (p * height) / this.principal
        h = h > 3 ? h : 3
        y = height - h

        bars.push({
          x,
          y,
          width: w,
          height: h,
          'data-p': p,
          'data-ip': this.amortizationPayments[i].interestPortion,
          'data-year': parseInt(i) + 1,
        })
      }
      return bars
    },
    setHover(p, e) {
      this.graphSelection = {
        style: {
          left: e.clientX + 20 + 'px',
          top: e.clientY - 25 + 'px',
        },
        year: p['data-year'],
        principal: formatAsCurrency(p['data-p']),
        principalPercent:
          ((p['data-p'] / this.principal) * 100).toFixed(1) + '%',
        visible: true,
      }
    },
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Patua+One');
$col0: #ffffff;
$col1: #d9e3f0;
$col2: teal;
$col3: #263238;
$chartHeight: 300px;
$chartWidth: 560px;

#app {
  width: 500px;
  background: $col2;
  padding: 0;
  margin: 0 auto;
  top: 10px;
  position: relative;
  font-family: 'Patua One', serif;
  color: $col2;
  border-radius: 4px;
  box-shadow: rgba(0, 0, 0, 0.3) 2px 2px 10px;

  h2 {
    text-align: center;
    font-size: 32px;
    margin: 20px 0 20px 0;
    padding-top: 20px;
    color: $col1;
    text-shadow: rgba(0, 0, 0, 0.5) 2px 2px 10px;
  }

  h3 {
    text-align: center;
    font-size: 24px;
    margin: 16px 0 20px 0;
    color: $col2;
  }
  .form {
    margin: -30px 20px 20px 20px;
    display: block;
    background: $col0;
    padding: 18px;
    border-radius: 4px 4px 0 0;
    transform: translateY(30px);
    box-shadow: rgba(0, 0, 0, 0.3) 2px 2px 18px;

    .form-group {
      width: 100%;
      position: relative;
      box-sizing: content-box;
      font-size: 18px;
      display: block;

      label {
        width: 49%;
        display: inline-block;
        text-align: right;
        box-sizing: content-box;
        line-height: 20px;
        padding: 10px;
      }

      .out {
        font-size: 26px;
        color: $col3;
      }

      input,
      select {
        width: 35%;
        display: inline-block;
        box-sizing: content-box;
        font-size: 18px;
        padding: 6px 8px 2px 2px;
        border: none;
        border-bottom: 3px solid $col1;
        background: none;
        outline: none;
        border-radius: 2px 2px 0 0;
        color: $col3;

        &:active,
        &:focus {
          background: lighten($col1, 5%);
        }

        option {
          background: $col0;

          &:active,
          &:hover,
          &:focus {
            background: $col1;
          }
        }
      }
    }
  }

  .hover {
    position: fixed;
    display: block;
    padding: 8px;
    font-size: 16px;
    color: $col2;
    transition: all 0.1s;
    background: $col0;
    pointer-events: none;
    border: 1px solid $col3;
    box-shadow: rgba(0, 0, 0, 0.3) 2px 2px 10px;
  }
  .note {
    text-align: center;
    font-size: 0.7em;
  }
  svg {
    text {
      fill: $col2;
      font-family: 'Mada', sans-serif;
    }

    line {
      stroke: $col2;
      stroke-width: 2px;
    }

    .xaxis {
      transform: translate(20px, $chartHeight - 50px);
    }

    .graph {
      transform: translate(20px, 0px);

      rect {
        fill: $col1;

        &:hover {
          fill: $col2;
        }
      }
    }
  }

  .yaxis {
    transform: translate(20px, 0px) rotate(90deg);
  }
}
</style>
