<template>
	<div>
	  <Bar :data="chartData" :options="chartOptions" />
	</div>
  </template>
  
  <script>
  import { Bar } from 'vue-chartjs';
  import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js';
  
  ChartJS.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale);
  
  export default {
	components: {
	  Bar
	},
	props: {
	  resultados: {
		type: Array,
		required: true
	  }
	},
	computed: {
	  chartData() {
		return {
		  labels: this.resultados.map(r => r.instituicao),
		  datasets: [
			{
			  label: 'Valor Solicitado',
			  data: this.resultados.map(r => r.valorSolicitado),
			  backgroundColor: 'rgba(75, 192, 192, 0.2)',
			  borderColor: 'rgba(75, 192, 192, 1)',
			  borderWidth: 1
			}
		  ]
		};
	  },
	  chartOptions() {
		return {
		  responsive: true,
		  plugins: {
			legend: {
			  position: 'top'
			},
			tooltip: {
			  callbacks: {
				label: function(context) {
				  let label = context.dataset.label || '';
				  if (label) {
					label += ': ';
				  }
				  if (context.parsed.y !== null) {
					label += context.parsed.y.toLocaleString();
				  }
				  return label;
				}
			  }
			}
		  }
		};
	  }
	}
  };
  </script>
  