<template>
	<tr>
        <td>{{ playerName }}</td>
        <td>{{ playerHand }}</td>
        <td>{{ playerBestHand }}</td>
        <td class="cell__result" :class="resultClass">{{ playerResult }}</td>
    </tr>
</template>

<script>
export default {
	name: 'CardGame',
	props: {
        record: {type: Object, required: true, default: function() {return {}}}
	},
	components: {},
	data: function () {
		return {
		}
	},
	created() {
	},
	computed: {
        player() {
            return this.record || {};
        },
        playerName() {
            return this.player.playerName || "";
        },
        playerHand() {
            return this.player.deck ? this.beautifyDeck(this.player.deck) :  "";
        },
        playerBestHand() {
            return this.player.bestDeck ? this.beautifyDeck(this.player.bestDeck) :  "";
        },
        playerResult() {
            return this.player.isWinner ? "Winner! Congratulations!" : "Loser.";
        },
        resultClass() {
            return this.player.isWinner ? "cell__winner" : "cell__loser";
        }
	},
	methods: {
		beautifyDeck(deck) {
			return deck.join(", ");
		},
	},
	watch: {

	}
}
</script>

<style scope>
    .cell__result {
        font-weight: 500;
    }
    .cell__winner {
        color: green;
    }

    .cell__loser {
        color: red;
    }
</style>
