<template>
	<div id="app">
		<img alt="Vue logo" src="./assets/logo.png">
		<calculator/>
	</div>
</template>

<script lang="ts">
import Vue from 'vue'
import calculator from './components/calculator.vue'

export default Vue.extend({
	name: 'app',
	components: {
		calculator,
	},
})
</script>

<style lang="scss">
#app {
	font-family: "Avenir", Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
