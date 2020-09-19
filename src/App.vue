<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <List
      v-for="list in lists"
      :key="list.id"
      class="list"
      :list="list"
      :listName.sync="list.name"
      @add-cart="addCard"
      @remove-list="removeList"
      @remove-card="removeCard"
    />
    <input type="text" @change="addList" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import List, { AddCardEvent } from "@/components/List.vue";
import { IList } from "@/type";
import { createInitialLists } from "@/initialData";
import { RemoveCardEvent } from "@/components/Card.vue";

@Component({
  components: {
    List
  }
})
export default class App extends Vue {
  lists: IList[] = createInitialLists();
  listCreateCount = createInitialLists().length;
  cardCreateCount = 4;

  addCard({ listId, text }: AddCardEvent): void {
    const list = this.lists.find(list => list.id === listId);
    /*
    findは見つからなかった場合undefinedを返す可能性があるのでその場合は早期リターンをする
    ここではlist: IList | undefined
    */
    if (list === undefined) {
      return;
    }
    const newCard = {
      id: this.cardCreateCount + 1,
      text
    };
    list.cards.push(newCard);
    ++this.cardCreateCount;
  }
  // 値を繰り返さない関数の返り値返り値の値としてvoid型を付与
  addList(event: Event & { currentTarget: HTMLInputElement }): void {
    const newList = {
      id: this.listCreateCount + 1,
      name: event.currentTarget.value,
      cards: []
    };
    this.lists.push(newList);
    ++this.listCreateCount;
    event.currentTarget.value = "";
  }
  removeList(listId: number): void {
    const listIndex = this.lists.findIndex(list => list.id == listId);

    // findIndexで見つからない場合は-1を返すのでその場合は早期リターン
    if (listIndex === -1) {
      return;
    }

    this.lists.splice(listIndex, 1);
  }

  removeCard({ listId, cardId }: RemoveCardEvent): void {
    const list = this.lists.find(list => list.id == listId);
    if (list === undefined) {
      return;
    }
    const cardIndex = list.cards.findIndex(card => card.id === cardId);
    if (cardIndex === -1) {
      return;
    }
    list.cards.splice(cardIndex, 1);
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
