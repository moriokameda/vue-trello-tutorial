<template>
  <div>
    {{ list.name }}
    <Card v-for="card in list.cards" :key="card.id" class="card" :card="card" />
    <input type="text" @change="addCard" />
  </div>
</template>

<script lang="ts">
export interface AddCardEvent {
  listId: number;
  text: string;
}
import { IList } from "@/type";
import Card from "@/components/Card.vue";
import { Component, Vue, Prop, Emit } from "vue-property-decorator";

@Component({
  components: {
    Card
  }
})
export default class List extends Vue {
  @Prop({ type: Object, required: true })
  list!: IList;

  @Emit()
  addCard(event: Event & { currentTarget: HTMLInputElement }): AddCardEvent {
    // 次の処理でリセットしてしまうので変数に格納
    const text = event.currentTarget.value;
    // フォームの値をリセット
    event.currentTarget.value = "";
    // 返す内容が複数あるのでオブジェクトで返す
    return {
      listId: this.list.id,
      text
    };
  }
}
</script>
