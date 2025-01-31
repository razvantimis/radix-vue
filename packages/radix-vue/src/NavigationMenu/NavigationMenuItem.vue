<script lang="ts">
export interface NavigationMenuItemProps extends PrimitiveProps {
  value?: string;
}

export type NavigationMenuItemContextValue = {
  value: string;
  contentId: string;
  triggerRef: Ref<HTMLElement | undefined>;
  focusProxyRef: Ref<HTMLElement | undefined>;
  wasEscapeCloseRef: Ref<boolean>;
  onEntryKeyDown(): void;
  onFocusProxyEnter(side: "start" | "end"): void;
  onContentFocusOutside(): void;
  onRootContentClose(): void;
};

export const NAVIGATION_MENU_ITEM_INJECTION_KEY =
  Symbol() as InjectionKey<NavigationMenuItemContextValue>;
</script>

<script setup lang="ts">
import { NAVIGATION_MENU_INJECTION_KEY } from "./NavigationMenuRoot.vue";
import { provide, ref, type InjectionKey, type Ref, inject } from "vue";
import { Primitive, type PrimitiveProps } from "@/Primitive";
import { useArrowNavigation, useCollection, useId } from "@/shared";
import {
  getTabbableCandidates,
  removeFromTabOrder,
  focusFirst,
  makeContentId,
} from "./utils";

const props = withDefaults(defineProps<NavigationMenuItemProps>(), {
  as: "li",
});
const { getItems } = useCollection();
const context = inject(NAVIGATION_MENU_INJECTION_KEY);

const value = props.value || useId();
const triggerRef = ref<HTMLElement>();
const focusProxyRef = ref<HTMLElement>();

const contentId = makeContentId(context!.baseId, value);

let restoreContentTabOrderRef: () => void = () => ({});

const wasEscapeCloseRef = ref(false);
const handleContentEntry = async (side = "start") => {
  // @ts-ignore
  const el = document.getElementById(contentId);
  if (el) {
    restoreContentTabOrderRef();
    const candidates = getTabbableCandidates(el);
    if (candidates.length)
      focusFirst(side === "start" ? candidates : candidates.reverse());
  }
};

const handleContentExit = () => {
  // @ts-ignore
  const el = document.getElementById(contentId);
  if (el) {
    const candidates = getTabbableCandidates(el);
    if (candidates.length)
      restoreContentTabOrderRef = removeFromTabOrder(candidates);
  }
};

provide(NAVIGATION_MENU_ITEM_INJECTION_KEY, {
  value,
  contentId,
  triggerRef,
  focusProxyRef,
  wasEscapeCloseRef,
  onEntryKeyDown: handleContentEntry,
  onFocusProxyEnter: handleContentEntry,
  onContentFocusOutside: handleContentExit,
  onRootContentClose: handleContentExit,
});

const handleClose = () => {
  context?.onItemDismiss();
  triggerRef.value?.focus();
};

const handleKeydown = (ev: KeyboardEvent) => {
  const currentFocus = document.activeElement as HTMLElement;
  if (ev.keyCode === 32 || ev.key === "Enter") {
    if (context?.modelValue.value === value) {
      handleClose();
      ev.preventDefault();
      return;
    } else {
      (ev.target as HTMLElement).click();
      ev.preventDefault();
      return;
    }
  }

  const itemsArray = getItems().filter((i) =>
    i.parentElement?.hasAttribute("aria-menu-item")
  );

  const newSelectedElement = useArrowNavigation(ev, currentFocus, undefined, {
    itemsArray,
    loop: false,
  });

  if (newSelectedElement) {
    newSelectedElement?.focus();
  }
  ev.preventDefault();
  ev.stopPropagation();
};
</script>

<template>
  <Primitive
    :as-child="props.asChild"
    :as="as"
    @keydown.up.down.left.right.home.end.space="handleKeydown"
    aria-menu-item
  >
    <slot></slot>
  </Primitive>
</template>
