<template>
  <div class="transaction-footer">
    <!-- Change network -->
    <transition v-bind="TransitionAlertScaleInOutTransition">
      <CommonErrorBlock
        v-if="buttonStep === 'network' && switchingNetworkError"
        class="mb-2"
        @try-again="eraWalletStore.setCorrectNetwork"
      >
        Network change error: {{ switchingNetworkError.message }}
      </CommonErrorBlock>
    </transition>

    <div v-if="buttonStep === 'network'" class="transaction-footer-row">
      <CommonButtonTopInfo>Incorrect network selected in your wallet</CommonButtonTopInfo>
      <CommonButton
        v-if="connectorName !== 'WalletConnect'"
        type="submit"
        :disabled="switchingNetworkInProgress"
        variant="primary-solid"
        @click="eraWalletStore.setCorrectNetwork"
      >
        Change wallet network to {{ eraNetwork.name }}
      </CommonButton>
      <CommonButton v-else disabled variant="primary-solid">
        Change network manually to {{ eraNetwork.name }} in your {{ walletName }} wallet
      </CommonButton>
    </div>
    <div v-else-if="buttonStep === 'continue'" class="transaction-footer-row">
      <slot name="after-checks" />
    </div>

    <TransactionButtonUnderlineContinueInWallet :opened="continueInWalletTipDisplayed" />
  </div>
</template>

<script lang="ts" setup>
import { computed } from "vue";

import { storeToRefs } from "pinia";

import { useOnboardStore } from "@/store/onboard";
import { useEraProviderStore } from "@/store/zksync/era/provider";
import { useEraWalletStore } from "@/store/zksync/era/wallet";
import { TransitionAlertScaleInOutTransition } from "@/utils/transitions";

const onboardStore = useOnboardStore();
const eraWalletStore = useEraWalletStore();

const { connectorName, walletName } = storeToRefs(onboardStore);
const { isCorrectNetworkSet, switchingNetworkInProgress, switchingNetworkError } = storeToRefs(eraWalletStore);
const { eraNetwork } = storeToRefs(useEraProviderStore());

const buttonStep = computed(() => {
  if (!isCorrectNetworkSet.value) {
    return "network";
  } else {
    return "continue";
  }
});

const continueInWalletTipDisplayed = computed(() => {
  if (buttonStep.value === "network" && switchingNetworkInProgress.value) {
    return true;
  }
  return false;
});
</script>

<style lang="scss" scoped>
.transaction-footer {
  @apply sticky bottom-0 z-[2] mt-auto flex flex-col items-center bg-gray bg-opacity-60 pb-2 pt-4 backdrop-blur-sm dark:bg-neutral-950;

  .transaction-footer-row {
    @apply flex w-full flex-col items-center;
  }
}
</style>
