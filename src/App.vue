<template>
  <div id="app">
    <List
      v-for="list in lists"
      :key="list.id"
      class="list"
      :list="list"
      :listName.sync="list.name"
      @add-card="addCard"
      @remove-list="removeList"
      @remove-card="removeCard"
      draggable
      @dragstart.native="onDragStart(list, $event)"
      @dragover.native="moveList(list.id, $event)"
      @dragend.native="onDragEnd"
      :draggedCardData.sync="draggedCardData"
      @move-card="moveCard"
      @dragenter.native="moveCardForEmpty(list.id)"
    />
    <input type="text" class="list-input" @change="addList" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import List, { AddCardEvent, MoveCardEvent } from "@/components/List.vue";
import { ICard, IList } from "@/types";
import { createInitialLists } from "@/initialData";
import { RemoveCardEvent } from "@/components/Card.vue";

enum CursorPosition {
  Left,
  Right,
  Center,
  Top,
  Bottom
}

export interface CardData {
  listId: number;
  card: ICard;
}

@Component({
  components: {
    List
  }
})
export default class App extends Vue {
  lists: IList[] = createInitialLists();
  listCreateCount = createInitialLists().length;
  cardCreateCount = 4;

  // ドラッグ中のリスト
  draggedList: IList | null = null;
  draggedCardData: CardData | null = null;
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
    const list = this.lists.find(list => list.id === listId);
    if (list === undefined) return;
    const cardIndex = list.cards.findIndex(card => card.id === cardId);
    if (cardIndex === -1) return;
    list.cards.splice(cardIndex, 1);
  }

  onDragStart(list: IList, event: DragEvent): void {
    // Firefoxでドラッグする際に必要
    if (event.dataTransfer == null) {
      return;
    }
    event.dataTransfer.setData("text/plain", "");

    this.draggedList = list;
  }

  onDragEnd(): void {
    this.draggedList = null;
  }

  moveList(
    listId: number,
    event: DragEvent & { currentTarget: HTMLDivElement }
  ): void {
    if (this.draggedList === null || this.draggedList.id === listId) {
      return;
    }

    // 重ねられてる要素の左端の座標(px)
    const left: number = event.currentTarget.getBoundingClientRect().left - 1;
    // 重ねられている要素の右側の座標(px)
    const right: number = event.currentTarget.getBoundingClientRect().right - 1;
    // 重ねられている要素(の水平方向)の中央の座標(px)
    const centerX: number = left + (right - left) / 2;

    // カーソルが重ねられている場所はどこか
    const cursorPosition: CursorPosition = (() => {
      // カーソルが左端から中央の場合は左側
      if (left <= event.clientX && event.clientX < centerX) {
        return CursorPosition.Left;
        // カーソルが中央から右端の場合は右側
      } else if (centerX < event.clientX && event.clientX <= right) {
        return CursorPosition.Right;
        // どちらでもない場合は中央
      } else {
        return CursorPosition.Center;
      }
    })();

    // 中央の時は早期リターン
    if (cursorPosition === CursorPosition.Center) {
      return;
    }

    // まずドラッグしているリストを削除
    const draggedListIndex = this.lists.findIndex(
      list => list.id === this.draggedList!.id
    );
    if (draggedListIndex === -1) {
      return;
    }
    this.lists.splice(draggedListIndex, 1);

    const listIndex = this.lists.findIndex(list => list.id === list.id);
    if (listIndex === -1) return;
    const toListIndex =
      cursorPosition === CursorPosition.Left ? listIndex : listIndex + 1;
    this.lists.splice(toListIndex, 0, this.draggedList);
  }

  moveCard({ listId, cardId, event }: MoveCardEvent): void {
    if (
      this.draggedCardData === null ||
      this.draggedCardData.card.id === cardId
    ) {
      return;
    }
    // 重ねられている要素の上端の座標(px)
    const top: number = event.currentTarget.getBoundingClientRect().top - 1;
    // 重ねられている要素の下端の座標(px)
    const bottom: number =
      event.currentTarget.getBoundingClientRect().bottom - 1;
    // 重ねられている要素の（垂直）の中央の座標(px)
    const centerY: number = top + (bottom - top) / 2;

    // カーソルが重ねられている要素はどこにあるか
    const cursorPosition: CursorPosition = (() => {
      // カーソルが上端の場合は上側
      if (top <= event.clientY && event.clientY < centerY) {
        return CursorPosition.Top;
      }
      // カーソルが下端の場合は下側
      if (centerY < event.clientY && event.clientY < bottom) {
        return CursorPosition.Bottom;
      }
      // どちらでもない場合は中央
      return CursorPosition.Center;
    })();
    // 中央のの時は早期リターン
    if (cursorPosition === CursorPosition.Center) {
      return;
    }
    // まずドラッグしているカードを削除
    const draggedCardList = this.lists.find(
      list => list.id === this.draggedCardData!.listId
    );
    if (draggedCardList === undefined) {
      return;
    }
    const draggedCardIndex = draggedCardList.cards.findIndex(
      card => card.id === this.draggedCardData!.card.id
    );
    if (draggedCardIndex === -1) {
      return;
    }
    draggedCardList.cards.splice(draggedCardIndex, 1);

    const list = this.lists.find(list => list.id === listId);
    if (list === undefined) {
      return;
    }
    const cardIndex = list.cards.findIndex(card => card.id === cardId);
    if (cardIndex === -1) {
      return;
    }
    // カーソルが重ねられている要素の上にある場合は上、下にある場合は下に要素を追加する
    const toCardIndex =
      cursorPosition === CursorPosition.Top ? cardIndex : cardIndex + 1;
    list.cards.splice(toCardIndex, 0, this.draggedCardData.card);
    // 帰属するリストが変わる可能性があるので、移動後のリストのidを代入
    this.draggedCardData.listId = listId;
  }

  moveCardEmpty(listId: number) {
    if (
      this.draggedCardData === null ||
      this.draggedCardData.listId === listId
    ) {
      return;
    }
    const list = this.lists.find(list => list.id === listId);
    if (list === undefined || list.cards.length !== 0) {
      //リスト内にカードが１つでもある場合は早期リターン
      return;
    }
    const draggedCardList = this.lists.find(
      list => list.id === this.draggedCardData!.listId
    );
    if (draggedCardList === undefined) {
      return;
    }
    const draggedCardIndex = draggedCardList.cards.findIndex(
      card => card.id === this.draggedCardData!.card.id
    );
    if (draggedCardIndex === -1) {
      return;
    }
    draggedCardList.cards.splice(draggedCardIndex, 1);
    list.cards.push(this.draggedCardData.card);

    this.draggedCardData.listId = listId;
  }
}
</script>

<style lang="scss">
#app {
  display: flex;
  align-items: flex-start;

  > .list {
    margin: 1px;
  }
  > .list-input {
    width: 160px;
    margin: 1px;
  }
}
</style>
