<template>
    <div class="library">
        <template v-for="sprite in this.list"  >
            <!--Moveable v-if="draggingId === sprite.id" :key="sprite.id" :id="sprite.id"
                class="sprite moveable" 
                @drag="handleDrag"
                @dragEnd="dragEnd"
                v-bind="moveable">
                <svg-box :ref="'sprite-' + sprite.id" :sourse="sprite.sourse"></svg-box>
            </Moveable -->
            <svg-box 
                :id="sprite.id" 
                :sourse="sprite.sourse" 
                :ref="'sprite-' + sprite.id" 
                class="sprite" 
                :key="sprite.id" 
                v-on:select="onSelectItem"
            ></svg-box>
        </template>
    </div>
</template>

<script>

import SvgBox from './SvgBox.vue';
import Moveable from 'vue-moveable';
export default {
    name: 'Library',
    components: { Moveable, SvgBox },
    props: {
        list: {
            type: Array,
            default: () => []
        }
    },
    methods: {
        onSelectItem(payload) {
            this.$emit('select', payload);
        }
    }
}
</script>

<style lang="scss">
.library {
    width: 110px;
    height: 100%;
    overflow: visible;
    position: absolute;
}
</style>