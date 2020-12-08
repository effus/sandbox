<template>
    <div class="sketch"
        @keyup.delete="onPressDelete"
        :class="{active: isActive}" 
        ref="sketch" 
        v-on:click="onClickSketch" 
        v-on:dblclick="onDoubleClick">
        <template v-for="sprite in this.sprites">
            <Moveable v-if="sprite.selected" :key="sprite.id" :id="sprite.id"
                class="sprite-wrapper"
                :style="stylePosition(sprite)"
                @drag="handleDrag"
                @dragEnd="handleDragEnd(sprite)"
                v-bind="moveable">
                <svg-box class="sprite" :ref="'sprite-' + sprite.id" :sourse="sprite.sourse"></svg-box>
            </Moveable>
            <div v-else class="sprite-wrapper placed" 
                :style="stylePosition(sprite)" 
                :key="sprite.id" 
                :id="sprite.id"
                @click.stop="() => true"
                >
                <svg-box 
                    :sourse="sprite.sourse" 
                    :ref="'sprite-' + sprite.id" 
                    class="sprite" 
                    v-on:select="onSelectItem(sprite)"
                ></svg-box>
            </div>
        </template>
        <template v-for="(textSprite, index) in this.texts">
            <Moveable v-if="textSprite.selected" :key="textSprite.id" :id="textSprite.id"
                class="text-sprite"
                :style="getTextSpritePosition(textSprite)"
                @drag="handleDrag"
                @dragEnd="handleTextDragEnd(index)"
                v-bind="moveable">
                {{textSprite.text}}
            </Moveable>
            <div v-else class="text-sprite" 
                @click.stop="onTextClick(index)" 
                v-on:dblclick.stop="onTextDblClick(index)" 
                :style="getTextSpritePosition(textSprite)" 
                :key="index">{{textSprite.text}}</div>
        </template>
        <div class="text-input-wrapper" v-if="text.isVisible" :style="textInputPosition">
            <input type="text" ref="inputText" v-on:blur="setText" v-model="text.value" />
        </div>
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
        texts: {
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
            },
            text: {
                isVisible: false,
                position: {
                    left: 0,
                    right: 0
                },
                value: ''
            }
        }
    },
    mounted() {
        document.addEventListener("keydown", this.onDocumentKeydown);
    },
    destroyed() {
        document.removeEventListener("keydown");
    },
    methods: {
        handleDrag({ target, left, top }) {
            target.style.left = `${left}px`;
            target.style.top = `${top}px`;
            this.drag.left = left;
            this.drag.top = top;
        },
        handleDragEnd(payload) {
            this.$emit('change-sprite-position', {
                sprite: payload,
                top: this.drag.top,
                left: this.drag.left
            });
        },
        handleTextDragEnd(payload) {
            this.$emit('change-text-position', {
                index: payload,
                top: this.drag.top,
                left: this.drag.left
            });
        },
        stylePosition(sprite) {
            return 'left: ' + sprite.left + 'px; top: ' + sprite.top + 'px;';
        },
        onSelectItem(payload) {
            console.log('on-select-item', payload);
            this.$emit('select-sprite', payload);
        },
        onClickSketch() {
            this.$emit('deselect-text');
            this.$emit('deselect-sprite');
            console.log('onClickSketch');
        },
        async onDoubleClick(event) {
            if (this.text.isVisible) {
                return;
            }
            this.text.value = '';
            this.text.isVisible = true;
            this.text.position.left = event.clientX - 8;
            this.text.position.top = event.offsetY - 10;
            event.preventDefault();
            await this.$nextTick();
            this.$refs.inputText.focus();
        },
        setText(event) {
            this.text.isVisible = false;
            if (!this.text.value) {
                return;
            }
            this.$emit('put-text', {
                text: this.text.value,
                top: this.text.position.top + 7,
                left: this.text.position.left + 8
            });
            event.preventDefault();
        },
        getTextSpritePosition(textSprite) {
            return 'left: ' + textSprite.left + 'px; top: ' + textSprite.top + 'px;';
        },
        onTextClick(index) {
            this.$emit('select-text', {
                index: index
            });
        },
        onPressDelete() {
            let selected = this.texts.filter((item, i) => {
                item.index = i;
                return item.selected === true;
            });
            if (selected.length === 1 && !this.text.isVisible) {
                this.$emit('remove-text', {index: selected[0].index});
            }
            console.log('remove-text', selected);
            selected = this.sprites.filter((item, i) => {
                item.index = i;
                return item.selected === true;
            });
             if (selected.length === 1 && !this.text.isVisible) {
                this.$emit('remove-sprite', {index: selected[0].index});
            }
            console.log('remove-sprite', selected);
        },
        handleSelectedTextClickOutside(index) {
            this.$emit('deselect-text', {
                index: index
            });
        },
        onDocumentKeydown(event) {
            if (event.keyCode === 46) {
                console.log('onDocumentKeydown', 46);
                this.onPressDelete();
            }
        }
    },
    computed: {
        textInputPosition() {
            return 'left: ' + this.text.position.left + 'px; top: ' + this.text.position.top + 'px;';
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
    -webkit-user-select: none;  /* Chrome all / Safari all */
    -moz-user-select: none;     /* Firefox all */
    -ms-user-select: none;      /* IE 10+ */
    user-select: none;  
    .sprite-wrapper {
        width: 110px;
        height: 110px;
        position: absolute;
    }
    &.active {
        background-color: #cccccc0f;
    }
    .text-input-wrapper {
        position: absolute;
        input {
            background-color: #33333366;
            border: 0 none;
            color: #ffb100;
            padding: 8px;
            border-bottom: 1px solid #ffb100;
            font-size: 14px;
            &:focus {
                border: 0 none;
                outline: none;
                border-bottom: 1px solid #ffb100;
            }
        }
    }
    .text-sprite {
        position: absolute;
        color: #ffb100;
        font-size: 14px;
    }
}
</style>