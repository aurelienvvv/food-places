<template>
  <div>
    <div id="map"></div>

    <!-- Filtres des restaurants sur la map -->
    <filter-ratings
      :ratingChooseReset=ratingChooseReset
      :changeRankRating=starComponentOnClick
      idName="map-rating-"
    />

    <!-- Popin formulaire d'ajout d'un restaurant -->
    <form class="popin map">
      <div @click="clickOnMap = false; clickOnMapLatLong = []" class="close-btn -white -right"></div>
      <div class="wrapper-title">
        <div class="title">Ajoutez un restaurant</div>
      </div>
      <div class="input-wrapper-form">
        <input type="text" v-model="newRestaurantName" placeholder="Nom du restaurant">
      </div>
      <div class="btn-wrapper">
        <button class="btn -gradient" @click="addRestaurantForm">Ajouter</button>
      </div>
    </form>
    <div class="overlay-popin map"></div>


    <!-- Conteneur des infos à gauche de la carte -->
    <div class="container-infos">

      <!-- Liste des restaurants -->
      <ul class="list-restaurants">
        <li 
          v-for="(restaurant, index) in restaurants" 
          :key="index" 
          :class="`info-${index + 1}`"
          @click='restaurantDetail(restaurant, restaurant.lat, restaurant.long)'
        >
            <div class="img-wrapper">
              <img :src="streetView + restaurant.lat + ',' + restaurant.long + '&heading=151.78&pitch=-0.76&key=' + apiKey" alt="image street view">
            </div>
            <div class="infos-wrapper">
              <div class="name">{{ restaurant.name }}</div>
              <div class="rating-wrapper">
                <div v-if="restaurant.ratings" class="average">{{ restaurant.ratings }}</div>
                <stars-component :rating=ratingStarsPercent(restaurant.ratings) />
              </div>
              <div class="address">{{ restaurant.address }}</div>
            </div>
        </li>
      </ul>

      <!-- Détails d'un restaurant -->
      <div class="restaurant-details">
        <button class="close-btn -left" @click="closeDetails"></button>
        <div v-for="(details, index) in currentRestaurant" :key="index">
          <div class="img-wrapper">
            <img :src="streetView + details.lat + ',' + details.long + '&heading=151.78&pitch=-0.76&key=' + apiKey" alt="image street view">
          </div>

          <div class="container-text">
            <div class="name">{{ details.name }}</div>
              <div class="rating-wrapper -details">
                <div v-if="details.ratings" class="average">{{ details.ratings }}</div>
                <stars-component :rating=ratingStarsPercent(details.ratings) />
              </div>

            <div class="infos-wrapper">
              <div class="address-wrapper">
                <img src="../../assets/icons/pin-icon.svg" alt="pin">
                <div class="address">{{ details.address }}</div>
              </div>
            </div>

            <div class="btn-wrapper">
              <button type="button" @click="openUserRating = true" class="btn -gradient">Ajouter un commentaire</button>
            </div>

            <!-- Formulaire popin d'ajout un commentaire -->
            <form @submit.prevent="addRatingRestaurant" class="popin user-rating">
              <div class="wrapper-title">
                <div class="title-rating-user">{{ details.name }}</div>
                <div class="subtitle-rating-user">Ajoutez votre avis : </div>
              </div>
              <div class="close-btn -rating" @click='openUserRating = false'></div>
              <div class="input-wrapper-form">
                <input type="text" v-model="userName" placeholder="Nom">
              </div>
              <div class="input-wrapper-form">
                <textarea type="textarea" v-model="userComment" placeholder="Commentaire" />
              </div>
              <filter-ratings
                :changeRankRating=starComponentOnClick
                idName="comment-rating-"
                class=comment-rating-stars
              />
              <div class="btn-wrapper">
                <button type="submit" class="btn -gradient">Ajouter</button>
              </div>
            </form>

            <div class="overlay-popin rating"></div>

            <div class="comments-title">Avis</div>

            <ul class="comments">
              <li v-for="(review, index) in details.reviews" :key="index">
                <div class="author-infos">
                  <div v-if="review.profile_photo_url" class="img-wrapper">
                    <img :src="review.profile_photo_url" alt="photo"/>
                  </div>
                  <div v-if="review.alien_pic" class="img-wrapper -circle">
                    <img :src="review.alien_pic" alt="photo"/>
                  </div>
                  <div v-if="review.author_name" class="name">{{ review.author_name }}</div>
                </div>
                <div class="rating-wrapper -comment">
                  <div class="average">{{ review.rating }}</div>
                  <stars-component :rating=ratingStarsPercent(review.rating) />
                </div>
                <div class="date">{{ review.relative_time_description }}</div>
                <div class="comment">{{ review.text }}</div>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import gmapsInit from '../../gmaps';
import datasJSON from '../../datas.json';
import $ from 'jquery';
import StarsComponent from '../StarsComponent/StarsComponent';
import FilterRatings from '../FilterRatings/FilterRatings';
import UTILS from '../Utils/Utils';

// variable global qui stock les markers
let markers = [];

export default {
  name: 'Map',

  components: {
    StarsComponent,
    FilterRatings,
  },

  data(){
    return {
      userLat: 48.8534,
      userLong: 2.3488,
      markers: [],
      currentRestaurant: [],
      restaurants: [],
      streetView: 'https://maps.googleapis.com/maps/api/streetview?size=600x300&location=',
      apiKey: process.env.VUE_APP_MAPS_API_KEY,
      datasJSON: datasJSON,
      searchOnMove: true,
      openUserRating: false,
      userComment: '',
      userName: '',
      newRestaurantName: '',
      userRating: '',
      addressNewRestaurant: '',
      usersReviews: [],
      clickOnMapLatLong: [],
      clickOnMap: false,
      newRestaurants: [],
      newRatingsRestaurants: [],
      ratingChoose: null,
      alienPics: [
        'https://media0.giphy.com/media/26hisVHpbBwfcfKus/giphy.gif', 
        'https://media3.giphy.com/media/l2JJwSHqR1t11I3Qc/giphy.gif',
        'https://i.giphy.com/media/26tP9lirArFJq5i0w/giphy.webp',
      ],
      userRestaurantId: 13081993,
      stylesMaps: [
    {
        "featureType": "all",
        "elementType": "labels.text.fill",
        "stylers": [
            {
                "saturation": 36
            },
            {
                "color": "#000000"
            },
            {
                "lightness": 40
            }
        ]
    },
    {
        "featureType": "all",
        "elementType": "labels.text.stroke",
        "stylers": [
            {
                "visibility": "on"
            },
            {
                "color": "#000000"
            },
            {
                "lightness": 16
            }
        ]
    },
    {
        "featureType": "all",
        "elementType": "labels.icon",
        "stylers": [
            {
                "visibility": "off"
            }
        ]
    },
    {
        "featureType": "administrative",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 20
            }
        ]
    },
    {
        "featureType": "administrative",
        "elementType": "geometry.stroke",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 17
            },
            {
                "weight": 1.2
            }
        ]
    },
    {
        "featureType": "administrative.locality",
        "elementType": "labels.text",
        "stylers": [
            {
                "hue": "#ff9400"
            },
            {
                "visibility": "on"
            },
            {
                "weight": "1.19"
            }
        ]
    },
    {
        "featureType": "landscape",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 20
            }
        ]
    },
    {
        "featureType": "poi",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 21
            }
        ]
    },
    {
        "featureType": "road.highway",
        "elementType": "geometry.fill",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 17
            }
        ]
    },
    {
        "featureType": "road.highway",
        "elementType": "geometry.stroke",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 29
            },
            {
                "weight": 0.2
            }
        ]
    },
    {
        "featureType": "road.arterial",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 18
            }
        ]
    },
    {
        "featureType": "road.local",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 16
            }
        ]
    },
    {
        "featureType": "transit",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#000000"
            },
            {
                "lightness": 19
            }
        ]
    },
    {
        "featureType": "water",
        "elementType": "geometry",
        "stylers": [
            {
                "color": "#1f3943"
            },
            {
                "lightness": 17
            }
        ]
    }
]
    }
  },

  methods: {
    initMap: async function() {
      try {
        // Init gmaps
        this.google = await gmapsInit();

        // Appel geocoder
        this.geocoder = new this.google.maps.Geocoder();

        // Creation de la map dans le dom
        this.map = new this.google.maps.Map(document.getElementById("map"), {
          center: {lat: this.userLat , lng: this.userLong},
          zoom: 16,
          styles: this.stylesMaps,
          disableDefaultUI: true,
        });

        // Créé un marker personnalisé sur la position de l'utilisateur
        this.createUserMarker();

        // Au click sur la map
        this.google.maps.event.addListener(this.map, 'click', (event) => {
          // On stocke les coordonées dans un tableau
          this.clickOnMapLatLong.push(event.latLng);

          // la popin avec le formulaire d'ajout d'un nouveau restaurant s'affiche
          this.clickOnMap = true; 
        });

        // Appel de l'API places
        this.service = new this.google.maps.places.PlacesService(this.map);

        // La méthode de recherche des restaurants se lance
        this.searchNearby();

        // Au déplacement de l'utilisateur sur la map
        this.google.maps.event.addListener(this.map, 'idle', () => {

            // Si searchOnMove est à true, donc si aucun restaurant n'est séléctionné
            if (this.searchOnMove) {

              // La méthode de recherche des restaurants se lance
              this.searchNearby();
            }
        });

        } catch (error) {
          console.error(error);
        }
    },

    // méthodes liées à l'API places: trouver + stocker des adresses
    searchNearby: function() {
      // Ajout des paramètres de recherches de l'API Places
        let requestPlaces = {
          location: new this.google.maps.LatLng(this.map.getCenter().lat(),this.map.getCenter().lng()),
          radius: '800',
          type: ['restaurant']
        };

        // Résultats de la recherche sont envoyés dans la méthode callbackPlaces
        this.service.nearbySearch(requestPlaces, this.callbackPlaces);
    },

    // Stock + affiche restaurants
    callbackPlaces: function(results, status) {
      // Si il y a des resultats pour la recherche Places
      if (status == this.google.maps.places.PlacesServiceStatus.OK) {
        // Vide le tableau contenant les restaurants
        this.restaurants = [];

        // Ajout des restaurants créer pas l'utilisateur au tableau des restaurants
        this.newRestaurants.forEach((rest) => this.restaurants.push(rest));

        // Ajout des restaurants récupérés du fichier JSON
        this.getJSONrestaurants(datasJSON);

        // Ajout des restaurants trouvés par l'API Places
        for (let i = 0; i < results.length; i++) {
          let place = results[i];
          this.restaurants.push({
            id: place.place_id,
            name : place.name,
            address : place.vicinity,
            lat : place.geometry.location.lat(),
            long : place.geometry.location.lng(),
            ratings : place.rating,
            type: 'placesAPI'
          })
        }

        // Met à jour les notes des restaurants par rapports aux notes ajoutés par l'utilisateur
        this.newAverageAfterComments();

        // Si le filtre des restaurants est activé, seul les restaurants correspondants au filtre sont affichés
        if (this.ratingChoose) {
          this.restaurants = this.restaurants.filter(restaurant => restaurant.ratings >= this.ratingChoose);
        }

        // Retrait de tous les markers correspondants aux anciens résultats
        this.removeAllMarkers();

        // Ajout des markers correspondants à la liste des restaurants
        this.addMarkers(this.google, this.map);

        // Affiche dans la liste, les restaurants au marker visible à l'utilisateur 
        this.showVisibleMarkers(this.map);
      }
    },

    // Récupération du nombre d'étoiles séléctionné au click sur le composant filterRatings
    starComponentOnClick: function(e) {
      let $elem = $(e.currentTarget);
      let $inputWrapper = $elem.closest('.input-wrapper');
      let $parent = $inputWrapper.closest('.filter-restaurants');

      if ($parent.hasClass('comment-rating-stars')){
        this.userRating = parseInt($elem.val());
      } else {
        this.ratingChoose = parseInt($elem.val());
      }

      $parent.find('.input-wrapper').removeClass('active');
      $inputWrapper.addClass('active');
      $inputWrapper.prevAll().addClass('active');
    },

    ratingStarsPercent: function(rating) {
			if (rating) {
				return `width: ${(rating / 5) * 100}%`
			} else {
				return `width: 0%`
			}
		},

    // Au click sur le bouton reset du composant filterRatings
    ratingChooseReset: function() {
      $('.input-wrapper').removeClass('active');
      this.ratingChoose = null;
    },

    // Méthodes liées aux markers présents sur la map
    addMarkers: function(google, map) {
      markers = [];

      // styles de markeurs
      this.iconNormal = {
        url: "https://i.postimg.cc/N08pDYTr/marker-3.png", // url
        scaledSize: new google.maps.Size(45, 45), // scaled size
        origin: new google.maps.Point(0,0), // origin
        anchor: new google.maps.Point(0, 0), // anchor
        name: 'normal'
      };

      this.iconActive = {
        url: "https://i.postimg.cc/QdRfpDNc/marker-4.png", // url
        scaledSize: new google.maps.Size(45, 45), // scaled size
        origin: new google.maps.Point(0,0), // origin
        anchor: new google.maps.Point(0, 0) // anchor
      };

      this.markerCustomUser = {
        url: "https://i.postimg.cc/ZYM2pvjH/marker.png", // url
        scaledSize: new google.maps.Size(45, 45), // scaled size
        origin: new google.maps.Point(0,0), // origin
        anchor: new google.maps.Point(0, 0), // anchor
        name: 'new'
      };

      this.markerCustomUserActive = {
        url: "https://i.postimg.cc/nLSg2spn/marker-2.png", // url
        scaledSize: new google.maps.Size(45, 45), // scaled size
        origin: new google.maps.Point(0,0), // origin
        anchor: new google.maps.Point(0, 0), // anchor
        name: 'new'
      };

      // Ajout des markers
      this.restaurants.forEach(el => {
        let nameIcon = this.iconNormal;
        el.type === "newRestaurant" ? nameIcon = this.markerCustomUser : nameIcon = this.iconNormal;
        
        let marker = new google.maps.Marker({
          position: new this.google.maps.LatLng(el.lat, el.long),
          map,
          id: el.id,
          icon: nameIcon,
          type: nameIcon.name,
          animation: this.google.maps.Animation.DROP,
        });

        markers.push(marker);
        this.markerOnClick(marker);
      });

    },

    removeAllMarkers: function() {
      markers.forEach((marker) => marker.setMap(null));
      markers = [];
    },

    showVisibleMarkers: function(map) {
      // les restaurants des marqueurs visible sur la carte sont ajoutés à gauche
      let bounds = map.getBounds();
      let count = 0;

      for(let i = 0; i < markers.length; i++) {
        let marker = markers[i];
        let infoPanel = $('.info-' + (i + 1));

        if (bounds.contains(marker.getPosition()) === true) {
          infoPanel.show();
          count++;
        }
        else {
          infoPanel.hide();
        }

        count;
      }
    },

    allMarkersNormal: function() {
      markers.forEach((marker) => {
        marker.type === 'new' ? marker.setIcon(this.markerCustomUser) : marker.setIcon(this.iconNormal);
      })
    },

    // Au click sur un marker
    markerOnClick: function(marker) {
      // Selection du marker
      this.google.maps.event.addListener(marker, 'click', () => {

        // Animation du marker séléctionné
        this.bounceMarker(marker);

        // Zoom sur le marker et change son style
        this.zoomOnMarker(marker);
        let markerLat = marker.getPosition().lat();
        let markerLong = marker.getPosition().lng();

        // Récupère le restaurant avec lat long
        let getRes = this.restaurants.filter(rest => rest.lat === markerLat && rest.long === markerLong)[0];

        // Met à jour le restaurant séléctionné avec les données du marker
        this.currentRestaurant = [];
        this.restaurantDetail(getRes, markerLat, markerLong);
      })
    },

    bounceMarker: function(marker) {
      markers.forEach(marker => marker.setAnimation(null));
      if (marker.getAnimation() !== null) {
        marker.setAnimation(null);
      } else {
        marker.setAnimation(this.google.maps.Animation.BOUNCE);
      }
    },

    zoomOnMarker: function(marker) {
      this.bounceMarker(marker);
      this.allMarkersNormal();
      marker.type === 'new' ? marker.setIcon(this.markerCustomUserActive) : marker.setIcon(this.iconActive);
      this.searchOnMove = false;

      this.map.setZoom(17);
      this.map.setCenter(marker.getPosition());
    },


    // Méthodes liées à la création d'un nouveau restaurant par l'utilisateur

    // Permet de trouver l'adresse sur le lieu cliqué par l'utilisateur
    geocodeConvertLatLng: function(lat, lng) {
      const latlng = {
        lat: lat,
        lng: lng,
      };

      // Geocoder trouve l'adresse avec les coordonnées de l'addresse
      this.geocoder.geocode({ location: latlng }, (results, status) => {
        if (status === "OK") {
          if (results[0]) {
            // Le resultat est stocké
            let address = results[0].formatted_address;
            this.addressNewRestaurant = address;

            // Puis ajouté aux autres restaurants
            this.addRestaurantUser();
          } else {
            window.alert("No results found");
          }
        } else {
          window.alert("Geocoder failed due to: " + status);
        }
      });
    },

    addRestaurantForm: function(e) {
      e.preventDefault();

      this.geocodeConvertLatLng(this.clickOnMapLatLong[0].lat(),this.clickOnMapLatLong[0].lng());
    },

    addRestaurantUser: function() {
      if (this.newRestaurantName) {

        // ajout du nouveau restaurant au tableau des restaurant
        this.newRestaurants.push({
          id: this.userRestaurantId + 1,
          name : this.newRestaurantName,
          ratings: undefined,
          address : this.addressNewRestaurant,
          lat : this.clickOnMapLatLong[0].lat(),
          long : this.clickOnMapLatLong[0].lng(),
          type: 'newRestaurant'
        });

        this.newRestaurantName = '';

        // Relance les methodes de recherche des restaurants
        this.searchNearby();
        this.clickOnMap = false;
        this.clickOnMapLatLong = [];

        // Ferme la fenetre des détails d'un restaurant si elle est ouverte
        this.closeDetails();
      }
    },


    // Methodes détails d'un restaurant
    restaurantDetail: function(restaurant, lat, lng) {   
      // Si le restaurant vient de l'api Places, on récupère les données de l'api
      // puis on stocke dans un tableau qu'on affiche à l'utilisateur
      if (restaurant.type === 'placesAPI') {
        let getMarker;
        let request = {
          placeId: restaurant.id,
          fields: ['name', 'rating', 'formatted_address', 'reviews']
        };
        

        this.service.getDetails(request, (place, status) => {
          if (status == this.google.maps.places.PlacesServiceStatus.OK) {
            this.currentRestaurant.push({
              id: restaurant.id,
              name : place.name,
              address : place.formatted_address,
              lat : lat,
              long : lng,
              ratings : place.rating,
              reviews: place.reviews,
            });

            document.querySelector('.list-restaurants').style.display = 'none';
            document.querySelector('.restaurant-details').style.display = 'block';

            getMarker = markers.filter(marker => marker.id === restaurant.id)[0];
            this.google.maps.event.trigger(getMarker, this.zoomOnMarker(getMarker));

            getMarker = null;

            this.addReviewsToRestaurant();
          }
        })
      } 
      // Sinon on prend les données déjà présentes du restaurant séléctionné
      // qu'on stockent dans un tableau qu'on affiche à l'utilisateur
      else {
        let getMarker;
        document.querySelector('.list-restaurants').style.display = 'none';
        document.querySelector('.restaurant-details').style.display = 'block';
        this.currentRestaurant.push(restaurant);

        getMarker = markers.filter(marker => marker.id === restaurant.id)[0];

        console.log(getMarker)
        this.google.maps.event.trigger(getMarker, this.zoomOnMarker(getMarker));

        getMarker = null;
      }

    },

    closeDetails: function() {
      this.searchOnMove = true;
      this.allMarkersNormal();
      markers.forEach(marker => marker.setAnimation(null));

      document.querySelector('.restaurant-details').style.display = 'none';
      document.querySelector('.list-restaurants').style.display = 'block';
      this.currentRestaurant = [];
    },


    // Methodes liées à la position de l'utilisateur
    findPositionUser: function() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          this.userLat = position.coords.latitude;
          this.userLong = position.coords.longitude;
        });
      } else {
        alert("La géolocalisation n'est pas supportée sur votre navigateur :'(");
      }
    },

    createUserMarker: function() {
      this.iconUser = {
        url: "https://media4.giphy.com/media/xT9IgBs921QcFeC0ms/giphy.gif", // url
        scaledSize: new this.google.maps.Size(60, 60), // scaled size
        origin: new this.google.maps.Point(0,0), // origin
        anchor: new this.google.maps.Point(0, 0) // anchor
      };

      new this.google.maps.Marker({
        position: new this.google.maps.LatLng(this.userLat, this.userLong),
        map: this.map,
        id: 'userMarker',
        icon: this.iconUser,
      });
    },

    // Methode d'ajout d'un avis à un restaurant
    addRatingRestaurant: function(e) {
      e.preventDefault();

      // Si reviews est undefined, on transform reviews en tableau
      if (!this.currentRestaurant[0].reviews) {
        this.currentRestaurant[0].reviews = [];
      }

      // Ajout du commentaire dans le tableau des commentaires de l'utilisateur
      if (this.userRating && this.userName && this.userComment) {
        this.usersReviews.push({
          restaurant_id: this.currentRestaurant[0].id,
          author_name: this.userName,
          alien_pic: UTILS.methods.randomChooseArray(this.alienPics),
          rating: this.userRating,
          text: this.userComment,
        });

        // Ajout du seul dernier commentaire ajouté
        this.addReviewsToRestaurant('addOne');

        // Après l'ajout de l'avis, calcule la moyenne des notes du restaurant
        this.updateRestaurantRating(this.currentRestaurant);

        this.userRating = '';
        this.userComment = '';
        this.userName = '';
        this.openUserRating = false;
        $('.overlay-popin').hide();
      }
    },

    // Ajout des avis de l'utilisateur sur les restaurants
    addReviewsToRestaurant: function(addOneOrMany) {
      let filterReviews = this.usersReviews.filter(review => review.restaurant_id === this.currentRestaurant[0].id);
      let lastReview = filterReviews[filterReviews.length - 1];

      if (!this.currentRestaurant[0].reviews) {
        this.currentRestaurant[0].reviews = [];
      }
      
      // Si l'avis vient d'etre ajouté, on pousse seulement ce dernier
      if (addOneOrMany === ('addOne')) {
        this.currentRestaurant[0].reviews.unshift({
          author_name: lastReview.author_name,
          alien_pic: lastReview.alien_pic,
          rating: lastReview.rating,
          text: lastReview.text,
        });

      // Sinon, on pousse tous les avis déjà ajoutés
      } else {
        filterReviews.forEach((review) => {
          this.currentRestaurant[0].reviews.unshift({
            author_name: review.author_name,
            alien_pic: review.alien_pic,
            rating: review.rating,
            text: review.text,
          });
        })
      }

      // Si il y a des avis ajouté par l'utilisateur, on recalcule la moyenne des notes du restaurant
      if (filterReviews.length) {
        this.updateRestaurantRating(this.currentRestaurant);
      }
    },

    // Calcule la moyenne des avis sur le restaurant
    updateRestaurantRating: function(restaurant) {
      let reviewsRestaurant = restaurant[0].reviews;

      // On récupère les moyennes avec le map et on additionne le tout avec reduce
      let reviewsArray = reviewsRestaurant.map(review => review.rating);
      let addition = reviewsArray.reduce((a, b) => a + b);
      let newAverage = (addition / reviewsRestaurant.length).toFixed(1);

      this.currentRestaurant[0].ratings = newAverage;

      // On stocke les restaurants qui ont un avis ajoutés dans un tableau
      this.newRatingsRestaurants.push({
        id: restaurant[0].id,
        newAverage: newAverage,
      });

      this.newAverageAfterComments();
    },

    newAverageAfterComments: function() {
      // met à jour le tableau des restaurants avec les nouvelles notes attribuées
      this.newRatingsRestaurants.forEach((restaurant) => {
        let r = this.restaurants.filter(rest => rest.id === restaurant.id)[0];
        r.ratings = restaurant.newAverage;
      })
    },

    // Récupération des données du json fourni
    getJSONrestaurants:function(datas) {
      datas.forEach((data, i) => {
        // Calcule de la moyenne qui n'est pas fournie
        let averageRatings = UTILS.methods.average(data.ratings);

        // Données ajoutées au tableau des restaurants
        let formatedRatings = [];
        
        Array.from(data.ratings).forEach((rating) => {
          formatedRatings.push({
            text: rating.comment,
            rating: rating.stars,
          })
        });

        this.restaurants.push({
          id: i + 1,
          name : data.restaurantName,
          address : data.address,
          lat : data.lat,
          long : data.long,
          ratings : averageRatings,
          reviews: formatedRatings,
          type: 'localJON'
        })
      });
    },
  },

  watch: {
    // Relance la carte quand l'utilisateur est localisé
    userLat: function() {
      this.initMap();
    },

    // Lancement de l'animation popin au click pour l'ajout d'un commentaire
    openUserRating: function() {
      $('.comment-rating-stars .input-wrapper').removeClass('active');
      UTILS.methods.animationPopin('.popin.user-rating', '.overlay-popin.rating', this.openUserRating);
    },

    // Lancement de l'animation popin au click sur la map
    clickOnMap: function() {
      UTILS.methods.animationPopin('.popin.map', '.overlay-popin.map', this.clickOnMap);
    },

    // Relance la recherche des restaurant quand l'utilisateur utilise le filtre
    ratingChoose: function() {
      this.searchNearby();
    }
  },

  mounted() {
    this.getJSONrestaurants(this.datasJSON);
    this.findPositionUser();
    this.initMap();
  },
};
</script>

<style lang='scss'>
  @import "../../assets/scss/elements";
  @import "./Map";
</style>