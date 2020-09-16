<template>
  <div>
    {{ card.text }}
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from "vue-property-decorator";
import { ICard } from "@/type";

@Component
export default class Card extends Vue {
  @Prop({ type: Object, required: true })
  card!: ICard;
}
</script>
<style lang="scss" scoped>

</style>
