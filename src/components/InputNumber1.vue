<script>
import ElInput from 'element-ui/packages/input'
import Focus from 'element-ui/src/mixins/focus'
import RepeatClick from 'element-ui/src/directives/repeat-click'

export default {
  name: 'ElInputNumber',
  mixins: [Focus('input')],
  inject: {
    elForm: {
      default: ''
    },
    elFormItem: {
      default: ''
    }
  },
  directives: {
    repeatClick: RepeatClick
  },
  components: {
    ElInput
  },
  props: {
    step: {
      type: Number,
      default: 1
    },
    max: {
      type: Number,
      default: Infinity
    },
    min: {
      type: Number,
      default: -Infinity
    },
    value: {},
    disabled: Boolean,
    size: String,
    controls: {
      type: Boolean,
      default: true
    },
    controlsPosition: {
      type: String,
      default: ''
    },
    name: String,
    label: String
  },
  data () {
    return {
      currentValue: 0
    }
  },
  watch: {
    value: {
      immediate: true,
      handler (value) {
        let newVal = value === undefined ? value : Number(value)
        if (newVal !== undefined && isNaN(newVal)) return
        if (newVal >= this.max) newVal = this.max
        if (newVal <= this.min) newVal = this.min
        this.currentValue = newVal
        this.$emit('input', newVal)
      }
    }
  },
  computed: {
    minDisabled () {
      return this._decrease(this.value, this.step) < this.min
    },
    maxDisabled () {
      return this._increase(this.value, this.step) > this.max
    },
    precision () {
      const { value, step, getPrecision } = this
      return Math.max(getPrecision(value), getPrecision(step))
    },
    controlsAtRight () {
      return this.controlsPosition === 'right'
    },
    _elFormItemSize () {
      return (this.elFormItem || {}).elFormItemSize
    },
    inputNumberSize () {
      return this.size || this._elFormItemSize || (this.$ELEMENT || {}).size
    },
    inputNumberDisabled () {
      return this.disabled || (this.elForm || {}).disabled
    }
  },
  methods: {
    toPrecision (num, precision) {
      if (precision === undefined) precision = this.precision
      return parseFloat(parseFloat(Number(num).toFixed(precision)))
    },
    getPrecision (value) {
      if (value === undefined) return 0
      const valueString = value.toString()
      const dotPosition = valueString.indexOf('.')
      let precision = 0
      if (dotPosition !== -1) {
        precision = valueString.length - dotPosition - 1
      }
      return precision
    },
    _increase (val, step) {
      if (typeof val !== 'number' && val !== undefined) return this.currentValue

      const precisionFactor = Math.pow(10, this.precision)
      // Solve the accuracy problem of JS decimal calculation by converting the value to integer.
      return this.toPrecision((precisionFactor * val + precisionFactor * step) / precisionFactor)
    },
    _decrease (val, step) {
      if (typeof val !== 'number' && val !== undefined) return this.currentValue

      const precisionFactor = Math.pow(10, this.precision)

      return this.toPrecision((precisionFactor * val - precisionFactor * step) / precisionFactor)
    },
    increase () {
      if (this.inputNumberDisabled || this.maxDisabled) return
      const value = this.value || 0
      const newVal = this._increase(value, this.step)
      this.setCurrentValue(newVal)
    },
    decrease () {
      if (this.inputNumberDisabled || this.minDisabled) return
      const value = this.value || 0
      const newVal = this._decrease(value, this.step)
      this.setCurrentValue(newVal)
    },
    handleBlur (event) {
      this.$emit('blur', event)
      this.$refs.input.setCurrentValue(this.currentValue)
    },
    handleFocus (event) {
      this.$emit('focus', event)
    },
    setCurrentValue (newVal) {
      const oldVal = this.currentValue
      if (newVal >= this.max) newVal = this.max
      if (newVal <= this.min) newVal = this.min
      if (oldVal === newVal) {
        this.$refs.input.setCurrentValue(this.currentValue)
        return
      }
      this.$emit('change', newVal, oldVal)
      this.$emit('input', newVal)
      this.currentValue = newVal
    },
    handleInputChange (value) {
      const newVal = value === '' ? undefined : Number(value)
      if (!isNaN(newVal) || value === '') {
        this.setCurrentValue(newVal)
      }
    }
  },
  mounted () {
    let innerInput = this.$refs.input.$refs.input
    innerInput.setAttribute('role', 'spinbutton')
    innerInput.setAttribute('aria-valuemax', this.max)
    innerInput.setAttribute('aria-valuemin', this.min)
    innerInput.setAttribute('aria-valuenow', this.currentValue)
    innerInput.setAttribute('aria-disabled', this.inputNumberDisabled)
  },
  updated () {
    let innerInput = this.$refs.input.$refs.input
    innerInput.setAttribute('aria-valuenow', this.currentValue)
  },
  render (h) {
    return (
      <div
        class={{
          'el-input-number': true,
          'is-disabled': this.inputNumberDisabled
        }}>
        <span
          class="el-input-number__decrease"
          role="button"
          v-repeat-click={this.decrease}
          class={{'is-disabled': this.minDisabled}}>
          <i class={`el-icon-${this.controlsAtRight ? 'arrow-down' : 'minus'}`}></i>
        </span>
        <span
          class="el-input-number__increase"
          role="button"
          v-repeat-click={this.increase}
          class={{'is-disabled': this.maxDisabled}}>
          <i class={`el-icon-${this.controlsAtRight ? 'arrow-up' : 'plus'}`}></i>
        </span>
        <el-input
          ref="input"
          value={this.currentValue}
          disabled={this.inputNumberDisabled}
          max={this.max}
          min={this.min}
          name={this.name}
          label={this.label}
          onblur={this.handleBlur}
          onfocus={this.handleFocus}
          onchange={this.handleInputChange}>
        </el-input>
      </div>
    )
  }
}
</script>
