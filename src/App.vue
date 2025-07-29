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
                      <v-tooltip text="Taxa de juros líquida média que você espera obter em seus investimentos (ex: CDI, tesouro IPCA, etc)." location="top">
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
                          {{ `Esse é o valor do imóvel que hoje vale ${formatCurrency(valorImovel || 0)}, depois ${formattedTempoAteConseguir}` }}
                        </div>
                        <div class="text-h6 primary--text">{{ formatCurrency(valorImovelNoFuturo) }}</div>
                      </v-card>
                    </v-col>

                  </v-row>
                </v-col>

                <v-col cols="12" class="mt-4">
                  <v-card outlined class="pa-3 result-card">
                    <div class="text-subtitle-1 font-weight-bold mb-2">Progresso para o Objetivo:</div>
                    <v-progress-linear
                      :model-value="progressPercentage"
                      :color="progressColor"
                      height="20"
                      rounded
                      striped
                    >
                      <template v-slot:default="{ value }">
                        <strong>{{ value.toFixed(1) }}%</strong>
                      </template>
                    </v-progress-linear>
                    <div class="text-caption mt-2 text-center" v-if="tempoAteConseguirEmMeses !== Infinity">
                      {{ progressStatusText }}
                    </div>
                  </v-card>
                </v-col>

                <v-col cols="12" class="mt-4">
                  <v-card outlined class="pa-3 result-card">
                    <div class="text-subtitle-1 font-weight-bold mb-2">Evolução Patrimonial e do Imóvel:</div>
                    <div id="chart">
                      <ApexChart type="line" :options="chartOptions" :series="chartSeries" />
                    </div>
                  </v-card>
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
import ApexChart from 'vue3-apexcharts'

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
    result += `${meses} ${meses > 1 ? 'meses' : 'mês'}`;
  }
  return result || '0 meses';
});

const progressPercentage = computed(() => {
  const currentImovelValue = valorImovel.value || 0;
  const currentSaldo = saldoLiquidoDisponivel.value || 0;
  
  // Se não há valor do imóvel ou o saldo já é suficiente, o progresso é 100%
  if (currentImovelValue <= 0 || currentSaldo >= valorImovelNoFuturo.value) {
    return 100;
  }

  // Se o valor futuro do imóvel for zero (ainda não calculado ou erro), evita divisão por zero
  if (valorImovelNoFuturo.value <= 0) {
    return 0;
  }

  // Calcula a porcentagem do saldo atual em relação ao valor futuro do imóvel
  // ajustado pela capacidade de poupança no primeiro mês.
  // Vamos usar o saldo inicial + a poupança do primeiro mês como o "saldo atual que já contribuiu".
  const effectiveCurrentSaldo = currentSaldo + (capacidadeMensalPoupanca.value || 0); // Considera o aporte do 1o mês como parte do saldo inicial para a %

  let percentage = (effectiveCurrentSaldo / valorImovelNoFuturo.value) * 100;

  // Garante que a porcentagem não exceda 100% ou seja menor que 0%
  percentage = Math.max(0, Math.min(100, percentage));

  return percentage;
});

const progressColor = computed(() => {
  if (progressPercentage.value < 25) return 'red-darken-2';
  if (progressPercentage.value < 50) return 'orange';
  if (progressPercentage.value < 75) return 'light-green';
  return 'green';
});

const progressStatusText = computed(() => {
  if (progressPercentage.value >= 100) {
    return 'Parabéns! Você já atingiu (ou superou) o valor necessário para o imóvel!';
  }
  
  // Se o cálculo indica que nunca vai conseguir
  if (tempoAteConseguirEmMeses.value === Infinity) {
    return 'Com as informações atuais, pode ser difícil atingir o objetivo. Considere aumentar a poupança ou o rendimento.';
  }

  const anosRestantes = tempoAteConseguirEmMeses.value / 12;

  if (anosRestantes < 2) {
    return 'Vejo que você está próximo do seu objetivo! Continue poupando.';
  } else { // Implicitamente, se for 2 anos ou mais
    return 'Talvez você precise de um planejamento patrimonial! Clique aqui para saber mais.';
  }
});

const chartData = ref<{ month: number; saldo: number; imovel: number; cdi: number; dateLabel: string }[]>([]);
const cdiRate = 0.01; // Taxa CDI hipotética de 1% ao mês (ajuste conforme necessário para seu comparativo)

const chartSeries = computed(() => {
  return [
    {
      name: 'Seu Saldo Total',
      type: 'bar', // Pode ser 'bar' ou 'line'
      data: chartData.value.map(item => item.saldo),
    },
    {
      name: 'Valor do Imóvel',
      type: 'line',
      data: chartData.value.map(item => item.imovel),
    },
    {
      name: '100% do CDI (Comparativo)',
      type: 'line',
      data: chartData.value.map(item => item.cdi),
    }
  ];
});

const chartOptions = computed(() => {
  return {
    chart: {
      height: 350,
      type: 'line',
      stacked: false,
      zoom: {
        enabled: true,
      },
      toolbar: { show: false }
    },
    dataLabels: {
      enabled: false,
    },
    stroke: {
      width: [3, 3, 3],
      curve: 'smooth',
    },
    xaxis: {
      categories: chartData.value.map(item => item.dateLabel), // Alterado para usar dateLabel
      title: {
        text: 'Tempo', // Pode ser apenas "Tempo" já que a data está no tooltip
      },
      labels: {
        formatter: function(val: string, index: number) {
          // Mantemos a exibição a cada 6 meses no eixo X
          if (index % 6 === 0) { 
            return val;
          }
          return '';
        }
      }
    },
    yaxis: {
      title: {
        text: 'Valor (R$)',
      },
      labels: {
        formatter: function (val: number) {
          return val.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL', maximumFractionDigits: 0 });
        },
      },
    },
    tooltip: {
      // NOVO: Formatar o eixo X no tooltip
      x: {
        formatter: function(val: string, { dataPointIndex }: { dataPointIndex: any}) {
          // O 'val' aqui já será a string do dateLabel que passamos em categories
          return chartData.value[dataPointIndex].dateLabel;
        }
      },
      y: {
        formatter: function (val: number) {
          return val.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
        },
      },
    },
    fill: {
      opacity: [0.85, 0.25, 0.25],
      gradient: {
        inverseColors: false,
        shade: 'light',
        type: 'vertical',
        opacityFrom: 0.85,
        opacityTo: 0.55,
        stops: [0, 100, 100, 100]
      }
    },
    colors: ['#EF5350', '#424242', '#3F51B5'],
    markers: {
      size: 0
    },
  };
});

// FUNÇÃO CALCULAR - AJUSTADA PARA GERAR DATAS NO GRÁFICO
const calcular = () => {
  let vValorImovelAtual = valorImovel.value || 0;
  let vSaldoLiquidoAtual = saldoLiquidoDisponivel.value || 0;
  const vCapacidadeMensalPoup = capacidadeMensalPoupanca.value || 0;
  const vRendimentoMensal = (rendimentoLiquidoInvestimento.value || 0) / 100;
  const vTaxaValorizacao = (taxaValorizacaoImovel.value || 0) / 100;

  let meses = 0;
  const MAX_ITERATIONS = 600;

  // Resetar resultados e dados do gráfico
  tempoAteConseguirEmMeses.value = 0;
  valorImovelNoFuturo.value = 0;
  chartData.value = [];

  let vSaldoCdi = saldoLiquidoDisponivel.value || 0; 
  
  // Obter a data atual para iniciar o cálculo dos meses
  const startDate = new Date();

  // Função auxiliar para formatar a data (MM/YYYY)
  const formatDate = (date: Date): string => {
    const month = (date.getMonth() + 1).toString().padStart(2, '0'); // Mês de 0 a 11
    const year = date.getFullYear();
    return `${month}/${year}`;
  };

  // Adicionar o ponto inicial (Mês 0) para o gráfico
  chartData.value.push({
    month: 0,
    saldo: vSaldoLiquidoAtual,
    imovel: vValorImovelAtual,
    cdi: vSaldoCdi,
    dateLabel: formatDate(startDate), // Data inicial
  });

  if (vSaldoLiquidoAtual >= vValorImovelAtual && vValorImovelAtual > 0) {
    tempoAteConseguirEmMeses.value = 0;
    valorImovelNoFuturo.value = vValorImovelAtual;
    return;
  }

  if (vCapacidadeMensalPoup === 0 && vRendimentoMensal <= 0 && vSaldoLiquidoAtual < vValorImovelAtual) {
    tempoAteConseguirEmMeses.value = Infinity;
    return;
  }
  
  if (vCapacidadeMensalPoup === 0 && vRendimentoMensal <= 0 && vTaxaValorizacao <= 0 && vSaldoLiquidoAtual < vValorImovelAtual) {
    tempoAteConseguirEmMeses.value = Infinity;
    return;
  }

  while (vSaldoLiquidoAtual < vValorImovelAtual && meses < MAX_ITERATIONS) {
    meses++;
    
    // Atualiza o valor do imóvel para o próximo mês
    vValorImovelAtual *= (1 + vTaxaValorizacao);
    
    // Calcula o saldo total com aporte e rendimento
    vSaldoLiquidoAtual = (vSaldoLiquidoAtual + vCapacidadeMensalPoup) * (1 + vRendimentoMensal);

    // Calcula o saldo para o comparativo CDI
    vSaldoCdi = (vSaldoCdi + vCapacidadeMensalPoup) * (1 + cdiRate);
    
    // Calcula a data para o mês atual no loop
    const currentDate = new Date(startDate);
    currentDate.setMonth(startDate.getMonth() + meses); // Adiciona 'meses' ao mês inicial

    // Adiciona os dados para o gráfico a cada mês
    chartData.value.push({
      month: meses,
      saldo: parseFloat(vSaldoLiquidoAtual.toFixed(2)),
      imovel: parseFloat(vValorImovelAtual.toFixed(2)),
      cdi: parseFloat(vSaldoCdi.toFixed(2)),
      dateLabel: formatDate(currentDate), // Adiciona a data formatada
    });

    if (Math.abs(vSaldoLiquidoAtual - vValorImovelAtual) < 0.01 && vSaldoLiquidoAtual >= vValorImovelAtual) {
      break;
    }
  }

  if (meses === MAX_ITERATIONS && vSaldoLiquidoAtual < vValorImovelAtual) {
    tempoAteConseguirEmMeses.value = Infinity;
    valorImovelNoFuturo.value = vValorImovelAtual;
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