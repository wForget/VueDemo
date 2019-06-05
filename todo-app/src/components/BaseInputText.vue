<template>
    <input
            type="text"
            class="input"
            v-bind:value="value"
            v-on="listeners"
    >
</template>

<script>
    export default {
        name: "BaseInputText",
        props: {
            value: {
                type: String,
                default: ''
            }
        },
        computed: {
            listeners() {
                return {
                    // Pass all component listeners directly to input
                    ...this.$listeners,
                    // Override input listener to work with v-model
                    input: event => this.$emit('input', event.target.value)
                }
            }
        }
    }
</script>

<style scoped>

    .input {
        width: 100%;
        padding: 8px 10px;
        border: 1px solid #32485F;
    }

</style>
