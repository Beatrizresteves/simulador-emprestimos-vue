<template>
	<div class="simulador-form">
	  <form @submit.prevent="enviarSimulacao">
		<div>
		  <label for="valor-emprestimo">Valor do Empréstimo:</label>
		  <input
			id="valor-emprestimo"
			v-model="valorEmprestimo"
			type="text"
			v-mask="'#.#00,00'"
			required
			@input="validarValorEmprestimo"
		  />
		  <p v-if="!valorEmprestimoValido" class="error-message">Valor inválido. Use o formato correto.</p>
		</div>
  
		<div>
		  <label for="instituicoes">Instituições:</label>
		  <select
			id="instituicoes"
			v-model="instituicoesSelecionadas"
			multiple
		  >
			<option
			  v-for="instituicao in instituicoes"
			  :key="instituicao.id"
			  :value="instituicao.id"
			>
			  {{ instituicao.nome }}
			</option>
		  </select>
		</div>
  
		<div>
		  <label for="convenios">Convênios:</label>
		  <select
			id="convenios"
			v-model="conveniosSelecionados"
			multiple
		  >
			<option
			  v-for="convenio in convenios"
			  :key="convenio.id"
			  :value="convenio.id"
			>
			  {{ convenio.nome }}
			</option>
		  </select>
		</div>
  
		<div>
		  <label for="parcelas">Parcelas:</label>
		  <select
			id="parcelas"
			v-model="parcelaSelecionada"
		  >
			<option
			  v-for="parcela in parcelas"
			  :key="parcela"
			  :value="parcela"
			>
			  {{ parcela }}
			</option>
		  </select>
		</div>
  
		<button type="submit" :disabled="enviando">Simular</button>
		<div v-if="enviando" class="loading-indicator">Carregando...</div>
	  </form>
  
	  <div v-if="resultados.length">
		<h2>Resultados</h2>
		<div v-for="resultado in resultados" :key="resultado.id">
		  <h3>{{ resultado.instituicao }}</h3>
		  <p>Valor Solicitado: {{ resultado.valorSolicitado }}</p>
		  <p><strong>{{ resultado.quantidadeParcelas }} x {{ resultado.valor }}</strong></p>
		  <p>Taxa de Juros ao Mês: {{ resultado.taxaJuros }}</p>
		</div>
  
		<ResultadosChart v-if="resultados.length" :resultados="resultados" />
	  </div>
	</div>
  </template>
  
  <script>
  import axios from 'axios';
  import { mask } from 'vue-the-mask';
  import ResultadosChart from './ResultadosChart.vue';
  
  export default {
	components: {
	  ResultadosChart
	},
	data() {
	  return {
		valorEmprestimo: '',
		valorEmprestimoValido: true,
		instituicoes: [],
		instituicoesSelecionadas: [],
		convenios: [],
		conveniosSelecionados: [],
		parcelas: [36, 48, 60, 72, 84],
		parcelaSelecionada: null,
		resultados: [],
		enviando: false
	  };
	},
	methods: {
	  validarValorEmprestimo() {
		const regex = /^(\d{1,3}(?:\.\d{3})*|\d+)(,\d{2})?$/;
		this.valorEmprestimoValido = regex.test(this.valorEmprestimo);
	  },
	  async enviarSimulacao() {
		if (!this.valorEmprestimoValido) {
		  return;
		}
  
		this.enviando = true;
		const payload = {
		  valor_emprestimo: parseFloat(this.valorEmprestimo.replace(/\./g, '').replace(',', '.')),
		  instituicoes: this.instituicoesSelecionadas,
		  convenios: this.conveniosSelecionados,
		  parcela: this.parcelaSelecionada
		};
  
		try {
		  const response = await axios.post('http://localhost:8000/api/simular', payload);
		  this.resultados = response.data;
		} catch (error) {
		  console.error(error);
		} finally {
		  this.enviando = false;
		}
	  }
	},
	async created() {
	  try {
		const [instituicoesResponse, conveniosResponse] = await Promise.all([
		  axios.get('http://localhost:8000/api/instituicao'),
		  axios.get('http://localhost:8000/api/convenio')
		]);
		this.instituicoes = instituicoesResponse.data;
		this.convenios = conveniosResponse.data;
	  } catch (error) {
		console.error(error);
	  }
	}
  };
  </script>
  
  <style scoped>
  .simulador-form {
	max-width: 600px;
	margin: 0 auto;
	padding: 20px;
	border: 1px solid #ccc;
	border-radius: 8px;
	background-color: #f9f9f9;
  }
  
  .simulador-form form {
	display: flex;
	flex-direction: column;
  }
  
  .simulador-form div {
	margin-bottom: 15px;
  }
  
  .simulador-form label {
	font-weight: bold;
	margin-bottom: 5px;
	display: block;
  }
  
  .simulador-form input,
  .simulador-form select {
	width: 100%;
	padding: 8px;
	border: 1px solid #ccc;
	border-radius: 4px;
	box-sizing: border-box;
  }
  
  .simulador-form button {
	padding: 10px 15px;
	background-color: #007bff;
	color: white;
	border: none;
	border-radius: 4px;
	cursor: pointer;
	font-size: 16px;
  }
  
  .simulador-form button:disabled {
	background-color: #007bff;
	cursor: not-allowed;
  }
  
  .simulador-form button:hover {
	background-color: #0056b3;
  }
  
  .simulador-form .loading-indicator {
	margin-top: 10px;
	color: #007bff;
  }
  
  .simulador-form .error-message {
	color: red;
	font-size: 12px;
	margin-top: 5px;
  }
  
  .simulador-form h2 {
	margin-top: 20px;
  }
  
  .simulador-form .resultados {
	margin-top: 20px;
  }
  
  .simulador-form .resultados h3 {
	margin-bottom: 5px;
	font-size: 18px;
  }
  
  .simulador-form .resultados p {
	margin: 5px 0;
  }
  </style>
  