<template>
    <li>
        {{ todo.text }}
        <button v-on:click="$emit('remove', todo.id)">
            X
        </button>
    </li>
</template>

<script>
    export default {
        name: "TodoListItem",
        props: {
            todo: {
                type: Object,
                required: true
            }
        }
    }
</script>

<style scoped>

</style>
