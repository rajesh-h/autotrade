<template id="custominput">
  <input
    type="text"
    class="form-control"
    :value="active ? val : formatted"
    @keydown.up.prevent="increment"
    @keydown.down.prevent="decrement"
    @blur="update"
    @keyup.enter.stop="update"
    @focus="active = true"
  />
</template>

<script>
function formatAsCurrency(value, dec) {
  dec = dec || 0
  return 'R ' + value.toFixed(dec).replace(/(\d)(?=(\d\d\d)+(?!\d))/g, '$1,')
}

export default {
  name: 'CalculationImport',
  props: {
    value: { type: Number, default: 0 },
    min: { type: Number, default: 0 },
    max: { type: Number, default: 100000000 },
    step: { type: Number, default: 10000 },
    decimals: { type: Number, default: 0 },
    type: { type: String, default: 'currency' },
  },
  data() {
    return {
      active: false,
      newvalue: null,
    }
  },
  computed: {
    val() {
      if (this.type === 'currency') {
        return Number(this.value) + ''
      } else if (this.type === 'years') {
        return Number(this.value) + ''
      } else if (this.type === 'percent') {
        return Number(this.value * 100).toFixed(3)
      } else {
        return ''
      }
    },
    formatted() {
      if (this.type === 'currency') {
        return formatAsCurrency(this.value, 0)
      } else if (this.type === 'years') {
        return Number(this.value + ''.replace(/[^0-9.]+/g, '')) + ' Years'
      } else if (this.type === 'percent') {
        return (this.value * 100).toFixed(this.decimals) + '%'
      } else {
        return ''
      }
    },
  },
  mounted() {
    this.newvalue = this.value
  },
  methods: {
    increment(e) {
      if (e.shiftKey) {
        this.newvalue += 10 * this.step
      } else {
        this.newvalue += this.step
      }
      if (this.value > this.max) {
        this.newvalue = this.max
      }
      this.changed()
    },
    decrement(e) {
      if (e.shiftKey) {
        this.newvalue -= 10 * this.step
      } else {
        this.newvalue -= this.step
      }
      if (this.value < this.min) {
        this.newvalue = this.min
      }
      this.changed()
    },
    update() {
      this.active = false
      const tempVal = this.$el.value + ''
      this.newvalue = Number(tempVal.replace(/[^0-9.]+/g, ''))
      if (this.type === 'percent') this.newvalue /= 100
      this.changed()
    },
    changed() {
      this.$emit('input', Number(this.newvalue))
    },
  },
}
</script>

<style></style>
