<!-- Parsec Cloud (https://parsec.cloud) Copyright (c) BUSL-1.1 2016-present Scille SAS -->

<template>
  <ms-modal
    :title="stepTitles.titles"
    :subtitle="stepTitles.subtitles"
    :close-button="currentStep === RecoveryMethodStep.Done ? { visible: false } : { visible: true }"
    :cancel-button="cancelButton"
    :confirm-button="confirmButton"
  >
    <div class="import-recovery-device-modal__content">
      <ion-radio-group
        v-if="currentStep === RecoveryMethodStep.MethodChoice"
        class="recovery-method-list"
        v-model="selectedMethod"
      >
        <ion-radio
          :value="RecoveryMethodStep.ConnectedDevice"
          class="recovery-method-item recovery-method--recommended"
          :class="{ 'recovery-method-item--selected': selectedMethod === RecoveryMethodStep.ConnectedDevice }"
        >
          <div class="recovery-method-item__container">
            <ms-image
              :image="RecoveryDeviceIcon"
              class="recovery-method-item__image"
            />
            <div class="recovery-method-item__content">
              <ion-text class="recovery-method-item__title title-h4">
                {{ $msTranslate('ImportRecoveryDevicePage.modal.options.otherConnectedDevice.title') }}
              </ion-text>
              <ion-text class="recovery-method-item__subtitle body-lg">
                {{ $msTranslate('ImportRecoveryDevicePage.modal.options.otherConnectedDevice.subtitle') }}
              </ion-text>
            </div>
          </div>
          <span class="recovery-method-item__recommended-badge subtitles-sm">
            {{ $msTranslate('ImportRecoveryDevicePage.modal.options.otherConnectedDevice.badge') }}
          </span>
        </ion-radio>

        <ion-radio
          :value="RecoveryMethodStep.RecoveryFile"
          class="recovery-method-item"
          :class="{ 'recovery-method-item--selected': selectedMethod === RecoveryMethodStep.RecoveryFile }"
        >
          <ms-image
            :image="RecoveryFileIcon"
            class="recovery-method-item__image"
          />
          <div class="recovery-method-item__content">
            <ion-text class="recovery-method-item__title title-h4">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.options.recoveryFile.title') }}
            </ion-text>
            <ion-text class="recovery-method-item__subtitle body-lg">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.options.recoveryFile.subtitle') }}
            </ion-text>
          </div>
        </ion-radio>

        <ion-radio
          v-if="false"
          :value="RecoveryMethodStep.Shamir"
          class="recovery-method-item"
          :class="{ 'recovery-method-item--selected': selectedMethod === RecoveryMethodStep.Shamir }"
        >
          <ms-image
            :image="RecoveryTrustIcon"
            class="recovery-method-item__image"
          />
          <div class="recovery-method-item__content">
            <ion-text class="recovery-method-item__title title-h4">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.options.shamir.title') }}
            </ion-text>
            <ion-text class="recovery-method-item__subtitle body-lg">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.options.shamir.subtitle') }}
            </ion-text>
          </div>
        </ion-radio>
      </ion-radio-group>

      <!-- step 2.1 - Method Recovery: Connected Device -->
      <template v-else-if="currentStep === RecoveryMethodStep.ConnectedDevice">
        <div class="method-step-list">
          <div class="method-step-list__item">
            <ion-icon
              class="method-step-list__item-icon"
              slot="start"
              :icon="logIn"
            />
            <ion-text class="method-step-list__item-description subtitles-normal">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.connectedDevice.steps.step1') }}
            </ion-text>
          </div>
          <div class="method-step-list__item">
            <ion-icon
              class="method-step-list__item-icon"
              slot="start"
              :icon="phonePortraitOutline"
            />
            <ion-text class="method-step-list__item-description subtitles-normal">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.connectedDevice.steps.step2') }}
            </ion-text>
          </div>
          <div class="method-step-list__item">
            <ion-icon
              class="method-step-list__item-icon"
              slot="start"
              :icon="add"
            />
            <ion-text class="method-step-list__item-description subtitles-normal">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.connectedDevice.steps.step3') }}
            </ion-text>
          </div>
          <div class="method-step-list__item">
            <ion-icon
              class="method-step-list__item-icon"
              slot="start"
              :icon="link"
            />
            <ion-text class="method-step-list__item-description subtitles-normal">
              {{ $msTranslate('ImportRecoveryDevicePage.modal.connectedDevice.steps.step4') }}
            </ion-text>
          </div>
        </div>
      </template>

      <!-- step 2.1 - Method Recovery: Recovery File -->
      <template v-else-if="currentStep === RecoveryMethodStep.RecoveryFile">
        <import-recovery-device-files-page ref="recoveryDeviceFilesPage" />
      </template>

      <!-- step 2.3 - Method Recovery: Shamir -->
      <template v-else-if="currentStep === RecoveryMethodStep.Shamir">
        <import-recovery-device-shamir-page />
      </template>

      <template v-else-if="currentStep === RecoveryMethodStep.Authentication && newDevice">
        <choose-authentication
          ref="chooseAuth"
          :server-config="serverConfig"
          :server-addr="newDevice.serverAddr"
        />
      </template>

      <template v-else-if="currentStep === RecoveryMethodStep.Done">
        <div class="final-step">
          <ms-image
            :image="ResourcesManager.instance().get(Resources.LogoIcon, LogoIconWhite) as string"
            class="final-step__logo"
          />
          <ion-text class="final-step__title title-h3">
            {{ $msTranslate('ImportRecoveryDevicePage.modal.done.title') }}
          </ion-text>
          <ion-button
            fill="solid"
            size="default"
            @click="nextStep()"
            class="final-step__button"
          >
            {{ $msTranslate('ImportRecoveryDevicePage.modal.done.login') }}
          </ion-button>
        </div>
      </template>
    </div>
  </ms-modal>
</template>

<script setup lang="ts">
import RecoveryDeviceIcon from '@/assets/images/recovery-device-icon.svg?raw';
import RecoveryFileIcon from '@/assets/images/recovery-file-icon.svg?raw';
import RecoveryTrustIcon from '@/assets/images/recovery-trusted-user-icon.svg?raw';
import { getDefaultDeviceName } from '@/common/device';
import ChooseAuthentication from '@/components/devices/ChooseAuthentication.vue';
import {
  AvailableDevice,
  DeviceAccessStrategy,
  DevicePrimaryProtectionStrategy,
  ImportRecoveryDeviceErrorTag,
  PrimaryProtectionStrategy,
  ServerConfig,
  constructAccessStrategy,
  constructSaveStrategy,
  getServerConfig,
  importRecoveryDevice,
  updateDeviceChangeAuthentication,
} from '@/parsec';
import { Information, InformationLevel, InformationManager, PresentationMode } from '@/services/informationManager';
import { Resources, ResourcesManager } from '@/services/resourcesManager';
import ImportRecoveryDeviceFilesPage from '@/views/devices/ImportRecoveryDeviceFilesPage.vue';
import ImportRecoveryDeviceShamirPage from '@/views/devices/ImportRecoveryDeviceShamirPage.vue';
import { IonButton, IonIcon, IonRadio, IonRadioGroup, IonText, modalController } from '@ionic/vue';
import { add, link, logIn, phonePortraitOutline } from 'ionicons/icons';
import { LogoIconWhite, MsImage, MsModal, MsModalResult, Translatable, asyncComputed } from 'megashark-lib';
import { computed, ref, useTemplateRef } from 'vue';

enum RecoveryMethodStep {
  MethodChoice = 'method-choice',
  ConnectedDevice = 'connected-device',
  RecoveryFile = 'recovery-file',
  Shamir = 'shamir',
  Authentication = 'authentication',
  Done = 'done',
}

const currentStep = ref(RecoveryMethodStep.MethodChoice);
const selectedMethod = ref<RecoveryMethodStep | undefined>(undefined);
const recoveryDeviceFilesPageRef = useTemplateRef<typeof ImportRecoveryDeviceFilesPage>('recoveryDeviceFilesPage');
const chooseAuthRef = useTemplateRef<InstanceType<typeof ChooseAuthentication>>('chooseAuth');
const serverConfig = ref<ServerConfig | undefined>(undefined);
let tmpDeviceProtection: DevicePrimaryProtectionStrategy | undefined;
let accessStrategy: DeviceAccessStrategy | undefined;
const newDevice = ref<AvailableDevice | undefined>(undefined);
const props = defineProps<{
  device?: AvailableDevice;
  informationManager: InformationManager;
}>();

interface StepTitles {
  titles: Translatable;
  subtitles: Translatable;
}

const stepTitles = computed<StepTitles>(() => {
  switch (currentStep.value) {
    case RecoveryMethodStep.MethodChoice:
      return {
        titles: 'ImportRecoveryDevicePage.modal.title',
        subtitles: 'ImportRecoveryDevicePage.modal.subtitle',
      };
    case RecoveryMethodStep.ConnectedDevice:
      return {
        titles: 'ImportRecoveryDevicePage.modal.connectedDevice.title',
        subtitles: 'ImportRecoveryDevicePage.modal.connectedDevice.subtitle',
      };
    case RecoveryMethodStep.RecoveryFile:
      return {
        titles: 'ImportRecoveryDevicePage.modal.recoveryFile.title',
        subtitles: 'ImportRecoveryDevicePage.modal.recoveryFile.subtitle',
      };
    case RecoveryMethodStep.Shamir:
      return {
        titles: 'ImportRecoveryDevicePage.modal.shamir.title',
        subtitles: 'ImportRecoveryDevicePage.modal.shamir.subtitle',
      };
    case RecoveryMethodStep.Authentication:
      return {
        titles: 'ImportRecoveryDevicePage.modal.authentication.title',
        subtitles: 'ImportRecoveryDevicePage.modal.authentication.subtitle',
      };
    default:
      return {
        titles: '',
        subtitles: '',
      };
  }
});

const changeButtonIsEnabled = asyncComputed(async (): Promise<boolean> => {
  if (!chooseAuthRef.value) {
    return false;
  }
  return await chooseAuthRef.value.areFieldsCorrect();
});

const cancelButton = computed(() => {
  switch (currentStep.value) {
    case RecoveryMethodStep.MethodChoice:
      return {
        disabled: false,
        label: 'TextInputModal.cancel',
      };
    case RecoveryMethodStep.ConnectedDevice:
    case RecoveryMethodStep.RecoveryFile:
    case RecoveryMethodStep.Shamir:
    case RecoveryMethodStep.Authentication:
      return {
        disabled: false,
        label: 'HomePage.topbar.back',
        onClick: goBack,
      };
    case RecoveryMethodStep.Done:
      return undefined;
    default:
      return undefined;
  }
});

const confirmButton = computed(() => {
  switch (currentStep.value) {
    case RecoveryMethodStep.MethodChoice:
      return {
        disabled: selectedMethod.value === undefined,
        label: 'ImportRecoveryDevicePage.modal.actions.chooseMethod',
        onClick: nextStep,
      };
    case RecoveryMethodStep.RecoveryFile:
      return {
        disabled: false,
        label: 'ImportRecoveryDevicePage.modal.actions.next',
        onClick: nextStep,
      };
    case RecoveryMethodStep.Authentication:
      return {
        disabled: !changeButtonIsEnabled.value,
        label: 'ImportRecoveryDevicePage.actions.validateAuth',
        onClick: nextStep,
      };
    case RecoveryMethodStep.Done:
      return undefined;
    case RecoveryMethodStep.ConnectedDevice:
      return {
        disabled: false,
        label: 'ImportRecoveryDevicePage.modal.actions.next',
        onClick: nextStep,
      };
    default:
      return {
        disabled: true,
      };
  }
});

async function nextStep(): Promise<boolean> {
  if (currentStep.value === RecoveryMethodStep.MethodChoice && selectedMethod.value) {
    currentStep.value = selectedMethod.value;
    return false;
  }

  if (currentStep.value === RecoveryMethodStep.ConnectedDevice) {
    await modalController.dismiss({ recoveryMethod: RecoveryMethodStep.ConnectedDevice }, MsModalResult.Confirm);
    return false;
  }

  if (currentStep.value === RecoveryMethodStep.RecoveryFile) {
    const filesResult = await recoveryDeviceFilesPageRef.value?.validateInputs();
    if (!filesResult) {
      return false;
    }

    const reader = filesResult.recoveryFile.stream().getReader();
    const content = new Uint8Array(filesResult.recoveryFile.size);
    let offset = 0;
    let buffer = await reader.read();
    while (!buffer.done) {
      content.set(buffer.value, offset);
      offset += buffer.value.length;
      buffer = await reader.read();
    }
    if (buffer.value) {
      content.set(buffer.value, offset);
    }

    tmpDeviceProtection = constructTemporaryProtection();

    const importResult = await importRecoveryDevice(
      props.device ? props.device.deviceLabel : getDefaultDeviceName(),
      content,
      filesResult.secretKey.trim(),
      constructSaveStrategy(tmpDeviceProtection),
    );

    if (importResult.ok) {
      newDevice.value = importResult.value;
      const serverConfigResult = await getServerConfig(newDevice.value.serverAddr);
      if (serverConfigResult.ok) {
        serverConfig.value = serverConfigResult.value;
      }
      currentStep.value = RecoveryMethodStep.Authentication;
    } else {
      const notificationInfo = { message: '', level: InformationLevel.Error };

      switch (importResult.error.tag) {
        case ImportRecoveryDeviceErrorTag.InvalidPassphrase:
          notificationInfo.message = 'ImportRecoveryDevicePage.errors.keyErrorMessage';
          break;
        case ImportRecoveryDeviceErrorTag.InvalidData:
          notificationInfo.message = 'ImportRecoveryDevicePage.errors.fileErrorMessage';
          break;
        default:
          notificationInfo.message = 'ImportRecoveryDevicePage.errors.internalErrorMessage';
          break;
      }
      props.informationManager.present(new Information(notificationInfo), PresentationMode.Toast);
    }
  }

  if (currentStep.value === RecoveryMethodStep.Authentication) {
    if (!(await chooseAuthRef.value?.areFieldsCorrect())) {
      return false;
    }

    if (!newDevice.value || !tmpDeviceProtection) {
      return false;
    }

    const saveStrategy = await chooseAuthRef.value?.getSaveStrategy();
    if (!saveStrategy) {
      return false;
    }

    const access = constructAccessStrategy(newDevice.value, tmpDeviceProtection);
    const result = await updateDeviceChangeAuthentication(access, saveStrategy);
    if (result.ok) {
      newDevice.value = result.value;
      accessStrategy = constructAccessStrategy(newDevice.value, saveStrategy.primaryProtection, saveStrategy.totpProtection);
      currentStep.value = RecoveryMethodStep.Done;
    } else {
      props.informationManager.present(
        new Information({
          message: 'ImportRecoveryDevicePage.errors.internalErrorMessage',
          level: InformationLevel.Error,
        }),
        PresentationMode.Toast,
      );
      currentStep.value = RecoveryMethodStep.RecoveryFile;
    }
    return false;
  }

  if (currentStep.value === RecoveryMethodStep.Done) {
    if (!newDevice.value || !accessStrategy) {
      return false;
    }

    await modalController.dismiss(
      {
        recoveryMethod: RecoveryMethodStep.RecoveryFile,
        device: newDevice.value,
        access: accessStrategy,
      },
      MsModalResult.Confirm,
    );
    return false;
  }

  return false;
}

async function goBack(): Promise<boolean> {
  if (currentStep.value === RecoveryMethodStep.Authentication) {
    currentStep.value = RecoveryMethodStep.RecoveryFile;
    return false;
  }

  if (currentStep.value === RecoveryMethodStep.Done) {
    currentStep.value = RecoveryMethodStep.Authentication;
    return false;
  }

  currentStep.value = RecoveryMethodStep.MethodChoice;
  return false;
}

function constructTemporaryProtection(): DevicePrimaryProtectionStrategy {
  return PrimaryProtectionStrategy.usePassword(window.crypto.randomUUID());
}
</script>

<style scoped lang="scss">
.import-recovery-device-modal__content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.recovery-method-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.recovery-method-item {
  border-radius: var(--parsec-radius-12);
  background: var(--parsec-color-light-secondary-white);
  padding: 1.5rem;
  position: relative;
  box-shadow:
    0 1px 1px 0 rgba(0, 0, 0, 0.05),
    0 1px 4px 0 rgba(0, 0, 0, 0.03),
    0 0 1px 0 rgba(0, 0, 0, 0.2);
  transition:
    transform 120ms ease,
    border-color 120ms ease,
    box-shadow 120ms ease;

  &::part(label) {
    display: flex;
    gap: 1rem;
    margin: 0;
    width: 100%;
  }

  // Hide default radio circle
  &::part(container) {
    display: none;
  }

  &:not(.recovery-method-item--selected):hover {
    outline: 1.5px solid var(--parsec-color-light-secondary-medium);
  }

  &--selected {
    outline: 1.5px solid var(--parsec-color-light-primary-400);
  }

  &__container {
    display: flex;
    gap: 1rem;
  }

  &--recommended {
    border-color: var(--parsec-color-light-primary-300);
  }

  &__image {
    width: fit-content;
    height: fit-content;
    flex-shrink: 0;
    box-shadow:
      0 1px 1px 0 rgba(0, 0, 0, 0.05),
      0 1px 4px 0 rgba(0, 0, 0, 0.03),
      0 0 1px 0 rgba(0, 0, 0, 0.2);
    border-radius: var(--parsec-radius-8);
  }

  &__content {
    display: flex;
    width: 100%;
    text-wrap: wrap;
    flex-direction: column;
    gap: 0.35rem;
    flex: 1;
  }

  &__title {
    color: var(--parsec-color-light-secondary-text);
  }

  &__subtitle {
    color: var(--parsec-color-light-secondary-hard-grey);
  }

  &__recommended-badge {
    padding: 0.2rem 0.6rem;
    border-radius: var(--parsec-radius-8);
    background: var(--parsec-color-light-primary-50);
    color: var(--parsec-color-light-primary-500);
    line-height: 1;
    white-space: nowrap;
    position: absolute;
    top: -0.5rem;
    right: -0.5rem;
    z-index: 100;
  }
}

.method-step-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;

  &__item {
    background: var(--parsec-color-light-secondary-premiere);
    display: flex;
    align-items: center;
    padding: 0.5rem 0.75rem;
    border-radius: var(--parsec-radius-12);
    gap: 1rem;

    &-icon {
      width: 1.125rem;
      height: 1.125rem;
      flex-shrink: 0;
      padding: 0.375rem;
      color: var(--parsec-color-light-secondary-white);
      background: var(--parsec-color-light-secondary-text);
      border-radius: var(--parsec-radius-circle);
    }

    &-description {
      color: var(--parsec-color-light-secondary-soft-text);
    }
  }
}
</style>
