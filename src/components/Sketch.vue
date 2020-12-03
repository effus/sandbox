<template>
    <div class="sketch" :class="{active: isActive}" ref="sketch">
        <template v-for="sprite in this.sprites">
            <Moveable v-if="sprite.selected" :key="sprite.id" :id="sprite.id"
                class="sprite-wrapper"
                :style="stylePosition(sprite)"
                @drag="handleDrag"
                @dragEnd="handleDragEnd(sprite)"
                v-bind="moveable">
                <svg-box class="sprite" :ref="'sprite-' + sprite.id" :sourse="sprite.sourse"></svg-box>
            </Moveable>
            <div v-else class="sprite-wrapper placed" :style="stylePosition(sprite)" :key="sprite.id" 
                :id="sprite.id" >
                <svg-box 
                    :sourse="sprite.sourse" 
                    :ref="'sprite-' + sprite.id" 
                    class="sprite" 
                    v-on:select="onSelectItem(sprite)"
                ></svg-box>
            </div>
        </template>
    </div>
</template>

<script>
import SvgBox from './SvgBox.vue';
import Moveable from 'vue-moveable';
export default {
    name: 'Sketch',
    components: { Moveable, SvgBox },
    props: {
        sprites: {
            type: Array,
            default: () => []
        },
        isActive: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            moveable: {
                draggable: true,
                throttleDrag: 0,
                resizable: false,
                throttleResize: 1,
                keepRatio: true,
                scalable: false,
                throttleScale: 0,
                rotatable: false,
                throttleRotate: 0,
            },
            drag: {
                left: null,
                top: null
            }
        }
    },
    methods: {
        handleDrag({ target, left, top }) {
            target.style.left = `${left}px`;
            target.style.top = `${top}px`;
            this.drag.left = left;
            this.drag.top = top;
            //console.log('handleDrag', target, left, top);
        },
        handleDragEnd(payload) {
            this.$emit('change-position', {
                sprite: payload,
                top: this.drag.top,
                left: this.drag.left
            });
        },
        stylePosition(sprite) {
            return 'left: ' + sprite.left + 'px; top: ' + sprite.top + 'px;';
        },
        onSelectItem(payload) {
            this.$emit('select', payload);
        }
    }
}
</script>

<style lang="scss">
.sketch {
    position: absolute;
    top: 0;
    width: 100%;
    bottom: 0;
    box-sizing: border-box;
    .sprite-wrapper {
        width: 110px;
        height: 110px;
        position: absolute;
    }
    &.active {
        background-color: #cccccc0f;
    }
}
</style>