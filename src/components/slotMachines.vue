<template>
    <div class="slot-machines">
        <div class="list" ref="list" v-for="(list, index) in numbers" :key="index" :class="{'animal': listAnimal}"
        @touchstart.prevent="e => handleTouchStart(e, index)"
        @touchmove.prevent="e => handleTouchMove(e, index)"
        @touchend.prevent="e => handleTouchEnd(e, index)"
        >
            <div class="item" ref="item"  v-for="(item, index) in itemlength" :key="index"></div>
        </div>
    </div>
</template>

<script>
    import random from "lodash/random"
    export default {
        data() {
            return {
                itemlength: 20,
                perAngleH: 0,
                numbers: [0, 0, 0, 0, 0],
                listAnimal: false,
                perAngle: 0,
                touchStatus: false,
                startY: 0,
                endY: 0,
                startAngle: 0
            }
        },
        mounted() {
            this.perAngle = 360/this.itemlength;
            this.perAngleH = this.$refs.item[0].clientHeight / this.perAngle;
            this.$refs.item.forEach((item, index) => {
                let result = this.setStyle(index);
                item.style.transform = result.transform;
                item.style.backgroundPosition = result.backgroundPosition;
            });
        },
        methods: {
            setStyle(index) { //根据每个item的高度和对应的角度计算偏移量
                let itemHeight = this.$refs.item[0].clientHeight;
                let zLength = this.tzLength(itemHeight / 2, this.perAngle / 2);
                return {
                    transform: `rotateX(${index * 18 * -1}deg) translateZ(${zLength}px)`,
                    backgroundPosition: `0 ${(index % 10) * itemHeight * -1}px`
                };
            },
            getCurrentAngle(index) {
                return this.$refs.list[index].style.transform.slice(8, -4) * 1;
            },
            getCurrentNumsByAngle(angle) {
                let temp = angle%360 >= 0 ? angle%360/this.perAngle : (360+angle%360)/this.perAngle;
                return temp%10;
            },
            tzLength(long, angle) {
                return (
                    Math.round((long / Math.tan((angle * Math.PI) / 180)) * 100000) / 100000
                );
            },
            setListTransform(index, angle) {
                this.$refs.list[index].style.transform = `rotateX(${angle}deg)`;
            },
            numbersByRandom() {
                let newNumbers = [0,0,0,0,0].map(() => {
                    return random(0, 9)
                })
                this.listAnimal = true;
                this.$refs.list.forEach((item, index) => {
                    item.style.transitionDuration = 7 + index * 0.2 + "s";
                    let current = this.getCurrentAngle(index);
                    let MoveAngle = (newNumbers[index] - this.numbers[index]) * this.perAngle + 720 + current;
                    this.setListTransform(index, MoveAngle);
                })
                this.numbers = newNumbers;
            },
            handleTouchStart(e ,index) {
                document.body.style.overflow = "hidden";
                this.touchStatus = true;
                let touch = e.targetTouches[0];
                this.startY = touch.pageY;
                this.$refs.list[index].style.transitionDuration = "";
                this.startAngle = this.getCurrentAngle(index);
            },
            handleTouchMove(e, index) {
                if (this.touchStatus) {
                    let touch = e.targetTouches[0];
                    this.endY = touch.pageY;
                    let MoveY = (this.endY - this.startY) / 1;
                    let MoveAngle = this.startAngle + MoveY * -1 / this.perAngleH;
                    this.setListTransform(index, MoveAngle);
                }   
            },
            handleTouchEnd(e, index) {
                let currentAngle = this.getCurrentAngle(index);
                let MoveAngle =
                    (currentAngle % this.perAngle + this.perAngle) % this.perAngle >= this.perAngle / 2
                    ? Math.ceil(currentAngle / this.perAngle) * this.perAngle
                    : Math.floor(currentAngle / this.perAngle) * this.perAngle;
                this.$refs.list[index].style.transitionDuration = "500ms";
                this.setListTransform(index, MoveAngle);
                this.numbers.splice(index, 1, this.getCurrentNumsByAngle(MoveAngle));
                this.touchStatus = false;
                this.endY = this.startY = this.startAngle = 0;
                document.body.style.overflow = "auto";
            },
        },
    }
</script>

<style scoped>
.slot-machines {
    margin-left: 100px;
    display: flex;
    align-items: center;
}
.slot-machines .list.listAnimal {
    transition-property: transform;
    transition-timing-function: cubic-bezier(0.1, 0.1, 0.15, 0.9);
}
.slot-machines .list {
    position: relative;
    transform-style: preserve-3d;
    width: 120px;
    height: 180px;
    transform: rotateX(0deg);
}
.slot-machines .item{
    position: absolute;
    left: 50%;
    top: 50%;
    border: 0px solid rgba(0, 0, 0, 0);
    box-sizing: border-box;
    background-image: url("../assets/images/numStrap.png");
    background-repeat: no-repeat;
    background-size: 100% 900px;
    background-position: 0 0;
    background-color: #fff;
    width: 120px;
    height: 90px;
    margin-left: -60px;
    margin-top: -45px;
}
</style>