<template>
    <div class="new-object">
        <div class="toolbar">
            <button type="button" @click="saveSprite">Save</button>
            <button type="button" @click="viewportClear">Clear</button>
            <div class="colors right">
                <button class="btn-bg white" @click="setStrokeColor('#ffffff')"></button>
                <button class="btn-bg orange" @click="setStrokeColor('#ffb100')"></button>
                <button class="btn-bg red" @click="setStrokeColor('#ff0000')"></button>
                <button class="btn-bg blue" @click="setStrokeColor('#026cff')"></button>
                <button class="btn-bg green" @click="setStrokeColor('#00bc00')"></button>
            </div>
            <button type="button" @click="undo" v-if="this.history.length > 0">Undo ({{this.history.length}})</button>
            <button class="right">Close</button>
        </div>
        <div class="viewport" v-if="flagClear"></div>
    </div>
</template>

<script>
import pencil from 'svg-pencil';
export default {
    name: 'NewSprite',
    props: {

    },
    data() {
        return {
            stroke: '#ffb100',
            pencil: null,
            flagClear: true,
            history: []
        }
    },
    mounted() {
        this.mountSvg();
    },
    methods: {
        mountSvg() {
            this.pencil = pencil({
                stroke: this.stroke,
                strokeWidth: 5
            });
            this.pencil.appendTo('.new-object .viewport');
            this.pencil.on('points', this.onDrawPath);
        },
        async saveSprite() {
            var svgSource = this.pencil.toSource();
            this.flagClear = false;
            await this.$nextTick();
            this.flagClear = true;
            this.$emit('submit', {
                id: (new Date()).getTime(),
                sourse: svgSource
            });
            this.history = [];
        },
        setStrokeColor(color) {
            this.color = color;
            this.pencil.stroke = color;
        },
        async viewportClear() {
            await this.resetViewport();
            this.history = [];
            this.mountSvg();
        },
        async resetViewport() {
            this.flagClear = false;
            await this.$nextTick();
            this.flagClear = true;
            await this.$nextTick();
        },
        async onDrawPath() {
            await this.$nextTick();
            var svgSource = this.pencil.toSource();
            this.history.push(svgSource);
        },
        async undo() {
            await this.resetViewport();
            await this.$nextTick();
            if (this.history.length > 0) {
                this.history.splice(this.history.length-1);
                this.pencil = pencil.parse(this.history[this.history.length - 1]);
            }
            await this.$nextTick();
            this.pencil.appendTo('.new-object .viewport');
            this.setStrokeColor(this.color);
            this.pencil.strokeWidth = 5;
            this.pencil.on('points', this.onDrawPath);
            this.pencil.enable();
        }
    }
}
</script>

<style lang="scss">
.new-object {
    position: absolute;
    z-index: 9999;
    top: 6%;
    left: 3%;
    width:500px;
    height: 535px;;
    background-color: #222222;
    border: 1px solid #333333;
    box-shadow: 1px 1px 5px #000;
    user-select: none;
    display: grid;
    grid-template-rows: 1fr 20fr;
    .toolbar {
        padding: 2px;
        display: flex;
        button {
            margin-right: 3px;
        }
        .right {
            margin-left: auto;
        }
        .btn-bg {
            height: 100%;
            width: 30px;
            &.white {
                background-color: white;   
            }
            &.red {
                background-color: red;   
            }
            &.orange {
                background-color: orange;   
            }
            &.blue {
                background-color: #026cff;   
            }
            &.green {
                background-color: #00bc00;   
            }
        }
    }
    .viewport {
        width: 500px;
        height: 500px;
        background-color: #1a1a1a;  
        svg {
            width: 500px;
            height: 500px;
        }
    }
}
</style>