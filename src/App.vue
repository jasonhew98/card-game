<template>
	<div id="app">
		<CardGame/>
	</div>
</template>

<script>
import CardGame from './components/CardGame.vue'

export default {
	name: 'App',
	components: {
		CardGame
	}
}
</script>

<style>
	#app {
		width: 100vw;
		height: 100vh;
		padding: 10px 12px;
		background: linear-gradient(to right, #00A4E4 0%, #FEFEFE 100%);
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		font-size: 16px;
	}
</style>
