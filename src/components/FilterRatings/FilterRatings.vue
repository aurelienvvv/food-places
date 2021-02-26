<template>
	<div class="filter-restaurants">
		<div class="title">Filtrez les restaurants</div>
		<div class="wrapper">
			<div class="input-wrapper" v-for="starNumb in 5" :key="starNumb">
				<input @click='changeRankRating' type="radio" :id="idName + starNumb" name="stars" :value="starNumb">
				<label :for="idName + starNumb">
					<div class="stars-wrapper">
						<img src="../../assets/icons/star-icon.svg" alt="star">
					</div>
				</label>
			</div>
		</div>
		<button class="btn -sm -hover-gradient" @click="ratingChooseReset">Reset</button>
	</div>
</template>

<script>
export default {
	props: ['ratingChooseReset', 'changeRankRating', 'idName']
}
</script>

<style lang='scss'>
	@import "../../assets/scss/elements";
	@import "./FilterRatings";
</style>