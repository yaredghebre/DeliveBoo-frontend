<script>
import axios from "axios";
import { store } from "../store";
import SectionJumbo from "../components/SectionJumbo.vue"
import AppCart from "../components/AppCart.vue";

export default {
    name: "RestaurantDetail",

    data() {
        return {
            store,
            products: [],
            restaurant: [],
            cartProducts: [],
            not_allowed: false,
        }
    },
    mounted() {
        const id = this.$route.params.id;
        this.getRestaurantDetails(id);
        this.cartProducts = this.store.method.getArray();


    },
    methods: {
        getProducts(restaurantId) {
            axios.get(`${this.store.ApiProductsUrl}`, { params: { restaurant_id: restaurantId } }).then((resp) => {
                this.products = resp.data.results;

            })
        },
        getRestaurantDetails(restaurantId) {
            axios.get(`${this.store.ApiRestaurantUrl}`, { params: { restaurant_id: restaurantId } }).then((resp) => {
                this.restaurant = resp.data.results[0];
                this.getProducts(restaurantId);
                this.not_allowed = false;
            })
        },

        // funzione che crea oggetto e lo aggiunge all array e salva nel localstorage
        newObj(object) {
            const obj = {
                id: object.id,
                name: object.name,
                quantity: 0,
                price: object.price,
                restaurant: object.restaurant_id
            }
            //  vedi se contiene gia oggetto
            if (this.cartProducts.some(item => item.id === object.id)) {

                const oggetto = this.cartProducts.find(item => item.id === object.id);
                oggetto.quantity++
            } else {
                // se è un altro ristorante non puoi
                if (this.cartProducts.some(item => item.restaurant !== object.restaurant_id)) {
                    console.log(this.cartProducts[0].restaurant);
                    this.not_allowed = true;
                    //  senno pusha oggetto
                } else {
                    this.cartProducts.push(obj);
                    obj.quantity = 1;
                    console.log(obj);
                    this.not_allowed = false;
                }
            }
            // salvo array aaggiornato nel local storage
            this.store.method.salva(this.cartProducts);
        },
        removeCart() {
            this.store.method.delete();
            this.cartProducts = [];
        },
    },
    components: {
        SectionJumbo,
        AppCart
    },

}
</script>

<template>
    <SectionJumbo :restaurant="restaurant" />
    <section class="restaurants">

        <div class="container">
            <div class="restaurant-details-col">
                <div class="restaurant-detail-card-container">
                    <div class="restaurant-detail-card ">
                        <h2 class="d-inline-block me-2">{{ restaurant.name }}</h2>
                        <!-- Altrimenti, puoi mostrare un messaggio di caricamento o una stringa predefinita -->
                        <p><i style="color:#28A182" class="fa-solid fa-location-dot"></i> {{ restaurant.address }}</p>
                    </div>
                </div>
                <div class="restaurant-products">

                    <div class="product-list">
                        <!-- CREAZIONE COMPONENTE? -->
                        <div v-for="product, index in products" class="product-card mb-3">
                            <section class="card-body" v-if="product.visible">

                                <div class="product-img">
                                    <img :src="store.method.getImgPath(product.image)" alt="">
                                </div>
                                <div class="product-content d-flex">
                                    <div class="product-description">
                                        <h6>{{ product.name }}</h6>
                                        <p v-if="product.description" class="product-detail">{{ product.description }}</p>
                                        <p v-else class="product-detail">nessuna descrizione</p>
                                    </div>
                                    <div class="product-price d-flex align-items-center justify-content-between">
                                        <div>
                                            <span>{{ product.price }}</span><span>€</span>
                                        </div>
                                        <div>
                                            <span @click="newObj(product, index)" class="buy">Aggiungi al <i class="fa-solid fa-cart-shopping" style="color: #000000;"></i></span>
                                        </div>

                                    </div>
                                </div>
                            </section>
                        </div>
                    </div>
                </div>
            </div>


            
            
            <AppCart class="desktop" :cartProducts="cartProducts" :not_allowed="not_allowed"
                @deleteCart="removeCart" @backToRestaurant="getRestaurantDetails" />



            <button class="btn  cart-mobile-btn" type="button" data-bs-toggle="offcanvas" data-bs-target="#offcanvasTop" aria-controls="offcanvasTop">
                <i class="fa-solid fa-cart-shopping"></i>
                <span v-if="cartProducts.length > 0">{{ cartProducts.length }}</span>
            </button>

            <div class="offcanvas offcanvas-top ms_trans" tabindex="-1" id="offcanvasTop"
                aria-labelledby="offcanvasTopLabel">
                <div class="offcanvas-header">
                    <h5 class="offcanvas-title" id="offcanvasTopLabel">Offcanvas top</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="offcanvas" aria-label="Close"></button>
                </div>
                <div class="offcanvas-body ">
                    <AppCart class="mobile" :cartProducts="cartProducts" :not_allowed="not_allowed" @deleteCart="removeCart"
                        @backToRestaurant="getRestaurantDetails" />

                </div>
            </div>

        </div>
    </section>
</template>

<style lang="scss" scoped>
@use "../styles/partials/root.scss" as *;

.desktop{
    @media screen and (max-width: 768px){
        display: none;
    }
    
}



section {
    background-color: #FFF3DA;

}

.container {
    display: flex;
    padding-right: 0;

    .ms_trans {
        background-color: transparent;
    }

    .cart-mobile-btn {
        position: relative;
        width: 50px;
        height: 50px;
        border-radius: 100%;
        top: 50px;
        margin: 10px;
        display: none;
        position: sticky;
        background-color: #ffffff;
        box-shadow: 0px 10px 15px rgb(166, 166, 166);
        i{
            color: black;
        }
        span{
            display: inline-block;
            width: 25px;
            height: 25px;
            border-radius: 100%;
            background-color: #FFDD44;
            position: absolute;
            bottom: -10px;
            font-weight: 500;
            font-size: 1.2rem;
            line-height: 25px;
            left: -5px;
            z-index: 3;
            color: #2c2c2c;
            box-shadow: 0px -3px 15px rgb(124, 124, 124);
        }

        @media screen and (max-width: 768px){
            display: block;
        }
    }

   

    @media screen and (max-width: 768px) {
        .restaurant-details-col {
            width: 100%;
        }

    }

    .restaurant-details-col {
        width: 60%;

        @media screen and (max-width: 768px) {
            width: 100%;
        }

        .restaurant-detail-card-container {
            position: relative;
            min-height: 110px;

            .restaurant-detail-card {

                position: absolute;
                padding: 1rem 2rem;
                top: -75px;
                border-radius: 20px;
                box-shadow: 0px 10px 15px rgb(166, 166, 166);
                background-color: #ffffff;
                left: 0px;
                right: 0;


                h2 {
                    margin: 0;
                    font-weight: 700;
                    font-size: 3rem;
                }

                p {
                    margin: 0;
                    font-size: .8rem;
                    color: #797979;
                }

                p.description {
                    font-size: .9rem;
                    text-align: left;
                    color: black;
                }
            }
        }

        .restaurant-products {

            display: flex;
            height: 100%;



            .product-list {
                width: 100%;
                padding: .5rem .5rem;

                .product-card {
                    .card-body {
                        border: 1px solid grey;
                        background-color: #ffff;
                        border-radius: 10px;
                        display: flex;
                        overflow: hidden;
                        box-shadow: 0px 10px 15px rgb(166, 166, 166);

                    }

                    .product-img {
                        width: 30%;

                        img {
                            object-fit: cover;
                            height: 100%;
                        }
                    }

                    .product-content {
                        width: 70%;
                        min-height: 100px;
                        padding: .5rem .5rem;
                        flex-direction: column;
                        justify-content: space-between;

                        .product-description {
                            height: 70%;

                            p.product-detail {
                                font-size: .75rem;
                                color: #797979;
                            }
                        }

                        .product-price {
                            height: 30%;

                            .buy {
                                display: inline-block;
                                padding: 0 .7rem;
                                line-height: 30px;
                                height: 30px;
                                font-size: .7rem;
                                background-color: $primary_color;
                                border-radius: 50px;
                                cursor: pointer;
                                transition: 200ms;

                                &:active {
                                    transform: scale(.8);
                                }
                            }
                        }
                    }
                }
            }
        }
    }


}
</style>