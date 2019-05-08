<template>
    <div id="app">
        <div class="header">
            <img alt="Vue logo" src="./assets/logo.png">
            <span class="header__title">ue Carousel</span>
        </div>
        <Carousel>
            <div
                v-for="item in items"
                class="carousel__item"
            >
                {{item}}
            </div>
            <!--<div class="carousel__item">-->
                <!--2-->
            <!--</div>-->
            <!--<div class="carousel__item" style="min-width: 400px">-->
                <!--3-->
            <!--</div>-->
            <!--<div class="carousel__item">-->
                <!--4-->
            <!--</div>-->
            <!--<div class="carousel__item" style="min-width: 1000px">-->
                <!--5-->
            <!--</div>-->
            <!--<div class="carousel__item" >-->
                <!--6-->
            <!--</div>-->
            <!--<div class="carousel__item">-->
                <!--7-->
            <!--</div>-->
            <!--<div class="carousel__item">-->
                <!--8-->
            <!--</div>-->
            <!--<div class="carousel__item">-->
                <!--9-->
            <!--</div>-->
        </Carousel>
    </div>
</template>

<script>
import Carousel from './components/Carousel.vue'

export default {
    name: 'app',
    components: {
        Carousel
    },

    data() {
        return {
            items: []
        }
    },

    mounted() {
        this.getData();
    },

    methods: {
        getData() {
            setTimeout(() => {
                this.items = [...Array(9).keys()].map(item => item + 1);
            }, 500)

            setTimeout(() => {
                this.items.push(11)
            }, 3000)
        }
    }
}
</script>

<style lang="scss">
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }

    .header {
        display: inline-block;
        position: relative;
        margin-bottom: 40px;

        &__title {
            position: absolute;
            top: 146px;
            white-space: nowrap;
            left: 114px;
            font-size: 38px;
            color: #35495e;
        }
    }

    .carousel__item {
        display: flex;
        align-items: center;
        justify-content: center;
        height: 100%;
        width: 200px;
        min-width: 200px;
        margin-right: 10px;
        font-size: 96px;
        color: #fff;
        background-color: blue;
    }
</style>
