<script setup lang="ts">
import { MenuPortal, type MenuPortalProps } from "@/Menu";
interface ContextMenuPortalProps extends MenuPortalProps {}

const props = defineProps<ContextMenuPortalProps>();
</script>

<template>
  <MenuPortal v-bind="props">
    <slot></slot>
  </MenuPortal>
</template>
