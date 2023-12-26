<template>
    <div class="card">
        {{ card.title }}
    </div>
</template>

<script>
    export default {
        props: ['card'],
    };
</script>

<style scoped>
    .card {
        margin: 5px 0;
        padding: 10px;
        border: 1px solid #ddd;
        border-radius: 5px;
        background-color: #fff;
        cursor: grab;
    }
</style>
