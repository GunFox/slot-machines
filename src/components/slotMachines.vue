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
    export default {
        props: {
            lotterynumnber: String //接受外部初始化号码
        },
        data() {
            return {
                itemlength: 20, //每列item的长度
                perAngleH: 0, //一度对应item的高度
                numbers: [0, 0, 0, 0, 0], //号码
                listAnimal: false,
                perAngle: 0, //每个item对应的度数
                touchStatus: false,
                startY: 0, //触摸开始时的Y值
                endY: 0, //触摸结束时的Y值
                startAngle: 0, //触摸开始时的角度,
                roundAnimal: false, //防止重复调用随机模式
                updateTimeout: null //随机模式下返回新的号码定时器，增强交互性
            }
        },
        mounted() { //页面不存在缓存时初始化老虎机动画
            this.init();
        },
        activated() { //页面存在缓存时初始化老虎机数据
            this.init();
        },
        watch: {
            lotterynumnber(now, old) { //重置初始化时不再触发滚动动画
            console.log(34444)
                if(old && !now) {
                    this.$refs.list.forEach((item, index) => {
                        item.style.transitionDuration = "0s";
                        let MoveAngle = 0;
                        this.setListTransform(index, MoveAngle);
                    });
                    clearTimeout(this.updateTimeout);
                    this.updateTimeout = null;
                    this.numbers = [0, 0, 0, 0, 0];
                }
            }
        },
        methods: {
            init() {
                this.perAngle = 360/this.itemlength;
                this.perAngleH = this.$refs.item[0].clientHeight / this.perAngle;
                this.$refs.item.forEach((item, index) => {
                    let result = this.setStyle(index);
                    item.style.transform = result.transform;
                    item.style.backgroundPosition = result.backgroundPosition;
                });
            },
            resetNumbers() {
                if(this.roundAnimal) {
                    this.roundAnimal = !this.roundAnimal;
                }
                this.$refs.list.forEach((item, index) => {
                    item.style.transitionDuration = "0s";
                    let MoveAngle = 0;
                    this.setListTransform(index, MoveAngle);
                });
                clearTimeout(this.updateTimeout);
                this.updateTimeout = null;
                this.numbers = [0, 0, 0, 0, 0];
                this.updateNumbers();
            },
            setStyle(index) { //根据每个item的高度和对应的角度计算偏移量
                let itemHeight = this.$refs.item[0].clientHeight;
                let zLength = this.tzLength(itemHeight / 2, this.perAngle / 2);
                return {
                    transform: `rotateX(${index * 18 * -1}deg) translateZ(${zLength}px)`,
                    backgroundPosition: `0 ${(index % 10) * itemHeight * -1}px`
                };
            },
            getCurrentAngle(index) { //获取每个item的当前角度
                return this.$refs.list[index].style.transform.slice(8, -4) * 1;
            },
            getCurrentNumsByAngle(angle) { //根据角度获取对应的数字
                let temp = angle%360 >= 0 ? angle%360/this.perAngle : (360+angle%360)/this.perAngle;
                return temp%10;
            },
            tzLength(long, angle) { //计算出每个item沿Z轴平移距离
                return (
                    Math.round((long / Math.tan((angle * Math.PI) / 180)) * 100000) / 100000
                );
            },
            setListTransform(index, angle) { //设置item应旋转的角度
                this.$refs.list[index].style.transform = `rotateX(${angle}deg)`;
            },
            numbersByRandom() { //提供给外部调用随机生成号码功能
                if(this.roundAnimal) { //防止重复调用
                    return false;
                }
                this.roundAnimal = !this.roundAnimal;
                let newNumbers = [0,0,0,0,0].map(() => {
                    return Math.floor(Math.random()*10);
                });
                this.listAnimal = true;
                this.$refs.list.forEach((item, index) => {
                    item.style.transitionDuration = 7 + index * 0.2 + "s";
                    let current = this.getCurrentAngle(index);
                    let MoveAngle = (newNumbers[index] - this.numbers[index]) * this.perAngle + 720 + current;
                    this.setListTransform(index, MoveAngle);
                });
                this.numbers = newNumbers;
                this.updateTimeout = setTimeout(() => { //延时返回数据，增强交互性
                    this.updateNumbers();
                    this.roundAnimal = !this.roundAnimal;
                },7800);
            },
            handleTouchStart(e ,index) { //触摸开始
                document.body.style.overflow = "hidden";
                this.touchStatus = true;
                let touch = e.targetTouches[0];
                this.startY = touch.pageY;
                this.$refs.list[index].style.transitionDuration = "";
                this.startAngle = this.getCurrentAngle(index);
            },
            handleTouchMove(e, index) { //触摸移动
                if (this.touchStatus) {
                    let touch = e.targetTouches[0];
                    this.endY = touch.pageY;
                    let MoveY = (this.endY - this.startY) / 1;
                    let MoveAngle = this.startAngle + MoveY * -1 / this.perAngleH;
                    this.setListTransform(index, MoveAngle);
                }   
            },
            handleTouchEnd(e, index) { //触摸结束
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
                this.updateNumbers();
            },
            updateNumbers() { //更新父组件号码
                this.$emit("update", this.numbers.join(""));
            }
        },
    }
</script>

<style scoped>
.slot-machines {
    display: flex;
    align-items: center;
    background: #ddd;
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
    margin-right: 2px;
}
.slot-machines .list:last-child {
    margin-right: 0;
}
.slot-machines .item{
    position: absolute;
    left: 0;
    top: 50%;
    border: 0px solid rgba(0, 0, 0, 0);
    box-sizing: border-box;
    background-image: url("../assets/images/numStrap.png");
    background-repeat: no-repeat;
    background-size: 100% 1000%;
    background-position: 0 0;
    background-color: #fff;
    width: 100%;
    height: 90px;
    margin-top: -45px;
    /* border-left: 1px solid #f17e7e;
    border-right: 1px solid #f17e7e; */
}
</style>