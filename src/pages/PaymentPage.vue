
<script>
import { store } from '../store';
import axios from 'axios';
import braintree from 'braintree-web';
import SectionJumbo from '../components/SectionJumbo.vue';
import AppCart from "../components/AppCart.vue";

export default {
    name: "PaymentPage",
    data() {
        return {
            store,
            hostedFieldInstance: false,
            cartProducts: [],
            detailsProducts: [],
            token: "",
            nameSurname: "",
            address: "",
            email: "",
            notes: "",
            cardOwner: "",
            hover: false,
            products: [],
            cartProducts: [],

        }
    },
    mounted() {

        this.cartProducts = this.store.method.getArray();
        // token

        axios.get(this.store.ApiToken).then((resp) => {


            braintree.client.create({
                authorization: resp.data.token

            })
                .then(clientInstance => {
                    let options = {
                        client: clientInstance,
                        styles: {
                            input: {
                                'font-size': '15px',
                                'font-family': 'Open Sans'
                            }
                        },
                        fields: {
                            number: {
                                selector: '#creditCardNumber',
                                placeholder: '0000-0000-0000-0000'
                            },
                            cvv: {
                                selector: '#cvv',
                                placeholder: '123'
                            },
                            expirationDate: {
                                selector: '#expireDate',
                                placeholder: '00 / 00'
                            }
                        }
                    }
                    return braintree.hostedFields.create(options)
                })
                .then(hostedFieldInstance => {
                    // @TODO - Use hostedFieldInstance to send data to Braintree
                    this.hostedFieldInstance = hostedFieldInstance;
                })
                .catch(err => {
                });


        })




    },
    methods: {

        getIdQuantity() {
            this.cartProducts.forEach(product => {
                const products = {
                    id: product.id,
                    quantity: product.quantity,
                }
                this.detailsProducts.push(products);

            })
        },
        sendPayment() {

            if (this.hostedFieldInstance) {
                this.getIdQuantity();

                this.hostedFieldInstance.tokenize().then(payload => {
                    axios.post(`${this.store.ApiPayment}`, {

                        // array oggetti id quantità
                        cart: this.detailsProducts,
                        // token
                        token: payload.nonce,
                        // array oggetto user
                        customer_name_surname: this.nameSurname,
                        customer_address: this.address,
                        customer_notes: this.notes,
                        customer_email: this.email,



                    }).then(resp => {
                        this.store.method.delete();
                        this.$router.push({ path: '/restaurants', query: { success: true } });
                        console.log(resp);
                    })
                })
                    .catch(err => {
                        console.error(err);
                    })
            }
        },

        // FUNZIONI PER RIEPILOGO
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

        getPrice(obj) {
            return parseFloat(obj.price) * obj.quantity
        },

        decrementProduct(index) {
            const product = this.cartProducts[index];
            if (product.quantity > 1) {
                product.quantity--;
            } else {
                this.removeObj(index);
            }
            this.store.method.salva(this.cartProducts);

        },
        removeObj(index) {

            if (index >= 0 && index < this.cartProducts.length) {
                this.cartProducts.splice(index, 1);
                this.store.method.salva(this.cartProducts);
            }
        },

        getTotal() {
            let total = 0;
            this.cartProducts.forEach(product => {
                total += parseFloat(product.price) * product.quantity
            })
            return total
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
        AppCart,
        SectionJumbo
    }
}

</script>
<template>
    <SectionJumbo />
    <section class="payment-page">

        <div class="container py-4">
            <!-- CARTA  ------------------------------------------------------------------>
            <div class="card-section-container">

                <div class="card-container">

                    <div :class="hover ? 'front-rotate' : ''" class="front">
                        <div class="image">
                            <img src="../assets/chip.png" alt="">
                            <img src="../assets/visa.png" alt="">
                        </div>
                        <div class="card-number-box">0000-0000-0000-0000</div>
                        <div class="flexbox">
                            <div class="box">
                                <span>Intestatario</span>
                                <div class="card-holder-name">{{ cardOwner }}</div>
                            </div>
                            <div class="box">
                                <span>Scade il</span>
                                <div class="expiration">
                                    <span class="exp-month">00 /</span>
                                    <span class="exp-year"> 00</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div :class="hover ? 'back-rotate' : ''" class="back">
                        <div class="stripe"></div>
                        <div class="box">
                            <span>cvv</span>
                            <div class="cvv-box">123</div>
                            <img src="../assets/visa.png" alt="">
                        </div>
                    </div>

                </div>
            </div>
            <!-- /CARTA----------------------------------------------------------------->


            <!-- FORM SPEDIZIONE-------------------------------------------------------->
            <div class="shipping-payment text-center ">
                <h2 class="text-center">Completa il tuo Ordine</h2>
                <div class="forms d-flex mt-4">
                    <!-- shipping form info -->
                    <div class="shipping text-start">
                        <h5 class="text-center">informazioni di consegna</h5>
                        <form>

                            <label for="name">Nome e Cognome: *</label>
                            <input class="mb-2" type="text" id="name" v-model="nameSurname">

                            <label for="address">indirizzo di Consegna: *</label>
                            <input class="mb-2" type="text" id="address" v-model="address">

                            <label for="email">email: *</label>
                            <input class="mb-2" type="email" id="email" v-model="email">

                            <label for="text">Note: </label>
                            <textarea class="mb-2" id="text" v-model="notes"></textarea>

                        </form>
                    </div>
                    <!-- /shipping form info -->

                    <!-- Credit Card from Info -->
                    <div class="payment text-start">
                        <h5 class="text-center">Dettagli Carta</h5>
                        <form>
                            <label for="cardOwner">Intestatario Carta</label>
                            <input type="text" class="mb-2" v-model="cardOwner" id="cardOwner">

                            <label for="creditCardNumber">Numero Carta</label>
                            <div id="creditCardNumber" class="form-control mb-2"></div>

                            <label for="expireDate">Data di Scadenza</label>
                            <div id="expireDate" class="form-control mb-2"></div>

                            <div @mouseenter="hover = true" @mouseleave="hover = false" class="cvv-input">
                                <label for="cvv">CVV</label>
                                <div id="cvv" class="form-control mb-2"></div>
                            </div>
                        </form>
                    </div>
                </div>

                <hr>

                <!-- RIEPILOGO ORDINE -->
                <h5 class="text-center">Riepilogo ordine</h5>
                <div class="cart-summary py-3">
                    <div class="mb-2 cart-item" v-for="obj, index in cartProducts">
                        <p>{{ obj.name }} x {{ obj.quantity }} : {{ getPrice(obj) }}&euro; </p>
                        <div class="cart-actions">
                            <button class="cart-btn minus" @click="decrementProduct(index)">&minus;</button>
                            <button class="cart-btn" @click="newObj(obj, index)">&plus;</button>
                        </div>
                    </div>

                </div>
                <div class="mb-3 fs-3">Totale: {{ getTotal() }}&euro;</div>
                <div class="final-actions d-flex justify-content-center gap-2">
                    <!-- <span class="btn btn-danger ms_btn">Svuota Carrello</span> -->
                    <!-- <AppCart @deleteCart="removeCart"/> -->
                    <!-- <span class="btn ms_btn btn-success" @click="$emit('backToRestaurant', cartProducts[0].restaurant)">Torna al ristorante</span> -->
                </div>

                <!-- / RIEPILOGO ORDINE -->



                <!-- /Credit Card from Info -->
                <button class="submit-btn" @click="sendPayment()"> Procedi all' ordine</button>

                <hr>

                <p class="text-center mb-2">
                    Ti invieremo un email con i dettagli dell' ordine ad avvenuto pagamento
                </p>
                <p>* campi obbligatori</p>
            </div>

        </div>

    </section>
</template>


<style lang="scss" scoped>

section.payment-page {
    background-color: #EEEEEE;
    padding: 2rem 0;

    min-height: 1200px;

    position: relative;

    @media screen and (max-width: 768px) {
        min-height: 1500px;

    }

    .container {
        background-color: #ffff;
        position: absolute;
        border-radius: 25px;
        top: -40px;
        left: 50%;
        transform: translateX(-50%);
        box-shadow: 0px 10px 30px rgb(77, 77, 77);
        z-index: 3;

        .shipping-payment {
            .submit-btn {
                display: inline-block;
                width: 50%;
                background-color: #90EE90;
                color: black;
                margin-top: 20px;
                padding: 10px;
                font-size: 20px;
                box-shadow: 0px 10px 10px rgb(163, 163, 163);
                border-radius: 10px;
                cursor: pointer;
                transition: 200ms;
                border: none;
                &:active{
                    transform: scale(.9);
                }
            }

            @media screen and (max-width:768px) {
                .forms {
                    flex-direction: column-reverse;
                    align-items: center;
                    gap: 2rem;

                    .payment,
                    .shipping {
                        width: 70%;

                    }
                }
            }

            .payment,
            .shipping {
                width: 50%;
                padding: 0 .5rem;

                form {
                    display: flex;
                    flex-direction: column;

                    .form-control {
                        height: 26px;
                        border-radius: 10px;
                        border-color: grey;
                        width: 100%;
                        transition: 200ms;


                    }

                    label {
                        display: block;
                        color: #585858;
                    }

                    input,
                    textarea {
                        border-radius: 10px;
                        border: 1px solid grey;
                        padding: 0 .8rem;
                        transition: 200ms;

                        &:focus {
                            outline: none;

                        }
                    }
                }
            }

            .cart-summary {
                width: 70%;
                max-height: 300px;
                min-height: 100px;
                margin: 0 auto;
                overflow: auto;
                &::-webkit-scrollbar {
                    background: transparent;
                    
                }

                &::-webkit-scrollbar-thumb {
                    background-color: #90EE90;
                    border-radius: 10px;
                }

                .cart-item {

                    .ms_btn {
                        width: 130px;
                        border-radius: 40px;
                        font-size: .8rem;
                        margin: 0 5px;
                    }

                    .btn-empty {
                        background-color: lightcoral;
                    }

                    display: flex;
                    width: 100%;
                    padding-bottom: 1rem;
                    border-bottom: 1px solid rgb(218, 218, 218);

                    p {
                        width: 70%;
                    }

                    .cart-actions {
                        width: 30%;

                        .cart-btn {

                            cursor: pointer;
                            margin: 0 2px;
                            height: 25px;
                            width: 25px;
                            border: none;
                            background-color: lightgreen;
                            display: inline-block;
                            border-radius: 100%;

                            &.minus {
                                background-color: lightcoral;
                            }
                        }
                    }
                }
            }

        }

        .card-section-container {

            display: flex;
            align-items: center;
            justify-content: center;
            flex-flow: column;
            margin-bottom: 2rem;

            .card-container {

                position: relative;
                height: 250px;
                margin-bottom: -100px;
                width: 400px;

                @media screen and (max-width: 450px) {
                    width: 100%;
                }

                @media screen and (max-width : 330px) {
                    display: none;
                }

                .back {
                    position: absolute;
                    top: -100px;
                    left: 0;
                    height: 100%;
                    width: 100%;
                    background: linear-gradient(45deg, blueviolet, deeppink);
                    border-radius: 5px;
                    padding: 20px 0;
                    text-align: right;
                    backface-visibility: hidden;
                    box-shadow: 0 15px 25px rgba(0, 0, 0, .2);
                    transform: perspective(1000px) rotateY(180deg);
                    transition: transform .4s ease-out;

                    .cvv-box {
                        height: 50px;
                        padding: 10px;
                        margin-top: 5px;
                        color: #333;
                        background: #fff;
                        border-radius: 5px;
                        width: 100%;
                    }

                    .stripe {
                        background: #000;
                        width: 100%;
                        margin: 10px 0;
                        height: 50px;
                    }

                    .box {
                        padding: 0 20px;

                        span {
                            color: #fff;
                            font-size: 15px;
                        }

                        img {
                            margin-top: 30px;
                            height: 30px;
                        }
                    }
                }

                .front {
                    position: absolute;
                    height: 100%;
                    width: 100%;
                    top: -100px;
                    left: 0;
                    background: linear-gradient(45deg, blueviolet, deeppink);
                    border-radius: 5px;
                    backface-visibility: hidden;
                    box-shadow: 0 15px 25px rgba(0, 0, 0, .2);
                    padding: 20px;
                    transform: perspective(1000px) rotateY(0deg);
                    transition: transform .4s ease-out;

                    .card-number-box {
                        padding: 30px 0;
                        font-size: 22px;
                        color: #fff;
                    }

                    .image {
                        display: flex;
                        align-items: center;
                        justify-content: space-between;
                        padding-top: 10px;

                        img {
                            height: 50px;
                        }
                    }

                    .flexbox {
                        display: flex;

                        .box:nth-child(1) {
                            margin-right: auto;
                        }

                        .box {
                            font-size: 15px;
                            color: #fff;

                        }
                    }
                }
            }
        }
    }
}

.front-rotate {
    transform: perspective(1000px) rotateY(-180deg) !important;
}

.back-rotate {
    transform: perspective(1000px) rotateY(0deg) !important;
}
</style>