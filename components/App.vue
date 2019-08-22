<template>
	<div>
		<main>
			<img src="../public/images/wow-classic-logo.png" class="logo" />
			<h1 class="main-title">Talent Planner</h1>
			<ul class="class-list">
				<class-list
					v-for="classType in data.classes"
					v-bind:classType="classType"
					v-bind:key="classType.id"
					v-bind:currentClass="data.currentClass"
					v-on:change-class="data.currentClass = classType.id"
				></class-list>
			</ul>
			<div class="talent-toolbar">
				<div class="talent-info">
					<p class="talent-info-stat">Skill points: {{data.classes[data.currentClass].availableSkillPoints}}</p>
					<p class="talent-info-stat">Required level: {{data.classes[data.currentClass].requiredLevel}}</p>
				</div>
			</div>
			<class-panel
				v-bind:class-type="data.classes[data.currentClass]"
			></class-panel>
			<talent-path
				v-bind:currentClass="data.classes[data.currentClass]"
			></talent-path>
			<div class="talent-actions">
				<button class="button" v-on:click="saveTalentTrees">Save</button>
			</div>
			<div>
				<a :href="saveLink">{{saveLink}}</a>
			</div>
		</main>
		<footer>
			<ul>
				<li><a href="https://github.com/hseager/Classic-WoW-Talent-Planner" target="_blank"><img src="images/github-logo.png" /></a></li>
			</ul>
		</footer>
	</div>
</template>
<script>
	import talentData from '../assets/data/talent-data.json';
	import classList from './ClassList';
	import classPanel from './ClassPanel';
	import talentPath from './TalentPath';

	export default {
		components: {
			classList,
			classPanel,
			talentPath
		},
		data(){
			return {
				data: talentData
			}
		},
		computed: {
			saveLink() {
				const currentClass = this.data.classes[this.data.currentClass];

				let urlAppendix = `?class=${this.data.currentClass}&skilltree=`;
				for(const skilltree of currentClass.talentTrees) {
					for(const talent of skilltree.skills) {
						urlAppendix += `${talent.currentRank},`
					}
				}
				// remove last seperation comma
				urlAppendix = urlAppendix.slice(0, urlAppendix.length-1)

				return `${window.location.href.split('?')[0]}${urlAppendix}`;
			}
		},
		mounted(){
			const copyOfData = this.deserializeTalentTreeFromUrl();
			if (copyOfData) {
				this.data = copyOfData;
				return
			}
			let localData = window.localStorage.getItem('talent-data-1');
			if(localData != null){
				this.data = JSON.parse(localData);
			}
		},
		methods: {
			getUrlParam: function(parameter){
				let url = new URL(window.location);
				return url.searchParams.get(parameter);
			},
			saveTalentTrees: function(){
				window.localStorage.setItem('talent-data-1', JSON.stringify(this.data));
			},
			deserializeTalentTreeFromUrl: function() {
				let dataCopy = this.data;

				dataCopy.currentClass = parseInt(this.getUrlParam('class'));
				if (!dataCopy.currentClass) return false;

				const skilltreeString = this.getUrlParam('skilltree');
				if(!skilltreeString) return false;

				let skilltreeRanks = skilltreeString.split(',');

				for(const skilltreeKey in dataCopy.classes[dataCopy.currentClass].talentTrees) {
					for(const talentKey in dataCopy.classes[dataCopy.currentClass].talentTrees[skilltreeKey].skills) {
						console.log(dataCopy.classes[dataCopy.currentClass].talentTrees[skilltreeKey].skills[talentKey].currentRank)
						dataCopy.classes[dataCopy.currentClass].talentTrees[skilltreeKey].skills[talentKey].currentRank = parseInt(skilltreeRanks[0]);
						skilltreeRanks = skilltreeRanks.slice(1, skilltreeRanks.length);
					}
				}

				return dataCopy;
			}
		}
	}
</script>