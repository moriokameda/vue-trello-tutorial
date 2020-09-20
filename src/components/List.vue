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
      draggable
      @dragstart.native.stop="onDragStart(card, $event)"
      @dragover.native="moveCard(card.id, $event)"
      @dragend.native="onDragEnd"
    />
    <input type="text" @change="addCard" />
  </div>
</template>

<script lang="ts">
export interface AddCardEvent {
  listId: number;
  text: string;
}
export interface MoveCardEvent {
  listId: number;
  cardId: number;
  event: DragEvent & { currentTarget: HTMLDivElement };
}

import { ICard, IList } from "@/types.ts";
import Card, { RemoveCardEvent } from "@/components/Card.vue";
import { Component, Vue, Prop, Emit, PropSync } from "vue-property-decorator";
import Cross from "@/components/Cross.vue";
import { CardData } from "@/App.vue";

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

  @PropSync("draggedCardData", { required: true })
  syncDraggedCardData!: CardData | null;
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
    console.log(this.list.id);
    return this.list.id;
  }

  @Emit()
  removeCard(event: RemoveCardEvent): RemoveCardEvent {
    return event;
  }

  onDragStart(card: ICard, event: DragEvent): void {
    if (event.dataTransfer == null) {
      return;
    }
    event.dataTransfer.setData("text/plain", "");
    this.syncDraggedCardData = {
      listId: this.list.id,
      card
    };
  }

  @Emit()
  moveCard(
    cardId: number,
    event: DragEvent & { currentTarget: HTMLDivElement }
  ): MoveCardEvent {
    return {
      listId: this.list.id,
      cardId,
      event
    };
  }
  onDragEnd(): void {
    this.syncDraggedCardData = null;
  }
}
</script>
<style lang="scss">
.list {
  width: 160px;
  border: 1px solid #000;
  &-name {
    position: relative;

    > .cross {
      right: 0;
    }
  }
  > .card {
    margin: 1px;
  }
  > .card-input {
    width: 100%;
  }
}
</style>
