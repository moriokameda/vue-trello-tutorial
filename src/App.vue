<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <List v-for="list in lists"
    :key="list.id"
    class="list"
    :list="list"
    :listName.sync="list.name"
    @add-cart="addCard"
    />
    <input type="text" @change="addList" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import List, { AddCardEvent } from "@/components/List.vue";
import { IList } from "@/type";
import { createInitialLists } from "@/initialData";

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
