<template>
  <div>
    <div
      :contenteditable="contenteditable"
      @dblclick="onDoubleClick"
      @keypress.enter="onKeyPressEnter"
      @blur="onBlur"
    >
      <Cross @click="removeList" />
      {{ list.name }}
    </div>
    <Card
      v-for="card in list.cards"
      :key="card.id"
      class="card"
      :listId="list.id"
      :card="card"
      :cardText.sync="card.text"
      @remove-card="removeCard"
    />
    <input type="text" @change="addCard" />
  </div>
</template>

<script lang="ts">
export interface AddCardEvent {
  listId: number;
  text: string;
}
import { IList } from "@/type";
import Card, { RemoveCardEvent } from "@/components/Card.vue";
import { Component, Vue, Prop, Emit, PropSync } from "vue-property-decorator";
import Cross from "@/components/Cross.vue";
import { RemoveCardEvent } from "@/components/Card.vue";

@Component({
  components: {
    Card,
    Cross
  }
})
export default class List extends Vue {
  @Prop({ type: Object, required: true })
  list!: IList;

  @PropSync("listName", { type: String, required: true })
  syncedListName!: IList["name"];
  contententable = false;

  onDoubleClick(event: MouseEvent & { currentTarget: HTMLDivElement }): void {
    // 要素のテキストの編集を可能にする
    this.contententable = true;
    // 要素にフォーカスを当てる
    event.currentTarget.focus();
  }

  onKeyPressEnter(
    event: KeyboardEvent & { currentTarget: HTMLDivElement }
  ): void {
    // 要素からフォーカスを外す
    event.currentTarget.blur();
  }

  onBlur(event: FocusEvent & { currentTarget: HTMLDivElement }): void {
    // 要素のテキストの編集を不可にする
    this.syncedListName = event.currentTarget.innerText;
    this.contententable = false;
  }

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
  @Emit()
  removeList(): number {
    return this.list.id;
  }

  @Emit()
  removeCard(event: RemoveCardEvent): RemoveCardEvent {
    return event;
  }
}
</script>
