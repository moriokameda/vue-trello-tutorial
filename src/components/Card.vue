<template>{{ card.text }}</template>

<script lang="ts">
import { ICard } from "@/type";
import { Component, Vue, Prop } from "vue-property-decorator";

@Component
export default class Card extends Vue {
  @Prop({ type: Object, required: true })
  card!: ICard;
}
</script>
