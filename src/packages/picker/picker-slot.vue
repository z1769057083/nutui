<template>
  <div class="nut-picker-list pd-select-item">
    <div class="nut-picker-mask"></div>
    <div class="nut-picker-indicator"></div>
    <div class="nut-picker-content">
        <div class="nut-picker-list-panel" ref="list">
            <div class="nut-picker-item" v-for="(item,index) in listData "
                :key="index">{{item}}
            </div>
        </div>
    </div>
</div>
</template>
<script>
export default {
    name:'nut-picker-slot',
    props: {
        listData: {
            type: Array,
            required: true
        },
        defaultValue: {
            type: String | Number
        },
        keyIndex: {
            type: Number,
            default: 0
        },
        isUpdate: {
            type: Boolean,
            default: false
        },
    },
    data() {
        return {
            touchParams: {
                startY: 0, 
                endY: 0, 
                startTime: 0, 
                endTime: 0
            },
            transformY: 0,
            scrollDistance: 0,
            lineSpacing: 36,
            timer: null
        }
    },
    watch: {
        'isUpdate': function() {
            this.transformY = 0;
            this.modifyStatus();
        },
        'defaultValue': function() {
            this.transformY = 0;
            this.modifyStatus();
        }
    },
    methods: {
        updateTransform(value) {
            if (value) {
                this.transformY = 0;
                this.timer = setTimeout(() => {
                    this.modifyStatus(null, value);
                }, 10); 
            } 
        },
        setTransform(translateY = 0, type, time = 1000) {
            if (type === 'end') {
                this.$refs.list.style.webkitTransition = `transform ${time}ms cubic-bezier(0.19, 1, 0.22, 1)`;
                //this.$refs.list.style.transition = `transform ${time}ms cubic-bezier(0.19, 1, 0.22, 1)`;
            } else {
                this.$refs.list.style.webkitTransition = '';
                //this.$refs.list.style.transition = '';
            }
            this.$refs.list.style.webkitTransform = `translateY(${translateY}px)`;
            //this.$refs.list.style.transform = `translateY(${translateY}px)`;
            this.scrollDistance = translateY;
        },

        setMove(move, type, time) {
            let updateMove = move + this.transformY;
            if (type === 'end') {
                // 限定滚动距离
                if (updateMove > 0) {
                    updateMove = 0;
                }
                if (updateMove < -(this.listData.length - 1) * this.lineSpacing) {
                    updateMove = -(this.listData.length - 1) * this.lineSpacing;
                }

                // 设置滚动距离为lineSpacing的倍数值
                let endMove = Math.round(updateMove / this.lineSpacing) * this.lineSpacing;
                this.setTransform(endMove, type, time);
                this.setChooseValue(endMove);
            } else {
                this.setTransform(updateMove);
            }
        },

        setChooseValue(move) {
            this.$emit('chooseItem', this.listData[(Math.round(-move / this.lineSpacing))], this.keyIndex);
        },
	
	    touchStart(event) {
            event.preventDefault();

            let changedTouches = event.changedTouches[0];
            this.touchParams.startY = changedTouches.pageY;
            this.touchParams.startTime = event.timestamp || Date.now();
            this.transformY = this.scrollDistance;
        },

        touchMove(event) {
            event.preventDefault();

            let changedTouches = event.changedTouches[0];
            this.touchParams.lastY = changedTouches.pageY;
            this.touchParams.lastTime = event.timestamp || Date.now();
            let move = this.touchParams.lastY - this.touchParams.startY;

            this.setMove(move);
        },

        touchEnd(event) {
            event.preventDefault();

            let changedTouches = event.changedTouches[0];
            this.touchParams.lastY = changedTouches.pageY;
            this.touchParams.lastTime = event.timestamp || Date.now();
            let move = this.touchParams.lastY - this.touchParams.startY;

            let moveTime = this.touchParams.lastTime - this.touchParams.startTime;
            if (moveTime <= 300) {
                move = move * 2;
                moveTime = moveTime + 1000;
                this.setMove(move, 'end', moveTime);
            } else {
                this.setMove(move, 'end');
            }
        },

        modifyStatus (type, defaultValue) {
            defaultValue = defaultValue ? defaultValue : this.defaultValue;
            let index = this.listData.indexOf(defaultValue);
            let move = index === -1 ? 0 : (index * this.lineSpacing);
            type && this.setChooseValue(-move);
            this.setMove(-move);
        }
    },

    mounted() {
        this.$nextTick(() => {
            this.modifyStatus(true);
            
            // 监听
            this.$el.addEventListener('touchstart', this.touchStart);
            this.$el.addEventListener('touchmove', this.touchMove);
            this.$el.addEventListener('touchend', this.touchEnd);
        });
    },
    beforeDestroy() {
        // 移除监听
        this.$el.removeEventListener('touchstart', this.touchStart);
        this.$el.removeEventListener('touchmove', this.touchMove);
        this.$el.removeEventListener('touchend', this.touchEnd);
        clearTimeout(this.timer);
    }
}
</script>
