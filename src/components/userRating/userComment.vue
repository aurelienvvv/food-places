<template>
	<form class="user-comment">
		<div class="input-wrapper">
			<input type="text" v-model="userName" placeholder="Nom">
		</div>
		<div class="input-wrapper">
			<input type="text" v-model="userComment" placeholder="Commentaire">
		</div>
		<div class="input-wrapper">
			<input type="text-area" v-model="userRating" placeholder="Note pour ce restaurant">
		</div>
		<button type="submit" @click="addRatingRestaurant">send</button>
	</form>
</template>

<script>
export default {
	props: {
		addRatingRestaurant: {
			type: Function
		},
		userRating: {
			type: Number
		},
		userComment: {
			type: String
		},
		userName: {
			type: String
		}
	}
}
</script>

<style>

</style>