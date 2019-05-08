<template>
    <div>
        <div
            ref="carousel"
            class='carousel'
        >
            <div
                ref="wrapper"
                class="carousel__wrapper"
                :style="{
                    'transform': 'translateX(' + translate + 'px)',
                    'transition': 'transform ' + transitionDuration + 's ease',
                    'width': wrapperWidth + 'px'
                }"
            >
                <slot></slot>
            </div>
        </div>
        <button @click="slideLeft">Left</button>
        <button @click="slideRight">Right</button>
    </div>
</template>

<script>
    export default {
        name: 'Carousel',

        props: {

        },

        data() {
            return {
                hasElement: false,
                coordX: 0,
                translate: 0,
                transitionDuration: 0.2,
                elementsWidth: [],
                wrapperWidth: 0,
                carouselWidth: 0,
                startPosition: 0,
                direction: 0
            }
        },

        mounted() {
            this.initMousedown();
            this.initMousemove();
            this.initMouseup();
            this.checkWidth();
        },

        methods: {
            isBeforeStart(offset = 0) {
                return this.translate + offset > 0;
            },

            isAfterEnd(offset = 0) {
                if (this.wrapperWidth >= this.carouselWidth) {
                    return Math.abs(this.translate + offset) >= Math.abs(this.wrapperWidth - this.carouselWidth);
                }
                else {
                    return true;
                }
            },

            initMousedown() {
                document.addEventListener('mousedown', (e) => {
                    if (!(this.$refs.wrapper && this.$refs.wrapper.contains(e.target))) return;
                    this.hasElement = true;
                    this.coordX = e.pageX;
                })
            },

            initMousemove() {
                document.addEventListener('mousemove', (e) => {
                    if (!this.hasElement) return;

                    this.transitionDuration = 0;

                    let moveX = e.pageX - this.coordX;

                    if ( Math.abs(moveX) < 3 ) {
                        return;
                    }

                    if (this.isBeforeStart(moveX) || this.isAfterEnd(moveX)) {
                        this.translate += moveX / 10;
                    }
                    else {
                        this.translate += moveX;
                    }

                    this.direction = moveX >= 0 ? -1 : 1;
                    this.coordX = e.pageX;
                })
            },

            initMouseup() {
                document.addEventListener('mouseup', (e) => {
                    if (!this.hasElement) return;
                    this.hasElement = false;
                    this.transitionDuration = 0.5;
                    this.startPosition = this.getPositionByOffset();

                    if (this.isBeforeStart()) {
                        this.translate = 0;
                        return;
                    }

                    this.translate = -this.getOffsetByPosition(this.direction > 0 ? this.startPosition + 1 : this.startPosition);
                    this.direction = 0;

                    if (this.isAfterEnd()) {
                        this.translate = -(this.wrapperWidth > this.carouselWidth ?
                            this.wrapperWidth - this.carouselWidth
                            : 0
                        );
                    }

                    this.startPosition = this.getPositionByOffset();
                })
            },

            checkWidth() {
                this.carouselWidth = this.$refs.carousel.offsetWidth;

                [...this.$refs.wrapper.childNodes].forEach(item => {
                    this.wrapperWidth += this.getElementWidth(item);
                    this.elementsWidth.push(this.getElementWidth(item));
                })
            },

            getPositionByOffset() {
                let position = 0;

                for (let index = 0; index < this.elementsWidth.length; index++) {
                    let currentOffset = this.getOffsetByPosition(index);
                    let nextOffset = this.getOffsetByPosition(index + 1);

                    if (
                        Math.abs(this.translate) >= currentOffset &&
                        Math.abs(this.translate) < nextOffset
                    ) {
                        position = index;
                    }
                }

                return position;
            },

            getElementWidth(element) {
                let style = element.currentStyle || window.getComputedStyle(element),
                    width = element.offsetWidth,
                    margin = parseFloat(style.marginLeft) + parseFloat(style.marginRight),
                    border = parseFloat(style.borderLeftWidth) + parseFloat(style.borderRightWidth);

                return width + margin + border;
            },

            getOffsetByPosition(index) {
                return this.elementsWidth
                    .slice(0, index)
                    .reduce((sum, item) => sum + item, 0);
            },

            slideRight() {
                if (this.isAfterEnd()) return;

                this.startPosition++;
                this.translate = -this.getOffsetByPosition(this.startPosition);
            },

            slideLeft() {
                if (this.startPosition === 0) return;

                this.startPosition--;
                this.translate = -this.getOffsetByPosition(this.startPosition);

                if (this.isBeforeStart()) {
                    this.translate = 0;
                }
            }
        }
    }
</script>

<style scoped lang="scss">
    .carousel {
        width: 1000px;
        height: 200px;
        margin: 0 auto;
        overflow: hidden;
        position: relative;

        &__wrapper {
            display: flex;
            transition: transform 0.5s;
            will-change: transform;
            position: relative;
            height: 100%;
            user-select: none;
        }
    }
</style>
