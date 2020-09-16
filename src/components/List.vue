<template>
  <div>{{ list.name }}</div>
</template>

<script lang="ts">
import { IList } from "@/type";
import Card from "@/components/Card.vue";
import { Component, Vue, Prop } from "vue-property-decorator";

@Component({
  components: {
    Card
  }
})
export default class List extends Vue {
  @Prop({ type: Object, required: true })
  list!: IList;
}
</script>
