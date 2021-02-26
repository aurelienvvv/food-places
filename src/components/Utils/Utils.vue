<script>
import $ from 'jquery';

export default {
	methods: {
		randomChooseArray: function(array) {
		return array[Math.floor(Math.random() * Math.floor(array.length))];
		},

		// Methode pour l'animation des popins au click
		animationPopin: function(popin, overlay, nameVar) {
			if (nameVar === false) {
				$(overlay).hide();
				$(popin).removeClass('-active');
				setTimeout(() => {
				$(popin).hide();
				}, 400);

			} else {
				$(overlay).show();
				$(popin).show();
				setTimeout(() => {
					$(popin).addClass('-active');
				}, 100);
			}
		},

		// Methodes de calcule des notes commentaires
		average: function(ratings) {
			let res = 0

			for (let rating of ratings) {
				res += rating.stars
			}

			return res / ratings.length;
		},
	}
}
</script>