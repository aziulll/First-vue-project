<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from 'chart.js/auto'

const weights = ref([]);
const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weighInput = ref(60.0);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weigth: 0 };
});

const addWeight = () => {
  weights.value.push({
    weights: weighInput.value,
    date: new Date().getTime(),
  });
};

watch(weights, (newWeights) => {
	const ws = [...newWeights]

	if (weightChart.value) {
		weightChart.value.data.labels = ws
			.sort((a, b) => a.date - b.date)
			.map(weight => new Date(weight.date).toLocaleDateString() )
			.slice(-7)

		weightChart.value.data.datasets[0].data = ws
			.sort((a, b) => a.date - b.date)
			.map(weight => weight.weight)
			.slice(-7)

		weightChart.value.update()
		return
	}

	nextTick(() => {
		weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
			type: 'line',
			data: {
				labels: ws
					.sort((a, b) => a.date - b.date)
					.map(weight => new Date(weight.date).toLocaleDateString()),
				datasets: [
					{
						label: 'Weight',
						data: ws
							.sort((a, b) => a.date - b.date)
							.map(weight => weight.weight),
						backgroundColor: 'rgba(255, 105, 180, 0.2)',
						borderColor: 'rgba(255, 105, 180, 1)',
						borderWidth: 1,
						fill: true
					}
				]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false
			}
		})
	})
}, { deep: true })
</script>

<template>
  <h1>Weight Tracker</h1>

  <div class="current">
    <span>{{ currentWeight.weigth }}</span>
    <small>current Weight (kg)</small>
  </div>

  <form @submit.prevent="addWeight">
    <input type="number" step="0.1" v-model="weighInput" />
    <input type="submit" value="Adicione o seu peso" />
  </form>

  <div v-if="weights && weights.length > 0">
    <h2>Últimos 7 dias</h2>
    <div class="cavnas-box">
      <canvas ref="weightChartEl"></canvas>
    </div>

    <div class="weight-history">
      <h2>Histórico de pesos</h2>

      <ul>
        <li v-for="(weight, index) in weights" :key="index">
          <span> {{ weight.weights }} kg </span>
          <small>Feito: {{ new Date(weight.date).toLocaleDateString() }}</small>
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
