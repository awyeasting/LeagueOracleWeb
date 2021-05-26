<template>
	<div>
		<table class="teamSelection">
			<tbody>
				<tr>
					<td>
						<ul class="predictTeam">
							<li v-for="i in team1.length" :key="i"> 
								<img :src="getChampPortrait(team1[i-1])" :style="'border:' + getSelected(0,i-1)" @click="changeSelected(0, i-1)">
							</li>
						</ul>
					</td>
					<td>
						<div id="champSearch">
							<input v-model="search" placeholder="Search" />
						</div>
						<ul class="champSelection">
							<li v-for="(champ, name) in champ_name_map" :key="champ">
								<img v-if="searchName(name.toLowerCase())" :src="getNamePortrait(name)" @click="selectChampion(ichamp_map[champ.toString()])">
								
							</li>
						</ul>
					</td>
					<td>
						<ul class="predictTeam">
							<li v-for="i in team2.length" :key="i"> 
								<img :src="getChampPortrait(team2[i-1])" :style="'border:' + getSelected(1,i-1)" @click="changeSelected(1, i-1)">
							</li>
						</ul>
					</td>
				</tr>
			</tbody>
		</table>
		<div class="predictionOutcome" v-if="outcome">
			Outcome: <span :class="(this.outcome == 0.5 ? 'outcomeSide' : (this.outcome >= 0.5 ? 'outcomeSide blueSideText' : 'outcomeSide redSideText'))">{{ outcome == 0.5 ? 'Unknown' : (outcome >= 0.5 ? "Blue side win" : "Red side win")}}</span>
			<br/>
			<span class="blueSideText">Blue side chance: {{ blueSideChance }}%</span>
			<br/>
			<span class="redSideText">Red side chance: {{ redSideChance }}%</span>
		</div>
	</div>
</template>

<script>
import axios from 'axios'

import Constants from "./Constants.vue"
import champ_map from "../assets/ChampionMaps/champion_map.json"
import i_champ_map from "../assets/ChampionMaps/inv_internal_champion_map.json"
import ichamp_map from "../assets/ChampionMaps/internal_champion_map.json"
import champ_name_map from "../assets/ChampionMaps/champion_name_map.json"
export default {
	name: 'Predict',
	data: () => ({
		team1: [-1,-1,-1,-1,-1],
		team2: [-1,-1,-1,-1,-1],
		champ_map: champ_map,
		i_champ_map: i_champ_map,
		ichamp_map: ichamp_map,
		champ_name_map: champ_name_map,
		selected: [0,0],
		outcome: null,
		search: "",
	}),
	computed: {
		blueSideChance: function() {
			return (this.outcome*100).toFixed(1)
		},
		redSideChance: function() {
			return ((1-this.outcome)*100).toFixed(1)
		}
	},
	methods: {
		getChampPortrait: function(ichampId) {
			if (ichampId < 0)
				return require("../assets/ChampionSquares/Unknown.png")

			return require("../assets/ChampionSquares/" + champ_map[i_champ_map[ichampId]] + '.png')
		},
		getNamePortrait: function(champName) {
			return require("../assets/ChampionSquares/" + champName + '.png')
		},
		getSelected: function(team, playeri) {
			if (team == this.selected[0] && playeri == this.selected[1])
				return "2px solid white"
			return "2px solid #333"
		},
		changeSelected: function(team, playeri) {
			this.selected[0] = team
			this.selected[1] = playeri
		},
		selectChampion: function(ichampId) {
			console.log(ichampId)
			if (this.selected[0]) {
				this.team2[this.selected[1]] = ichampId
			} else {
				this.team1[this.selected[1]] = ichampId
			}
			this.outcome = null
			this.search = ""

			this.selected[1] += 1
			this.selected[0] = (this.selected[0] + Math.floor(this.selected[1]/5)) % 2
			this.selected[1] %= 5
			this.predictModel()
		},
		searchName: function(champName) {
			return champName.includes(this.search.toLowerCase())
		},
		predictModel: function() {
			for (var j=0; j < 5; j++) {
				if (this.team1[j] < 0)
					return
				else if (this.team2[j] < 0)
					return
			}
			var players = []
			for (j = 0; j < 5; j++)
				players.push({"champion": i_champ_map[this.team1[j]], "team": 0})
			for (j = 0; j < 5; j++)
				players.push({"champion": i_champ_map[this.team2[j]], "team": 1})
			var games = {"games":[{"players":players}]}
			console.log(this.model)
			console.log(Constants.HOSTNAME)
			console.log(games)
			var headers = {'Content-Type':'application/json'}
			axios.post(Constants.HOSTNAME + '/predict', games, headers).then( response => {
				this.outcome = parseFloat(response.data.predictions[0].avg_blue_chance)
				console.log(this.outcome)
			})
		}
	}
}
</script>

<style>
* {
	box-sizing: border-box;
}
body {
	padding:0;
	margin: 0;
	background: #111
}
ul {
	list-style-type: none;
	margin: 0;
	padding: 0;
}
table {
	width: 100%;
}
td {
	background: #000;
	padding: 2vh;
	height: 70vh;
}
.champSelection img {
	width: 8vh;
}
.champSelection li {
	display: inline-block;
}
.predictTeam {
	display: inline-block;
	background: #000;
}
.predictTeam img {
	width: 10vh;
	border: .1vh solid #333;
	border-radius: 100%;
	margin-top: 1vh;
	margin-bottom: 1vh;
}
#champSearch {
	display: block;
	padding-bottom:1vh;
	padding-top:1vh;
}
#champSearch input {
	background: #111;
	color: #FFF;
	padding: 1vh;
	border: none;
	width: 30vw;
	min-width: 300px;
	height: 40px;
	text-align: center;
	font-size: 20px;
}
.champSelection {
	display: block;
	height: 100%;
	max-height: calc(70vh - (40px + 4vh));
	overflow-x: hidden;
	overflow-y: auto;
}
/* width */
::-webkit-scrollbar {
  width: 1vw;
}

/* Track */
::-webkit-scrollbar-track {
  background: #222;
}

/* Handle */
::-webkit-scrollbar-thumb {
  background: #444;
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: #666;
}
.predictionOutcome {
	background: #000;
	padding: 1vh;
	color: white;
	font-size: 3vh;
	height: 29vh;
}
.blueSideText {
	color: #46F;
}
.redSideText {
	color: #F55;
}
.outcomeSide {
	display: block;
	font-size: 6vh;
}
.teamSelection {
	height:70vh;
}
</style>