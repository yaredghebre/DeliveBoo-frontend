<script >
import axios from "axios";
import { store } from "../store.js";
import SectionJumbo from "../components/SectionJumbo.vue";
import RestaurantCard from "../components/RestaurantCard.vue";
import Pagination from "../components/Pagination.vue";

export default {
  name: "RestaurantsPage",
  components: {
    SectionJumbo,
    RestaurantCard,
    Pagination
  },
  data() {
    return {
      store,
      restaurants: [],
      currentPage: 1,
      lastPage: null,
      loading: false,
      types: [],
      allRestaurants: 0,
      payment_message:this.$route.query.success

     

    };
  },
  mounted() {
    this.getRestaurants();
    this.getTypes();
  
  },
  methods: {
    getTypes() {
      axios.get(`${store.ApiTypesUrl}`).then(resp => {
        this.types = resp.data.results;
      });
    },
    getRestaurants(pageNumber) {
      this.loading = true;
      const params = {
        page: pageNumber
      }

      axios.get(`${store.ApiRestaurantsUrl}`, { params }).then((resp) => {
        this.restaurants = resp.data.results.data;
        this.currentPage = resp.data.results.current_page;
        this.lastPage = resp.data.results.last_page;
        this.allRestaurants = resp.data.results.total;
      }).finally(() => {
        this.loading = false;
      });
    }
  }
}
</script>

<template>
  <div v-if="payment_message" class="overlay">

    <div  class="payment-success text-center  p-2 ">
      <i class="fa-regular fa-circle-check fa-3x"></i>
      <h5>pagamento effettuato</h5>
      <hr>
      <p class="mb-2">riceverai un email con il riepilogo del tuo ordine</p>
      <button type="button" class="btn btn-secondary" @click="payment_message =false">Close</button>
    </div>

  </div>
  <SectionJumbo />
  <section class="restaurants">


    <div v-if="!loading">
      <div class="background-extencion"></div>
      <div class="container">
        <div class="section-title">
          <div class="img-absolute">
            <img
              src="https://res.cloudinary.com/glovoapp/f_auto,h_48,w_48,q_auto:best/CX/new_backend_driven_home_screen/newUserRecommendationsWidgetIconLight"
              alt="">
          </div>
          <h5>Tutti i locali</h5>
        </div>
        <!-- message payment success -->
  
        <!-- /message payment success -->

     
        <div class="row">
          <div v-for="restaurant in restaurants" class="ms_col">
            <RestaurantCard :restaurant="restaurant" />
          </div>
        </div>
        <Pagination :currentPage="currentPage" :lastPage="lastPage" @changePage="getRestaurants" />
      </div>
    </div>
    <div v-else class="restaurants">
      <div class="background-extencion"></div>
      <span class="text-center ms_loader"></span>
    </div>
  </section>

</template>

<style lang="scss" scoped>
@use "../styles/partials/root.scss" as *;
.overlay{
  width: 100%;
  background-color: rgba($color: #000000, $alpha: .5);
  position: absolute;
  z-index: 556;
  top: 0;
  left: 0;
  height: 100%;
  
  .payment-success{
    position: sticky;
    width: 70%;
    max-width: 800px;
    margin: 0 auto;
    top: 40%;
    border-radius: 15px;
    box-shadow: 0px 10px 20px black;
    z-index: 555;
    background-color: #E1F4E5;
    border: 1px solid #28a181;
    i{color:#28a181};
    h5{
      color: #28a181;
    }
  }
}


.restaurants {
  background-color: $secondary_color;
  padding: 1rem 0;
  position: relative;
  min-height: 700px;

  .background-extencion {

    position: absolute;
    left: 0px;
    right: 0px;
    top: -25px;
    height: 50px;
    border-radius: 100%;
    background-color: $secondary_color ;
  }

  .container {
    position: relative;
    z-index: 3;
    padding: 0;
    width: 95% !important;

    .section-title {
      color: black;
      padding-left: 2rem;

      @media screen and (max-width : 768px) {
        padding-left: 40%;
      }

      .img-absolute {
        position: relative;
      }

      img {
        position: absolute;
        z-index: 0;
        top: -15px;
        left: -50px;
        transform: rotate(25deg);
      }

      h5 {
        position: relative;
        z-index: 1;
        font-weight: bold;
      }
    }

    .row {
      
      flex-wrap: wrap;
      justify-content: center;
      padding-top: 1.5rem;

      .ms_col {

        width: 350px;
        height: 150px;
        padding: 5px 5px;
      }
    }
  }
}



@keyframes animMarker {
  0% {
    transform: rotate(45deg) translate(5px, 5px);
  }

  100% {
    transform: rotate(45deg) translate(-5px, -5px);
  }
}

@keyframes animShadow {
  0% {
    transform: scale(0.5);
  }

  100% {
    transform: scale(1);
  }
}</style>
