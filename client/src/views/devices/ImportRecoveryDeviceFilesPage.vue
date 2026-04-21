<!-- Parsec Cloud (https://parsec.cloud) Copyright (c) BUSL-1.1 2016-present Scille SAS -->

<template>
  <div class="recovery-device-files-page">
    <ms-report-text
      :theme="MsReportTheme.Warning"
      id="warning-text"
    >
      {{ $msTranslate('ImportRecoveryDevicePage.subtitles.recoveryFilesMustExistWarning') }}
    </ms-report-text>
    <div class="recovery-list">
      <div class="recovery-item">
        <ion-text class="recovery-item__title title-h4">
          <span class="recovery-number button-large">1</span>
          {{ $msTranslate('ImportRecoveryDevicePage.modal.recoveryFile.itemRecoveryFile') }}
          <ion-icon
            class="input-validity-icon"
            v-show="recoveryFile"
            :icon="checkmarkCircle"
          />
        </ion-text>
        <div class="recovery-item-key">
          <input
            type="file"
            hidden
            ref="hiddenInput"
            accept=".psrk"
          />
          <div
            class="file-waiting"
            v-if="!recoveryFile"
          >
            <ion-button
              class="file-waiting__button"
              @click="importButtonClick()"
              fill="outline"
            >
              <ion-icon :icon="documentOutline" />
              {{ $msTranslate('ImportRecoveryDevicePage.modal.recoveryFile.actions.addRecoveryFile') }}
            </ion-button>
          </div>

          <div
            v-else
            class="file-added"
          >
            <ion-icon :icon="documentText" />
            <ion-text class="file-added__name button-medium">{{ recoveryFile.name }}</ion-text>
            <ion-button
              class="file-added__update"
              @click="importButtonClick()"
            >
              {{ $msTranslate('ImportRecoveryDevicePage.modal.recoveryFile.actions.updateRecoveryFile') }}
            </ion-button>
          </div>
        </div>
      </div>

      <div class="recovery-item">
        <ion-text class="recovery-item__title title-h4">
          <span class="recovery-number button-large">2</span>
          {{ $msTranslate('ImportRecoveryDevicePage.modal.recoveryFile.itemRecoveryKey') }}
          <ion-icon
            class="input-validity-icon"
            v-show="isSecretKeyValid"
            :icon="checkmarkCircle"
          />
        </ion-text>
        <ms-input
          class="recovery-item__input"
          id="secret-key-input"
          placeholder="ImportRecoveryDevicePage.secretKeyPlaceholder"
          v-model="secretKey"
          @change="checkSecretKeyValidity()"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { secretKeyValidator } from '@/common/validators';
import { IonButton, IonIcon, IonText } from '@ionic/vue';
import { checkmarkCircle, documentOutline, documentText } from 'ionicons/icons';
import { MsInput, MsReportText, MsReportTheme, Validity } from 'megashark-lib';
import { Ref, ref, useTemplateRef } from 'vue';

const hiddenInputRef = useTemplateRef<HTMLInputElement>('hiddenInput');
const secretKey: Ref<string> = ref('');
const recoveryFile: Ref<File | null> = ref(null);
const isSecretKeyValid = ref(false);

defineExpose({
  validateInputs,
});

async function onInputChange(_event: Event): Promise<void> {
  if (hiddenInputRef.value!.files!.length === 1) {
    recoveryFile.value = hiddenInputRef.value!.files![0];
  }
  hiddenInputRef.value!.removeEventListener('change', onInputChange);
}

async function importButtonClick(): Promise<void> {
  hiddenInputRef.value!.addEventListener('change', onInputChange);
  hiddenInputRef.value!.click();
}

async function checkSecretKeyValidity(): Promise<void> {
  isSecretKeyValid.value = (await secretKeyValidator(secretKey.value)).validity === Validity.Valid;
}

async function validateInputs(): Promise<{ recoveryFile: File; secretKey: string } | undefined> {
  if (!recoveryFile.value || !secretKey.value) {
    return;
  }

  await checkSecretKeyValidity();
  if (!isSecretKeyValid.value) {
    return undefined;
  }

  return {
    recoveryFile: recoveryFile.value,
    secretKey: secretKey.value,
  };
}
</script>

<style lang="scss" scoped>
.recovery-device-files-page {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.recovery-list {
  display: flex;
  flex-direction: column;
  gap: 2rem;

  .recovery-item {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    padding: 1rem;
    border-radius: var(--parsec-radius-12);
    background: var(--parsec-color-light-secondary-background);
    border: 1px solid var(--parsec-color-light-secondary-medium);

    &__title {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      color: var(--parsec-color-light-secondary-text);

      .input-validity-icon {
        font-size: 1.125rem;
        color: var(--parsec-color-light-success-700);
      }
    }

    .recovery-number {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 1.5rem;
      height: 1.5rem;
      color: var(--parsec-color-light-secondary-text);
      border-radius: var(--parsec-radius-circle);
      background: var(--parsec-color-light-secondary-white);
      border: 1px solid var(--parsec-color-light-secondary-medium);
      box-shadow: var(--parsec-shadow-input);
    }

    .file-waiting {
      display: flex;
      width: 100%;

      &__button {
        display: flex;
        align-items: center;
        gap: 1rem;
        color: var(--parsec-color-light-secondary-text);
        --background: var(--parsec-color-light-secondary-white);
        --background-hover: var(--parsec-color-light-secondary-premiere);
        width: 100%;

        &::part(native) {
          padding: 0.625rem 1rem;
          border: 1px dashed var(--parsec-color-light-secondary-light);
        }

        &:hover {
          &::part(native) {
            border-color: var(--parsec-color-light-secondary-grey);
          }
        }

        ion-icon {
          font-size: 1.25rem;
          margin-right: 0.5rem;
        }
      }
    }

    .file-added {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      color: var(--parsec-color-light-secondary-text);
      background-color: var(--parsec-color-light-secondary-white);
      border: 1px solid var(--parsec-color-light-secondary-medium);
      border-radius: var(--parsec-radius-12);
      padding: 0.625rem;
      min-height: 2.5rem;
      overflow: hidden;

      &__update {
        --background: var(--parsec-color-light-secondary-medium);
        --background-hover: var(--parsec-color-light-secondary-disabled);
        color: var(--parsec-color-light-secondary-text);
        margin-left: auto;
        box-shadow: var(--parsec-shadow-input);

        &::part(native) {
          padding: 0.75rem 1rem;
        }

        &:hover {
          color: var(--parsec-color-light-secondary-text);
        }
      }

      ion-icon {
        background: var(--parsec-color-light-secondary-premiere);
        color: var(--parsec-color-light-secondary-text);
        display: flex;
        align-items: center;
        padding: 0.5rem;
        border-radius: var(--parsec-radius-8);
        font-size: 1.25rem;
      }
    }

    &__input {
      width: 100%;
      border-radius: var(--parsec-radius-12);
      background: var(--parsec-color-light-secondary-white);
    }
  }
}
</style>
