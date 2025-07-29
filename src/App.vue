<template>
  <v-app class="app-background">
    <v-main>
      <v-container fluid class="pa-0">
        <h2 class="text-h3 font-weight-bold text-red-darken-1 mb-5 text-center">
          Simulador de Aquisição Imobiliária
        </h2>
        <div class="d-flex align-center justify-center">
          <v-card class="calculator-card" :elevation="10">
            <v-card-text class="pa-6">
              <v-row dense>
                <v-col cols="12" md="6">
                  <v-row dense class="mx-6">
                    <v-col cols="12">
                      <v-tooltip text="O valor atual de mercado do imóvel que você deseja adquirir." location="top">
                        <template v-slot:activator="{ props }">
                          <CurrencyInput
                            v-bind="props"
                            v-model="valorImovel"
                            label="Valor do Imóvel (hoje):"
                            prefix="R$"
                            clearable
                            variant="outlined"
                            color="primary"
                            density="compact"
                            hide-details
                            @input="calcular"
                          />
                        </template>
                      </v-tooltip>
                    </v-col>
      
                    <v-col cols="12">
                      <v-tooltip text="Seu montante financeiro disponível hoje para investir ou como entrada." location="top">
                        <template v-slot:activator="{ props }">
                          <CurrencyInput
                            v-bind="props"
                            v-model="saldoLiquidoDisponivel"
                            label="Saldo líquido disponível (hoje):"
                            prefix="R$"
                            clearable
                            variant="outlined"
                            density="compact"
                            hide-details
                            color="primary"
                            @input="calcular"
                          />
                        </template>
                      </v-tooltip>
                    </v-col>
      
                    <v-col cols="12">
                      <v-tooltip text="Quantia que você pode poupar e investir mensalmente." location="top">
                        <template v-slot:activator="{ props }">
                          <CurrencyInput
                            v-bind="props"
                            v-model="capacidadeMensalPoupanca"
                            label="Capacidade mensal de poupança (hoje):"
                            prefix="R$"
                            clearable
                            variant="outlined"
                            density="compact"
                            hide-details
                            color="primary"
                            @input="calcular"
                          />
                        </template>
                      </v-tooltip>
                    </v-col>
      
                    <v-col cols="12">
                      <v-tooltip text="Taxa de juros líquida média que você espera obter em seus investimentos (ex: CDI, poupança)." location="top">
                        <template v-slot:activator="{ props }">
                          <v-text-field
                            v-bind="props"
                            v-model.number="rendimentoLiquidoInvestimento"
                            label="Rendimento líquido do Investimento (mensal)"
                            type="number"
                            suffix="%"
                            clearable
                            variant="outlined"
                            density="compact"
                            hide-details
                            color="primary"
                            step="0.01"
                            @input="calcular"
                          ></v-text-field>
                        </template>
                      </v-tooltip>
                    </v-col>
      
                    <v-col cols="12">
                      <v-tooltip text="Estimativa de quanto o valor do imóvel pode crescer mensalmente no mercado." location="top">
                        <template v-slot:activator="{ props }">
                          <v-text-field
                            v-bind="props"
                            v-model.number="taxaValorizacaoImovel"
                            label="Taxa de Valorização do Imóvel (mensal)"
                            type="number"
                            suffix="%"
                            clearable
                            variant="outlined"
                            density="compact"
                            hide-details
                            color="primary"
                            step="0.01"
                            @input="calcular"
                          ></v-text-field>
                        </template>
                      </v-tooltip>
                    </v-col>
                  </v-row>
                </v-col>
                <v-col cols="12" md="6">
                  <h3 class="text-h6 font-weight-bold mb-3 primary--text">Resultados da Simulação:</h3>

                  <v-row dense class="results-row">
                    <v-col cols="12">
                      <v-card outlined class="pa-3 result-card">
                        <div class="text-subtitle-1 font-weight-bold">Tempo até conseguir o valor à vista:</div>
                        <div class="text-h6 primary--text">{{ formattedTempoAteConseguir }}</div>
                      </v-card>
                    </v-col>
                    <v-col cols="12">
                      <v-card outlined class="pa-3 result-card">
                        <div class="text-subtitle-1 font-weight-bold">Valor que precisa guardar:</div>
                        <div class="text-caption text-wrap">
                          {{ `Esse é o valor do imóvel que hoje vale ${formatCurrency(valorImovel || 0)}, depois ${formattedTempoAteConseguir} R$` }}
                        </div>
                        <div class="text-h6 primary--text">{{ formatCurrency(valorImovelNoFuturo) }}</div>
                      </v-card>
                    </v-col>
                    </v-row>
                </v-col>
              </v-row>
            </v-card-text>
            <v-card-actions class="pa-6 pt-0 text-caption text-medium-emphasis">
              <p>Este Simulador de Aquisição Imobiliária é uma ferramenta útil para ajudar você a entender melhor suas finanças. Lembre-se de que os resultados da simulação são apenas estimativas e podem variar com as circunstâncias do mercado. Ao usar o Simulador de Aquisição Imobiliária, você concorda que os resultados serão apenas para fins educativos. É importante validar as informações e considerar sua situação financeira antes de tomar qualquer decisão.</p>
            </v-card-actions>
          </v-card>
        </div>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import CurrencyInput from './components/CurrencyInput.vue';

const valorImovel = ref<number | null>(500000);
const saldoLiquidoDisponivel = ref<number | null>(50000);
const capacidadeMensalPoupanca = ref<number | null>(5000);

const rendimentoLiquidoInvestimento = ref<number>(0.77);
const taxaValorizacaoImovel = ref<number>(0.5);

const tempoAteConseguirEmMeses = ref<number>(0);
const valorImovelNoFuturo = ref<number>(0);

const formatCurrency = (value: number): string => {
  return new Intl.NumberFormat('pt-BR', {
    style: 'currency',
    currency: 'BRL',
  }).format(value);
};

// Computed property para formatar o tempo em anos e meses
const formattedTempoAteConseguir = computed(() => {
  if (tempoAteConseguirEmMeses.value === 0) return '0 meses';
  if (tempoAteConseguirEmMeses.value === Infinity) return 'Nunca'; // Caso não consiga atingir
  
  const anos = Math.floor(tempoAteConseguirEmMeses.value / 12);
  const meses = tempoAteConseguirEmMeses.value % 12;

  let result = '';
  if (anos > 0) {
    result += `${anos} ano${anos > 1 ? 's' : ''}`;
  }
  if (meses > 0) {
    if (result) result += ' e ';
    result += `${meses} mês${meses > 1 ? 'es' : ''}`;
  }
  return result || '0 meses';
});


// Função para realizar os cálculos da simulação
const calcular = () => {
  let vValorImovelAtual = valorImovel.value || 0;
  let vSaldoLiquidoAtual = saldoLiquidoDisponivel.value || 0;
  const vCapacidadeMensalPoup = capacidadeMensalPoupanca.value || 0;
  const vRendimentoMensal = (rendimentoLiquidoInvestimento.value || 0) / 100; // Converte para decimal
  const vTaxaValorizacao = (taxaValorizacaoImovel.value || 0) / 100; // Converte para decimal

  let meses = 0;
  const MAX_ITERATIONS = 600; // Limite de 50 anos para evitar loops infinitos

  // Resetar resultados
  tempoAteConseguirEmMeses.value = 0;
  valorImovelNoFuturo.value = 0;

  // Se o saldo inicial já for suficiente, o tempo é 0 e o valor futuro é o valor atual do imóvel
  if (vSaldoLiquidoAtual >= vValorImovelAtual && vValorImovelAtual > 0) {
    tempoAteConseguirEmMeses.value = 0;
    valorImovelNoFuturo.value = vValorImovelAtual;
    return;
  }

  // Se a capacidade de poupança e o rendimento forem zero e o saldo inicial não for suficiente,
  // ou se o imóvel não valoriza e o saldo e poupança não crescem o suficiente
  if (vCapacidadeMensalPoup === 0 && vRendimentoMensal <= 0 && vSaldoLiquidoAtual < vValorImovelAtual) {
    tempoAteConseguirEmMeses.value = Infinity; // Nunca vai conseguir
    return;
  }
  
  // Se a taxa de valorização for 0 e a poupança for 0, mas tem rendimento, ou se poupança positiva mas rendimento 0
  if (vCapacidadeMensalPoup === 0 && vRendimentoMensal <= 0 && vTaxaValorizacao <= 0 && vSaldoLiquidoAtual < vValorImovelAtual) {
    tempoAteConseguirEmMeses.value = Infinity;
    return;
  }


  while (vSaldoLiquidoAtual < vValorImovelAtual && meses < MAX_ITERATIONS) {
    meses++;
    
    // Atualiza o valor do imóvel com a valorização mensal
    vValorImovelAtual *= (1 + vTaxaValorizacao);
    
    // Acumula a poupança mensal e aplica o rendimento
    vSaldoLiquidoAtual = (vSaldoLiquidoAtual + vCapacidadeMensalPoup) * (1 + vRendimentoMensal);
    // vSaldoLiquidoAtual = (vSaldoLiquidoAtual * (1 + vRendimentoMensal)) + vCapacidadeMensalPoup;
    
    // Pequeno ajuste para evitar arredondamento flutuante causar loops infinitos próximos do limite
    if (Math.abs(vSaldoLiquidoAtual - vValorImovelAtual) < 0.01 && vSaldoLiquidoAtual >= vValorImovelAtual) {
      break; // Considera como atingido se a diferença for muito pequena e o saldo for maior
    }
  }

  // Se o loop terminou porque atingiu o MAX_ITERATIONS, significa que não conseguiu
  if (meses === MAX_ITERATIONS && vSaldoLiquidoAtual < vValorImovelAtual) {
    tempoAteConseguirEmMeses.value = Infinity;
    valorImovelNoFuturo.value = vValorImovelAtual; // Valor do imóvel no final da simulação
  } else {
    tempoAteConseguirEmMeses.value = meses;
    valorImovelNoFuturo.value = vValorImovelAtual;
  }
};

// Chamar a função calcular na inicialização para ter valores default
calcular();

</script>

<style scoped>
/* O CSS permanece o mesmo do exemplo anterior */
.app-background {
  background: linear-gradient(135deg, #fafafa 0%, #f5f5f5 100%);
}

.calculator-card {
  width: 90%;
  max-width: 900px; /* Aumenta a largura máxima para acomodar 2 colunas em desktop */
  border-radius: 15px;
  overflow: hidden;
}

/* Removido o toolbar-title do CSS pois foi removido o v-toolbar no template */

.v-card-text {
  background-color: #ffffff;
}

.v-text-field {
  margin-bottom: 10px;
}

.results-row .v-card {
  border-left: 5px solid #FF0000;
}

.result-card {
  background-color: #ffebee;
}

/* Ajustes para desktop para que os campos e resultados caibam sem scroll */
@media (min-width: 960px) { /* breakpoint md do Vuetify */
  .v-main {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .calculator-card {
    height: auto; /* Permite que a altura se ajuste ao conteúdo */
    max-height: 90vh; /* Limita a altura para garantir que caiba na tela */
    overflow-y: auto; /* Adiciona scroll apenas se exceder a altura máxima */
  }
  .v-container.fluid {
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
}
</style>