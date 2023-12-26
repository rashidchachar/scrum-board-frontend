<template>
  <div class="container">
    <h1 class="text-center">Rashid's Board</h1>
    <div class="col-md-3 ms-4">
      <input v-model="newList" class="form-control w-100" @keyup.enter="addList" placeholder="Add a list..." />
    </div>
    <draggable v-model="lists" @end="onListDragEnd"  class="row lists-section p-4" group="lists">
      <div class="col-md-3 list m-2" ref="listRef" v-for="list in lists" :key="list.id">
        <h2>{{ list.title }}</h2>
        <draggable v-model="list.cards" @end="onCardDragEnd" group="list.cards" :id="list.id">
          <list-card v-for="card in list.cards" :key="card.id" :card="card" :id="card.id" />
        </draggable>
        <input v-model="list.newCard" class="form-control mb-4" @keyup.enter="addCard(list.id)" placeholder="Add a card" />
      </div>
<!--      <board-list v-for="list in lists" :key="list.id" :list="list" @add-card="addCard" @update-list-cards="updateListCards" />-->
    </draggable>
  </div>
</template>

<script>
  import { VueDraggableNext } from 'vue-draggable-next'
  // import BoardList from './components/BoardList.vue';
  import ListCard from './components/listCard.vue';
  import axios from 'axios'

  export default {
    name: 'App',
    components: {
      draggable: VueDraggableNext,
      // BoardList,
      ListCard,
    },
    data() {
      return {
        lists: [],
        newList: '',
        newCard: '',
      };
    },
    mounted() {
      this.fetchLists();
    },
    methods: {
      async fetchLists() {
        try {
          const response = await axios.get('http://localhost:8000/api/lists');
          this.lists = response.data;
        } catch (error) {
          console.error('Error fetching lists:', error);
        }
      },
      onListDragEnd() {
        const newOrder = this.lists.map((list) => list.id);

        console.log('List Order:', this.lists.map((list) => list.id));
        try {
          axios.post('http://localhost:8000/api/lists/sort', { listOrder: newOrder });
          console.log('List order saved successfully.');
        } catch (error) {
          console.error('Error saving list order:', error);
        }
      },
      async addList() {
        if (this.newList.trim() !== '') {
          try {
            const response = await axios.post('http://localhost:8000/api/lists', {
              title: this.newList.trim(),
            });
            this.lists.push(response.data);
            this.newList = '';
          } catch (error) {
            console.error('Error adding list:', error);
          }
        }
      },
      async addCard(listId) {
        const list = this.lists.find((list) => list.id === listId);
        if (list.newCard.trim() !== '') {
          try {
            const response = await axios.post('http://localhost:8000/api/cards', {
              list_id: listId,
              title: list.newCard.trim(),
            });
            list.cards.push(response.data);
            list.newCard = '';
          } catch (error) {
            console.error('Error adding card:', error);
          }
        }
      },
      onCardDragEnd(event) {

        const toListId = event.to.id;
        const fromListId = event.from.id;

        let cardsOrder = [];
        const toList = this.lists.find((list) => list.id == toListId);
        const fromList = this.lists.find((list) => list.id == fromListId);
        toList.cards.forEach((item, index) => {
          cardsOrder.push({'list_id': toListId,'card_id':item.id,'order':index})
        })
        if(toListId != fromListId){
          fromList.cards.forEach((item, index) => {
            cardsOrder.push({'list_id': fromListId,'card_id':item.id,'order':index})
          })
        }
        console.log('toList:',toList)
        console.log('fromList:',fromList)
        console.log('cardsOrder:',cardsOrder)

        try {
          axios.post('http://localhost:8000/api/cards/sort', {
            cardsOrder: cardsOrder,
          });
        } catch (error) {
          console.error('Error adding card:', error);
        }
      },
    },
  };
</script>

<style>
  /* Add your styling here */
</style>

<style scoped>
  .lists-section{
    display: flex;
    overflow-x: auto;
    -ms-flex-wrap: nowrap;
    flex-wrap: nowrap;
  }

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
