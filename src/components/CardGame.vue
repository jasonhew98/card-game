<template>
	<div class="content__container">
		<div class="page__header">Card Game</div>
		<div class="table__container">
			<table>
				<tr>
					<th v-for="tableHeader in tableHeaders" v-bind:key="tableHeader">{{ tableHeader }}</th>
				</tr>
				<result-table-row v-for="player in players" v-bind:key="player.playerName" :record="player"></result-table-row>
			</table>
		</div>
		<div class="action__container"><button class="btn__reshuffle" @click="reshuffle">Re-shuffle</button></div>
	</div>
</template>

<script>
import _ from 'lodash';
import ResultTableRow from './ResultTableRow.vue';

const FACEVALUES = {
	'2': '2',
	'3': '3',
	'4': '4',
	'5': '5',
	'6': '6',
	'7': '7',
	'8': '8',
	'9': '9',
	'10': '10',
	'11': 'J',
	'12': 'Q',
	'13': 'K',
	'14': 'A'
}

const FACEVALUESREVERSE = {
	'2': '2',
	'3': '3',
	'4': '4',
	'5': '5',
	'6': '6',
	'7': '7',
	'8': '8',
	'9': '9',
	'10': '10',
	'J': '11',
	'Q': '12',
	'K': '13',
	'A': '14' 
};

const WIN_CONDITION = {
	DEFAULT: 0,
	HIGH_COUNT: 1,
	HIGH_FACE_VALUE: 2,
	HIGH_SYMBOL_VALUE: 3
};

const SYMBOLS = { "@": 1, "#": 2, "^": 3, "*": 4 };

export default {
	name: 'CardGame',
	props: {
	},
	components: {ResultTableRow},
	data: function () {
		return {
			playerCount: 4,
			cardCount: 13,
			players: [],
			cards: [
				"2@", "2#", "2^", "2*",
				"3@", "3#", "3^", "3*",
				"4@", "4#", "4^", "4*",
				"5@", "5#", "5^", "5*",
				"6@", "6#", "6^", "6*",
				"7@", "7#", "7^", "7*",
				"8@", "8#", "8^", "8*",
				"9@", "9#", "9^", "9*",
				"10@", "10#", "10^", "10*",
				"J@", "J#", "J^", "J*",
				"Q@", "Q#", "Q^", "Q*",
				"K@", "K#", "K^", "K*",
				"A@", "A#", "A^", "A*",
			],
			tableHeaders: [
				"Player Name",
				"Player's Hand(s)",
				"Player's Best Hand(s)",
				"Result"
			]
		}
	},
	created() {
		this.reshuffle();
	},
	computed: {
	},
	methods: {
		beautifyDeck(deck) {
			return deck.join(", ");
		},
		setCurrentWinner({currentWinner, index, fullDeck, winningDeck, winningCardCount, winCondition}) {
			currentWinner.index = index;
			currentWinner.fullDeck = fullDeck;
			currentWinner.winningDeck = winningDeck;
			currentWinner.winningCardCount = winningCardCount;
			currentWinner.winCondition = winCondition;

			return currentWinner;
		},
		reshuffle() {
			let players = [];
			let tempCards = [...this.cards];

			for (let i = 0; i < this.playerCount; i++) {
				let filteredCards = [];
				
				for (let i = 0; i < this.cardCount; i++) {
					let card = tempCards.splice(Math.floor(Math.random()*tempCards.length), 1);
					filteredCards.push(card[0]);
				}

				filteredCards.sort((a, b) => this.compareCards(a, b));
				let winningCardCount = this.countWinningCards(filteredCards);

				let bestDeck = filteredCards.filter(x => x.startsWith(FACEVALUES[winningCardCount.faceCard]));
				
				players.push(
					{
						playerName: `Player ${i + 1}`,
						deck: filteredCards,
						winningCardCount: winningCardCount,
						bestDeck: bestDeck,
						isWinner: false
					}
				);
			}

			let currentWinner = {};

			for (let [i, player] of players.entries()) {
				let deck = player.deck;
				let winningCardCount = player.winningCardCount;
				var winningCards = player.bestDeck;

				if (_.isEmpty(currentWinner)) {
					currentWinner = this.setCurrentWinner({
						currentWinner: currentWinner,
						index: i,
						fullDeck: deck,
						winningDeck: winningCards,
						winningCardCount: winningCardCount,
						winCondition: WIN_CONDITION.DEFAULT});
					continue;
				}

				// // If contender doesn't have better count, reigning champ remains
				if (winningCardCount.count < currentWinner.winningCardCount.count) {
					continue;
				}

				// // If max is more than current best, assign new best because more cards more eligible to win
				if (winningCardCount.count > currentWinner.winningCardCount.count) {
					currentWinner = this.setCurrentWinner({
						currentWinner: currentWinner,
						index: i,
						fullDeck: deck,
						winningDeck: winningCards,
						winningCardCount: winningCardCount,
						winCondition: WIN_CONDITION.HIGH_COUNT});
				}

				// If same max number, check the face card
				if (winningCardCount.count == currentWinner.winningCardCount.count) {
					// Check face card
					if (winningCardCount.faceCard == currentWinner.winningCardCount.faceCard) {
						// If same face card, check symbol
						let winningCardsSymbolValue = winningCards.map(x => this.getSymbolValue(x));

						let currentWinnerCards = currentWinner.fullDeck.filter(x => x.startsWith(FACEVALUES[currentWinner.winningCardCount.faceCard]));
						let currentWinnerCardsSymbolValue = currentWinnerCards.map(x => this.getSymbolValue(x));

						if (Math.max(...winningCardsSymbolValue) > Math.max(...currentWinnerCardsSymbolValue)) {
							currentWinner = this.setCurrentWinner({
								currentWinner: currentWinner,
								index: i,
								fullDeck: deck,
								winningDeck: winningCards,
								winningCardCount: winningCardCount,
								winCondition: WIN_CONDITION.HIGH_SYMBOL_VALUE});
						}
					} else {
						// If different face, check whichever has highest value
						let winningCardsFaceValue = winningCards.map(x => this.getValue(x))[0];

						let currentWinnerCards = currentWinner.fullDeck.filter(x => x.startsWith(FACEVALUES[currentWinner.winningCardCount.faceCard]));
						let currentWinnerCardsFaceValue = currentWinnerCards.map(x => this.getValue(x))[0];

						if (winningCardsFaceValue > currentWinnerCardsFaceValue) {
							currentWinner = this.setCurrentWinner({
								currentWinner: currentWinner,
								index: i,
								fullDeck: deck,
								winningDeck: winningCards,
								winningCardCount: winningCardCount,
								winCondition: WIN_CONDITION.HIGH_SYMBOL_VALUE});
						}
					}
				}
			}

			let finalPlayers = players.map((x, index) => {
				if (index == currentWinner.index) {
					x.isWinner = true;
					x.winCondition = currentWinner.winCondition;
				}

				return x;
			});

			this.players = finalPlayers;
		},
		isWinner(player) {
			let winCondition = "";

			switch (player.winCondition) {
				case WIN_CONDITION.DEFAULT:
					winCondition = "Won by default";
					break;
				case WIN_CONDITION.HIGH_COUNT:
					winCondition = "Won by having highest number of similar cards";
					break;
				case WIN_CONDITION.HIGH_FACE_VALUE:
					winCondition = "Won by having highest number of similar cards and highest face value";
					break;
				case WIN_CONDITION.HIGH_SYMBOL_VALUE:
					winCondition = "Won by having highest number of similar cards and highest symbol value";
					break;
			}

			return player.isWinner ? `Winner! ${winCondition}.\n Winning Deck: ${player.bestDeck}` : `Loser. Best Cards: ${player.bestDeck}`;
		},
		// Find the best cards in a deck
		countWinningCards(deck) {
			const counts = {};
			let maxCount = 0;

			for (const card of deck) {
				const value = this.getValue(card);
				counts[value] = (counts[value] || 0) + 1;
				maxCount = Math.max(maxCount, counts[value]);
			}
			
			let maxPropertyValue = null;
			let maxPropertyValueKey = null;

			for (const key in counts) {
				if (counts.hasOwnProperty(key)) {
					const value = counts[key];

					if (maxPropertyValue === null || value > maxPropertyValue) {
						maxPropertyValue = value;
						maxPropertyValueKey = key;
						continue;
					}

					if (value == maxPropertyValue && parseInt(key) >  parseInt(maxPropertyValueKey)) {
						maxPropertyValue = value;
						maxPropertyValueKey = key;
					}
				}
			}

			return {
				faceCard: maxPropertyValueKey,
				count: maxPropertyValue
			};
		},
		// Get Numeric Value for Alphanumeric
		getValue(card) {
			const numericValue = parseInt(card);
			return isNaN(numericValue) ? this.getFaceCardValue(card.toString().charAt(0)) : numericValue;
		},
		// Get Numeric Value for Face(s)
		getFaceCardValue(face) {
			return FACEVALUESREVERSE[face];
		},
		// Get Numeric Value for Symbol(s)
		getSymbolValue(card) {
			return SYMBOLS[card.toString().charAt(card.length - 1)];
		},
		// Sort the cards
		compareCards(cardA, cardB) {
			const valueA = this.getValue(cardA);
			const valueB = this.getValue(cardB);

			if (valueA !== valueB) {
				return valueB - valueA;
			} else {
				return this.getSymbolValue(cardB) - this.getSymbolValue(cardA);
			}
		}
	},
	watch: {

	}
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
	.btn__reshuffle {
		font-weight: 550;
		padding: 8px 12px;
		color: #FFFFFF;
		background-color: #0066cc;
		border: none;
		border-radius: 4px;
		box-shadow: 0 8px 16px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19);
		-webkit-transition-duration: 0.4s;
  		transition-duration: 0.4s;
		opacity: 1;
	}

	.btn__reshuffle:hover {
		opacity: 0.8;
	}

	.content__container {
		width: 100%;
		height: 100%;
	}

	.content__container .table__container {
		width: 100%;
		margin: auto;
	}

	table {
		width: 100%;
		max-width: 100%;
		border: 1px solid #ddd;
		table-layout: fixed;
		border-radius: 10px;
		margin-bottom: 20px;
	}

	table td > div {
		height: 65px;
		padding: 8px;
	}

	table, th, td {
		border: 1px solid black;
		border-collapse: collapse;
		text-align: center;
		padding: 8px;
	}

	.page__header {
		font-size: 21px;
		font-weight: bold;
		margin-bottom: 18px;
	}

	.footer {
		margin-top: 15px;
	}
</style>
