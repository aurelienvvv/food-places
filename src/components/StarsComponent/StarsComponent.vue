<template>
	<div class="stars">
		<div class="stars-grey">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
		</div>
		<div class="rating-stars" :style="rating">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
			<img src="../../assets/icons/star-icon.svg" alt="star">
		</div>
	</div>
</template>

<script>
export default {
	props: {
		rating: {
			type: String,
		}
	}
}
</script>

<style lang='scss'>
	@import "../../assets/scss/vars";
	@import "./StarsComponent";
</style>