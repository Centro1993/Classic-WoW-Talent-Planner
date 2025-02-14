<template>
	<div class="talent-tree-panel">
		<h3 class="talent-tree-title">{{tree.name}} <span class="talent-tree-points">({{tree.skillPoints}})</span></h3>
		<div class="talent-tree" :style="getTreeBackgroundImage">
			<skill
				v-for="skill in tree.skills"
				v-bind:skill="skill"
				v-bind:key="skill.id"
				v-bind:tree="tree"
				v-bind:className="className"
				v-bind:currentSkillTier="tree.currentSkillTier"
				v-bind:availableSkillPoints="availableSkillPoints"
				v-bind:requiredLevel="requiredLevel"
				v-on:increaseTreeSkillPoints="onIncreaseTreeSkillPoints"
				v-on:decreaseTreeSkillPoints="onDecreaseTreeSkillPoints"
				v-on:increaseCurrentSkillTier="onIncreaseCurrentSkillTier"
				v-on:decreaseCurrentSkillTier="onDecreaseCurrentSkillTier">
			</skill>
		</div>
		<span class="talent-tree-reset" v-on:click="resetTalentTree">Reset</span>
	</div>
</template>
<script>
	import { config } from './Config.js';
	import skill from './Skill';

	export default {
		name: 'talent-tree',
		props: {
			className: String,
			tree: Object,
			availableSkillPoints: Number,
			requiredLevel: Number,
		},
		components: {
			skill
		},
		computed: {
			getTreeBackgroundImage: function(){
				let backgroundImageUrl = 	config.imageDirectory + 
											config.backgroundDirectory + 'background-' + 
											this.className.toLowerCase() + '-' + 
											this.getImageFileName(this.tree.name);
				return {
					backgroundImage: `url('${backgroundImageUrl}')`,
					backgroundRepeat: 'no-repeat',
					backgroundSize: 'cover',
					backgroundPosition: 'center',
				}
			},
		},
		methods: {
			onIncreaseTreeSkillPoints: function(){
				this.tree.skillPoints++;
			},
			onDecreaseTreeSkillPoints: function(){
				this.tree.skillPoints--;
			},
			onIncreaseCurrentSkillTier: function(tier){
				if(tier > this.tree.currentSkillTier)
					this.tree.currentSkillTier = tier;
			},
			onDecreaseCurrentSkillTier: function(tier){
				let totalTierSkillPoints = this.getTotalTierSkillPoints(tier);
				if(totalTierSkillPoints == 0)
					this.tree.currentSkillTier = tier - 1;
			},
			getTotalTierSkillPoints: function(tier){
				let tierSkillPoints = 0;
				this.tree.skills.forEach((skill) => {
					if(skill.position[0] == tier){
						tierSkillPoints = tierSkillPoints + skill.currentRank;
					}
				});
				return tierSkillPoints;
			},
			resetTalentTree: function(){
				if(this.tree.skillPoints > 0){
					this.$emit('increaseAvailableSkillPoints', this.tree.skillPoints);
					this.$emit('decreaseRequiredLevel', this.tree.skillPoints);
					this.tree.skillPoints = 0;
					this.tree.currentSkillTier = 0;
					this.tree.skills.forEach((skill) => {
						skill.currentRank = 0;
						if(skill.requirements){
							skill.enabled = false;
						}
						this.$emit('removeTreeFromTalentPath', this.tree.id);
					});
				}
			}
		}
	}
</script>