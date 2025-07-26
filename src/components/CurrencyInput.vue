<template>
  <v-text-field
    :label="label"
    :prefix="prefix"
    :clearable="clearable"
    :variant="variant"
    :color="color"
    :density="density"
    :model-value="displayedValue"
    @input="handleInput"
    @focus="handleFocus"
    @blur="handleBlur"
    inputmode="numeric"
    ref="inputRef"
  ></v-text-field>
</template>

<script setup lang="ts">
import { ref, watch, computed, nextTick } from 'vue';

const props = defineProps<{
  modelValue: number | null; // O valor numérico real que o componente pai vai receber (ex: 10.00)
  label: string;
  prefix?: string;
  clearable?: boolean;
  variant?: string;
  color?: string;
  density?: string;
}>();

const emit = defineEmits(['update:modelValue', 'input']);

const inputRef = ref<HTMLInputElement | null>(null); // Referência para o v-text-field
const internalNumericValue = ref<number | null>(null); // Valor numérico puro (ex: 1000.00)

// Computed property para o valor exibido no input
const displayedValue = computed(() => {
  if (internalNumericValue.value === null || isNaN(internalNumericValue.value)) {
    return '';
  }
  return internalNumericValue.value.toLocaleString('pt-BR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
    useGrouping: true,
  });
});

// Observa o modelValue externo e atualiza o internalNumericValue
watch(() => props.modelValue, (newValue) => {
  if (newValue !== null && newValue !== undefined) {
    internalNumericValue.value = newValue;
  } else {
    internalNumericValue.value = null;
  }
}, { immediate: true });

// Função para formatar um número para o display
const formatNumberForDisplay = (value: number | null): string => {
  if (value === null || isNaN(value)) {
    return '';
  }
  return value.toLocaleString('pt-BR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
    useGrouping: true,
  });
};


let lastCursorPosition = 0; // Armazena a posição do cursor antes da formatação

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const oldRawValue = target.value; // Valor antes da mudança (já formatado)

  // 1. Obter o valor numérico puro da string digitada
  let rawDigits = oldRawValue.replace(/\D/g, ''); // Remove tudo que não for dígito
  if (rawDigits === '') { // Se tudo for apagado
    internalNumericValue.value = null;
    emit('update:modelValue', null);
    return;
  }

  // Se o usuário digitou um 0 e não tem nada antes, mantém 000
  if (rawDigits.length === 1 && rawDigits === '0') {
    rawDigits = '000'; // Permite começar com 0,00
  } else {
    // Remove zeros à esquerda, exceto para o caso de "0,XX"
    rawDigits = rawDigits.replace(/^0+(?!$)/, ''); // Remove zeros à esquerda a menos que seja só "0"
    if (rawDigits.length < 3) {
      rawDigits = rawDigits.padStart(3, '0'); // Garante pelo menos 3 dígitos (para 0,0x ou 0,00)
    }
  }


  const newNumericValue = parseFloat(rawDigits) / 100;

  // 2. Calcular a posição do cursor
  // O cursor se move da direita para a esquerda quando digitamos, mas precisamos ajustá-lo para a formatação
  const oldLength = oldRawValue.length;
  const newFormattedValue = formatNumberForDisplay(newNumericValue);
  const newLength = newFormattedValue.length;

  // Calcula a diferença de caracteres adicionados pela formatação
  const charsAdded = newLength - rawDigits.length;

  // A posição do cursor ideal é onde ele estava antes da formatação, ajustada pelos novos caracteres
  // Vamos usar a diferença entre o valor antigo (formatado) e o valor novo (formatado)
  const previousValueBeforeFormat = (parseFloat(internalNumericValue.value ? internalNumericValue.value.toFixed(2).replace('.', '') : '000') / 100);
  const previousFormattedValueLength = formatNumberForDisplay(previousValueBeforeFormat).length;
  
  const currentCursorPosition = target.selectionStart || 0;
  
  // Tenta manter o cursor na mesma posição relativa aos números digitados
  // Isso é um pouco tricky com a formatação em tempo real
  let newCursorPos = currentCursorPosition + (newLength - previousFormattedValueLength);

  // Garante que o cursor não vá para além do final
  if (newCursorPos > newLength) {
    newCursorPos = newLength;
  }
  // Garante que o cursor não vá para antes do início (raro, mas evita erros)
  if (newCursorPos < 0) {
    newCursorPos = 0;
  }


  internalNumericValue.value = newNumericValue;
  emit('update:modelValue', newNumericValue);
  emit('input', event);

  // nextTick para garantir que o DOM seja atualizado antes de mover o cursor
  nextTick(() => {
    if (inputRef.value) {
      const inputElement = inputRef.value.input as HTMLInputElement;
      if (inputElement) {
        // Tenta ajustar o cursor para a posição correta após a formatação
        // Esta é a parte mais complexa e que pode precisar de ajustes finos
        // A lógica de "charsAdded" ou "calculo de diferença de pontos/virgulas" é a chave
        
        // Uma abordagem mais robusta para o cursor:
        // Contar quantos caracteres de não-dígito (ponto/vírgula) existem antes do cursor original
        const originalNonDigitsBeforeCursor = (oldRawValue.substring(0, currentCursorPosition).match(/[,.]/g) || []).length;
        const newNonDigitsBeforeCursor = (newFormattedValue.substring(0, newCursorPos).match(/[,.]/g) || []).length;

        // Se o número de caracteres de formatação mudou, ajuste o cursor
        if (newNonDigitsBeforeCursor !== originalNonDigitsBeforeCursor) {
            newCursorPos += (newNonDigitsBeforeCursor - originalNonDigitsBeforeCursor);
        }
        
        // Ajuste final se o cursor ficou antes da vírgula e o valor não é zero
        if (newNumericValue !== null && newNumericValue > 0 && newCursorPos < newFormattedValue.length - 2) {
            const commaIndex = newFormattedValue.indexOf(',');
            if (commaIndex > -1 && newCursorPos > commaIndex) { // Se o cursor estiver na parte decimal, não mexe
                // Sem ação
            } else if (commaIndex > -1 && newCursorPos === commaIndex) { // Se o cursor estiver na vírgula, move para direita
                newCursorPos = commaIndex + 1;
            } else if (commaIndex > -1 && newCursorPos < commaIndex) { // Se o cursor estiver antes da vírgula
                // Move para a direita se um ponto foi adicionado antes
                if (newFormattedValue.length > oldRawValue.length && newFormattedValue.charAt(newCursorPos -1) === '.') {
                    newCursorPos++;
                }
            }
        }
        
        // Garante que o cursor não passe do final
        if (newCursorPos > newLength) newCursorPos = newLength;
        // Garante que o cursor não vá para a parte decimal se não for o caso
        if (newNumericValue !== null && newNumericValue < 1 && newCursorPos > 1) { // Para valores como 0,01, o cursor não deve ir antes do 0
            if (newFormattedValue.indexOf(',') > -1 && newCursorPos > newFormattedValue.indexOf(',')) {
                // Fica onde está
            } else if (newFormattedValue.indexOf(',') > -1 && newCursorPos <= newFormattedValue.indexOf(',')) {
                // Move para depois do primeiro '0'
                newCursorPos = 1;
            }
        }


        inputElement.setSelectionRange(newCursorPos, newCursorPos);
      }
    }
  });
};

const handleFocus = (event: FocusEvent) => {
  // Nada de especial no foco, o display já é formatado
  // O que é importante é que o cursor seja manipulado no @input
  const target = event.target as HTMLInputElement;
  lastCursorPosition = target.selectionStart || 0;
};

const handleBlur = () => {
  // Quando perde o foco, o valor já estará formatado, então nada a fazer aqui especificamente
  // O watch já garante que o modelValue atualiza o internalNumericValue se o pai mudar
};
</script>