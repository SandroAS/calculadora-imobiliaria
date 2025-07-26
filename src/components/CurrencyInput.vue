<template>
  <v-text-field
    :label="label"
    :prefix="prefix"
    :clearable="clearable"
    :variant="variant"
    :color="color"
    :density="density"
    :model-value="displayFormattedValue"
    @keydown.prevent="handlePriceKeydown"
    @focus="handleFocus"
    @blur="handleBlur"
    inputmode="numeric"
  ></v-text-field>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

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

// Valor interno que armazena os centavos como string para manipulação
const internalPriceInCentsString = ref<string>('000'); // Inicia com '000' para '0.00'
const isFocused = ref<boolean>(false);

// Observa o modelValue do componente pai e atualiza o valor interno
watch(() => props.modelValue, (newValue) => {
  if (newValue !== null && newValue !== undefined) {
    const cents = Math.round(newValue * 100);
    // Garante que tenha pelo menos 3 dígitos (para 0.00)
    internalPriceInCentsString.value = String(cents).padStart(3, '0');
  } else if (!isFocused.value) { // Só zera se não estiver focado
    internalPriceInCentsString.value = '000';
  }
}, { immediate: true });


// Computed property para exibir o valor formatado (R$ 10,00)
const displayFormattedValue = computed(() => {
  if (isFocused.value) {
    // Quando focado, mostra o valor bruto sem formatação de agrupamento, apenas com vírgula para centavos
    const rawValue = (parseFloat(internalPriceInCentsString.value) / 100).toFixed(2);
    return rawValue.replace('.', ',');
  } else {
    // Quando não focado, formata como moeda brasileira completa
    const valueAsNumber = parseFloat(internalPriceInCentsString.value) / 100;
    return valueAsNumber.toLocaleString('pt-BR', {
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
      useGrouping: true,
    });
  }
});

// Lógica para lidar com a digitação da direita para a esquerda
function handlePriceKeydown(event: KeyboardEvent) {
  const key = event.key;
  
  let currentPrice = internalPriceInCentsString.value;

  if (!/^\d$/.test(key) && key !== 'Backspace') {
    // Permite algumas teclas de navegação, mas previne outras que não são números
    if (['ArrowLeft', 'ArrowRight', 'Tab', 'Home', 'End'].includes(key)) {
      return;
    }
    event.preventDefault(); // Impede entrada de caracteres não numéricos
    return;
  }

  let newPriceString = currentPrice;

  if (key === 'Backspace') {
    newPriceString = newPriceString.slice(0, -1);
    if (newPriceString.length === 0) {
      newPriceString = '0'; // Garante que nunca fique vazio, mantém '0'
    }
  } else {
    newPriceString += key;
  }

  // Remove zeros à esquerda, mas mantém um zero se o número for menor que 100 (0.XX)
  if (newPriceString.length > 3) { // Se tiver mais de 3 dígitos (ex: 100 para 1,00)
    newPriceString = newPriceString.replace(/^0+(?=\d)/, ''); // Remove zeros à esquerda
  } else if (newPriceString.length > 1 && newPriceString.startsWith('0')) {
     newPriceString = newPriceString.replace(/^0+/, '0'); // Garante que não tenha múltiplos zeros iniciais
  }


  internalPriceInCentsString.value = newPriceString;

  // Converte para o valor numérico em reais e emite para o v-model
  const valueForParent = parseFloat(internalPriceInCentsString.value) / 100;
  emit('update:modelValue', isNaN(valueForParent) ? null : valueForParent);
  emit('input', event); // Mantém o evento 'input' original para compatibilidade
}

const handleFocus = () => {
  isFocused.value = true;
  // Ao focar, certifique-se de que o internalPriceInCentsString esteja correto
  // e sem zeros extras para a digitação.
  internalPriceInCentsString.value = String(Math.round((props.modelValue || 0) * 100)).padStart(3, '0');
};

const handleBlur = () => {
  isFocused.value = false;
  // Ao perder o foco, se o campo estiver '000', emita null ou 0.
  if (internalPriceInCentsString.value === '0' || internalPriceInCentsString.value === '00' || internalPriceInCentsString.value === '000') {
    internalPriceInCentsString.value = '000'; // Reseta para o padrão '0.00'
    emit('update:modelValue', 0); // Emite 0 para o pai
  } else {
    // Garante que o valor exibido tenha sempre 2 casas decimais ao perder o foco
    const numValue = parseFloat(internalPriceInCentsString.value) / 100;
    internalPriceInCentsString.value = String(Math.round(numValue * 100)).padStart(3, '0');
  }
};
</script>
