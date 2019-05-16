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
        <button
            v-if="showLeftButton"
            @click="slideLeft"
        >
            Left
        </button>
        <button
            v-if="showRightButton"
            @click="slideRight"
        >
            Right
        </button>
    </div>
</template>

<script>
    export default {
        name: 'Carousel',

        props: {

        },

        data() {
            return {
                element: null,
                coordX: 0,
                translate: 0,
                transitionDuration: 0.2,
                elementsWidth: [],
                wrapperWidth: 0,
                carouselWidth: 0,
                startPosition: 0,
                direction: 0,
                showRightButton: true,
                showLeftButton: true,
                mutationObserver: null
            }
        },

        created() {
            this.handleMutation = this.debounce(this.updateWidth, 0);
        },

        mounted() {
            this.initMousedown();
            this.initMousemove();
            this.initMouseup();
            this.updateWidth();
            this.initObserver();
        },

        beforeDestroy() {
            this.mutationObserver.disconnect();
        },

        computed: {
            getMaxOffset() {
                return (this.wrapperWidth > this.carouselWidth ?
                    this.wrapperWidth - this.carouselWidth
                    : 0);
            }
        },

        methods: {
            isBeforeStart(translate, offset = 0) {
                return translate + offset > 0;
            },

            isAfterEnd(translate, offset = 0) {
                if (this.wrapperWidth >= this.carouselWidth) {
                    return Math.abs(translate + offset) >= Math.abs(this.wrapperWidth - this.carouselWidth);
                }
                else {
                    return true;
                }
            },

            initObserver() {
                this.mutationObserver = new MutationObserver((mutations) => {
                    mutations.forEach((mutation) => {
                        this.handleMutation();
                    });
                });

                let config = { childList: true };

                this.mutationObserver.observe(this.$refs.wrapper, config);
            },

            initMousedown() {
                document.addEventListener('mousedown', (e) => {
                    if (!(this.$refs.wrapper && this.$refs.wrapper.contains(e.target))) return;
                    this.element = e.target;
                    this.coordX = e.pageX;
                    this.element.removeEventListener('click', this.preventClick);
                })
            },

            initMousemove() {
                document.addEventListener('mousemove', (e) => {
                    if (!this.element) return;

                    this.transitionDuration = 0;

                    let moveX = e.pageX - this.coordX;

                    if ( Math.abs(moveX) < 3 ) {
                        return;
                    }

                    if (this.isBeforeStart(this.translate, moveX) || this.isAfterEnd(this.translate, moveX)) {
                        this.translate += moveX / 10;
                    }
                    else {
                        this.translate += moveX;
                    }

                    this.direction = moveX >= 0 ? -1 : 1;
                    this.coordX = e.pageX;
                })
            },

            preventClick(e) {
                e.preventDefault();
            },

            initMouseup() {
                document.addEventListener('mouseup', (e) => {
                    if (!this.element) return;

                    if (!this.direction) {
                        this.element = null;
                        this.transitionDuration = 0.5;
                        return;
                    }

                    if (this.element === e.target) {
                        this.element.addEventListener('click', this.preventClick);
                    }

                    let currentPosition = this.getPositionByOffset(this.translate);
                    let currentTranslate = this.getOffsetByPosition(currentPosition);

                    if (this.direction > 0) {
                        currentPosition++;
                    }

                    currentTranslate = -this.getOffsetByPosition(currentPosition);


                    if (this.isBeforeStart(currentTranslate)) currentTranslate = 0;

                    if (this.isAfterEnd(currentTranslate)) {
                        currentTranslate = -this.getMaxOffset;
                    }

                    currentPosition = this.getPositionByOffset(currentTranslate);

                    this.direction = 0;
                    this.startPosition = currentPosition;
                    this.translate = currentTranslate;
                    this.element = null;
                    this.transitionDuration = 0.5;
                })
            },

            updateWidth() {
                this.wrapperWidth = 0;
                this.elementsWidth = [];
                this.carouselWidth = this.$refs.carousel.offsetWidth;

                [...this.$refs.wrapper.childNodes].forEach(item => {
                    this.wrapperWidth += this.getElementWidth(item);
                    this.elementsWidth.push(this.getElementWidth(item));
                });
            },

            getInMiddleOfItem(position) {
                let result = false;
                for (let index = 0; index < this.elementsWidth.length; index++) {
                    let currentOffset = this.getOffsetByPosition(index);
                    let nextOffset = this.getOffsetByPosition(index + 1);

                    if (
                        this.getOffsetByPosition(position) > currentOffset &&
                        this.getOffsetByPosition(position) < nextOffset
                    ) {
                        result = true;
                        break;
                    }
                }

                return result;
            },

            getPositionByOffset(offset) {
                let position = 0;

                if (this.isBeforeStart(offset)) {
                    offset = 0;
                }

                if (this.isAfterEnd(offset)) {
                    offset = -this.getMaxOffset;
                }

                for (let index = 0; index < this.elementsWidth.length; index++) {
                    let currentOffset = this.getOffsetByPosition(index);
                    let nextOffset = this.getOffsetByPosition(index + 1);

                    if (
                        Math.abs(offset) >= currentOffset &&
                        Math.abs(offset) < nextOffset
                    ) {
                        position = index;
                        break;
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
                if (Math.abs(this.translate) === this.getMaxOffset) return;

                let currentPosition = this.startPosition;

                currentPosition++;

                if (this.isAfterEnd(this.getOffsetByPosition(currentPosition))) {
                    this.translate = -this.getMaxOffset;
                }
                else {
                    this.translate = -this.getOffsetByPosition(currentPosition);
                }

                // console.log(this.getInMiddleOfItem(currentPosition), currentPosition)
                //
                // if (this.getInMiddleOfItem(currentPosition)) {
                //     currentPosition = this.getPositionByOffset(this.translate) + 1;
                // }
                // else {
                //     currentPosition = this.getPositionByOffset(this.translate);
                // }

                this.startPosition = this.getPositionByOffset(this.translate);
            },

            slideLeft() {
                if (this.translate === 0) return;

                let currentPosition = this.startPosition;

                currentPosition--;

                if (this.isAfterEnd(this.getOffsetByPosition(currentPosition))) {
                    this.translate = 0;
                }
                else {
                    this.translate = -this.getOffsetByPosition(currentPosition);
                }

                this.startPosition = this.getPositionByOffset(this.translate);
            },

            debounce(f, ms) {
                let timer = null;

                return function (...args) {
                    const onComplete = () => {
                        f.apply(this, args);
                        timer = null;
                    };

                    if (timer) {
                        clearTimeout(timer);
                    }

                    timer = setTimeout(onComplete, ms);
                };
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

            a, img {
                -webkit-user-drag: none;
            }
        }
    }
</style>
