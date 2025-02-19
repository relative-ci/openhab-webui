<template>
  <f7-stepper ref="stepper" v-bind="config" :value="value" @stepper:change="onChange" @click.native.stop
              :input="config.enableInput === true" :manual-input-mode="false" :format-value="formatValue" />
</template>

<style lang="stylus">
.stepper-value
  margin-top: 0px
</style>

<script>
import mixin from '../widget-mixin'
import variableMixin from '../variable-mixin'
import { OhStepperDefinition } from '@/assets/definitions/widgets/system'

export default {
  mixins: [mixin, variableMixin],
  widget: OhStepperDefinition,
  mounted () {
    delete this.config.value
  },
  computed: {
    value () {
      const applyOffset = (value) => (typeof this.config.offset === 'number') ? value + this.config.offset : value
      if (this.config.variable) {
        if (this.config.variableKey) {
          return applyOffset(this.getLastVariableKeyValue(this.context.vars[this.config.variable], this.config.variableKey))
        }
        return applyOffset(this.context.vars[this.config.variable])
      }
      let value = applyOffset(this.toStepFixed(parseFloat(this.context.store[this.config.item].state)))
      if (this.config.min !== undefined) value = Math.max(value, this.config.min)
      if (this.config.max !== undefined) value = Math.min(value, this.config.max)
      return value
    }
  },
  watch: {
    value (newValue) {
      if (isNaN(newValue) || !isFinite(newValue)) return
      this.$refs.stepper.setValue(this.toStepFixed(newValue).toString())
    }
  },
  methods: {
    formatValue (value) {
      return this.toStepFixed(value)
    },
    toStepFixed (value) {
      // uses the number of decimals in the step config to round the provided number
      if (!this.config.step) return value
      const nbDecimals = Number(this.config.step).toString().replace(',', '.').split('.')[1]
      return parseFloat(Number(value).toFixed(nbDecimals))
    },
    onChange (value) {
      const applyOffset = (value) => (typeof this.config.offset === 'number') ? value - this.config.offset : value
      let newValue = applyOffset(this.toStepFixed(value))
      if (newValue === this.value) return
      if (this.config.variable) {
        if (this.config.variableKey) {
          newValue = applyOffset(this.setVariableKeyValues(this.context.vars[this.config.variable], this.config.variableKey, value))
        }
        this.$set(this.context.vars, this.config.variable, newValue)
      } else if (this.config.item) {
        this.$store.dispatch('sendCommand', { itemName: this.config.item, cmd: newValue.toString() })
      }
    }
  }
}
</script>
