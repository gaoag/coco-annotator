<template>
  <div v-show="line.isActive">
    <PanelButton name="Complete Line" @click="line.complete" />
    <PanelButton name="Delete Line" @click="line.deleteLine" />
    <PanelToggle name="Guidance" v-model="line.line.guidance" />
    <PanelToggle name="Auto Select Color" v-model="line.color.auto" />
    <PanelToggle
      v-show="line.color.auto"
      name="Only Black or White"
      v-model="line.color.blackOrWhite"
    />
    <PanelInputString
      name="Stroke Color"
      v-model="line.line.pathOptions.strokeColor"
    />
  </div>
</template>

<script>
import PanelButton from "@/components/PanelButton";
import PanelToggle from "@/components/PanelToggle";
import PanelInputString from "@/components/PanelInputString";
import PanelInputNumber from "@/components/PanelInputNumber";

export default {
  name: "LinePanel",
  components: { PanelButton, PanelToggle, PanelInputString, PanelInputNumber },
  props: {
    line: {
      type: Object,
      required: true
    }
  }
};
</script>
