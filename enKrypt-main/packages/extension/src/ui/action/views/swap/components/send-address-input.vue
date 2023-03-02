<template>
  <div class="send-address-input" :class="{ focus: isFocus }">
    <div class="send-address-input__avatar">
      <img v-if="isValidAddress" :src="network.identicon(value)" alt="" />
      <Vue3Lottie
        v-else-if="isLoading"
        :animation-data="LottieStatus"
        :loop="true"
        class="send-address-input__avatar__animation"
      />
      <Vue3Lottie
        v-else-if="address !== '' && !isValidAddress && !isLoading"
        :animation-data="LottieError"
        :loop="false"
      />
    </div>
    <div class="send-address-input__address">
      <p>
        To
        {{
          toNetwork
            ? `${toNetwork.charAt(0).toUpperCase()}${toNetwork.slice(
                1
              )} Network`
            : ""
        }}:
      </p>
      <input
        ref="addressInput"
        v-model="address"
        type="text"
        :placeholder="`${network.name_long} address`"
        :style="{ color: !isValidAddress && !isLoading ? 'red' : 'black' }"
        @focus="changeFocus"
        @blur="changeFocus"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { BaseNetwork } from "@/types/base-network";
import { replaceWithEllipsis } from "@/ui/action/utils/filters";
import { computed } from "@vue/reactivity";
import { Vue3Lottie } from "vue3-lottie";
import LottieStatus from "@action/assets/animation/status.json";
import LottieError from "@action/assets/animation/error.json";
import { PropType, ref } from "vue";

const isFocus = ref<boolean>(false);
const addressInput = ref<HTMLInputElement>();

const pasteFromClipboard = () => {
  addressInput.value?.focus();
  document.execCommand("paste");
};
defineExpose({ addressInput, pasteFromClipboard });

const props = defineProps({
  value: {
    type: String,
    default: () => {
      return "";
    },
  },
  isValidAddress: {
    type: Boolean,
    default: () => false,
  },
  isLoading: {
    type: Boolean,
    default: () => false,
  },
  network: {
    type: Object as PropType<BaseNetwork>,
    default: () => ({}),
  },
  toNetwork: {
    type: String,
    default: () => null,
  },
});
const emit = defineEmits<{
  (e: "update:inputAddress", address: string): void;
  (e: "toggle:showContacts", show: boolean): void;
}>();
const address = computed({
  get: () => {
    let displayAddress: string | undefined;

    try {
      displayAddress = props.network.displayAddress(props.value);
    } catch {
      displayAddress = props.value;
    }

    return isFocus.value
      ? displayAddress
      : replaceWithEllipsis(displayAddress, 6, 6);
  },
  set: (value) => emit("update:inputAddress", value),
});

const changeFocus = (val: FocusEvent) => {
  isFocus.value = val.type === "focus";
  if (isFocus.value) emit("toggle:showContacts", isFocus.value);
};
</script>

<style lang="less">
@import "~@action/styles/theme.less";

.send-address-input {
  height: 64px;
  background: #ffffff;
  margin: 12px 32px 8px 32px;
  box-sizing: border-box;
  border: 1px solid @gray02;
  box-sizing: border-box;
  border-radius: 10px;
  width: calc(~"100% - 64px");
  padding: 16px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  flex-direction: row;
  position: relative;

  &.focus {
    border: 2px solid @primary;
    width: calc(~"100% - 62px");
    margin: 12px 31px 8px 31px;
  }

  &__avatar {
    background: @buttonBg;
    box-shadow: inset 0px 0px 1px rgba(0, 0, 0, 0.16);
    width: 32px;
    height: 32px;
    border-radius: 100%;
    overflow: hidden;
    margin-right: 12px;

    * {
      width: 100%;
      height: 100%;
    }
  }

  &__address {
    p {
      font-style: normal;
      font-weight: 400;
      font-size: 12px;
      line-height: 16px;
      letter-spacing: 0.5px;
      color: @secondaryLabel;
      margin: 0;
    }

    input {
      width: 290px;
      height: 24px;
      font-style: normal;
      font-weight: 400;
      font-size: 16px;
      line-height: 24px;
      letter-spacing: 0.25px;
      color: @primaryLabel;
      border: 0 none;
      outline: none;
      padding: 0;
    }
  }

  &__arrow {
    position: absolute;
    font-size: 0;
    cursor: pointer;
    padding: 4px;
    right: 8px;
    top: 16px;
  }
}
</style>