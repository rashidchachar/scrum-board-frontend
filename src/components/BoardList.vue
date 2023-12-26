<template>
    <div class="col-md-3 list m-2" ref="listRef">
        <h2>{{ list.title }}</h2>
        <draggable v-model="cards" @end="onCardDragEnd" group="list.cards" :id="list.id">
                <list-card v-for="card in list.cards" :key="card.id" :card="card" :id="card.id" />
        </draggable>
        <input v-model="newCard" class="form-control mb-4" @keyup.enter="addCard" placeholder="Add a card..." />
    </div>
</template>

<script>
    import { VueDraggableNext } from 'vue-draggable-next'
    import ListCard from './listCard.vue';

    export default {
        components: {
            draggable: VueDraggableNext,
            ListCard,
        },
        props: {
            list: Object,
        },
        data() {
            return {
                newCard: '',
                cards: this.list.cards,
            };
        },
        methods: {
            onCardDragEnd(event) {

                const toListID = event.to.id;
                const fromListID = event.from.id;
                const cardId =  event.item.id;
                // this.$emit('update-list-cards', { toListID, fromListID, cardId});
                this.$emit('update-list-cards',toListID,fromListID, cardId);

            },
            addCard() {
                if (this.newCard.trim() !== '') {
                    this.$emit('add-card', this.list.id, this.newCard.trim());
                    this.newCard = '';
                }
            },
        },
    };
</script>

<style scoped>
    .list {
        border: 1px solid #ccc;
        border-radius: 5px;
        background-color: #f9f9f9;
    }

    input {
        margin-top: 5px;
        padding: 5px;
        width: 100%;
    }
</style>
