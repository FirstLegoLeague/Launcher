<template>
    <div class="three wide field">
        <label>{{ field.display || field.name }}:</label>
        <input
            type="number"
            :name="field.name"
            :value="value"
            :min="field.min"
            :max="field.max"
            @change="$emit('value-change', $event.target.value)"
        />
    </div>
</template>

<script>
  export default {
    name: 'decimal-field',
    props: ['field', 'value']
  }
</script>

<style scoped>

</style>
