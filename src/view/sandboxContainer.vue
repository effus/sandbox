<template>
    <section class="container">
        <header>
            <div class="controls">
                <button @click="showNewSpriteDialog">New sprite</button>
                <button @click="onClearSkatch">Clear</button>
                <button @click="onDeleteSelected">Delete selected</button>
            </div>
            <div class="title">Sandbox <span>effus/sandbox</span></div>
            <div class="reserved"></div>
        </header>
        <div class="workarea">
            <library :list="library" v-on:select="onLibSelect" :class="{atTop: atTop==='library'}"></library>
            <sketch 
                :sprites="sprites" :class="{atTop: atTop==='sketch'}" 
                :texts="texts"
                :is-active="atTop==='sketch'"
                v-on:select="onSelectSprite"
                v-on:put-text="onPutText"
                v-on:remove-text="onRemoveText"
                v-on:change-position="onChangeSpritePosition"></sketch>
        </div>
        
        <new-sprite 
            v-if="newSprite.isVisible" 
            v-on:submit="onSaveSprite"></new-sprite>
    </section>
</template>

<script>
import SvgBox from '../components/SvgBox.vue';
import Moveable from 'vue-moveable';
import Vue from 'vue';
import ClickOutside from 'vue-click-outside'
import Library from '../components/Library.vue';
import NewSprite from '../components/NewSprite.vue';
import Sketch from '../components/Sketch.vue';


export default {
    name: 'sandboxContainer',
    components: { SvgBox, Moveable, Library, NewSprite, Sketch },

    data: () => {
        return {
            library: [],
            sprites: [],
            texts: [],
            newSprite: {
                isVisible: false
            },
            atTop: 'library'
        }
    },
    mounted() {
        this.popupItem = this.$el;
    },
    methods: {
        getActiveAreaHeight() {
            return (window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight) - 100;
        },
        getLibraryCellSize() {
            return {
                width: 110,
                height: 110
            }
        },
        getLibPosition(libIndex) { // 1
            const rowsInColumn = Math.floor(this.getActiveAreaHeight() / this.getLibraryCellSize().height); // 6
            const column = Math.floor(libIndex / rowsInColumn);
            const row = libIndex - rowsInColumn * column;
            return {
                top: this.getLibraryCellSize().height * row,
                left: this.getLibraryCellSize().width * column
            }
        },
        onSaveSprite(payload) {
            const libPosition = this.getLibPosition(this.library.length);
            payload = Object.assign({}, payload, {
                libIndex: this.library.length,
                top: libPosition.top,
                left: libPosition.left, 
                selected: false
            });
            this.library.push(payload);
            this.newSprite.isVisible = false;
        },
        async showNewSpriteDialog() {
            this.newSprite.isVisible = true;
        },
        async onLibSelect(payload) {
            this.disableSkatchSprites();
            await this.$nextTick();
            this.newSprite.isVisible = false;
            const libPosition = this.getLibPosition(payload.libIndex);
            this.sprites.push(Object.assign({}, payload, {
                libId: payload.id,
                top: libPosition.top,
                left: libPosition.left, 
                selected: true
            }));
            this.atTop = 'sketch';
        },
        onChangeSpritePosition(payload) {
            this.sprites = this.sprites.map((item) => {
                if (item.id === payload.sprite.id) {
                    item.top = payload.top;
                    item.left = payload.left;
                    item.selected = false;
                    item.id = Math.random();
                }
                return item;
            });
            this.atTop = 'library';
        },
        onSelectSprite(payload) {
            this.newSprite.isVisible = false;
            this.sprites = this.sprites.map((item) => {
                if (item.id === payload.id) {
                    item.selected = true;
                }
                return item;
            });
            this.disableLibSprites();
            this.atTop = 'library';
        },
        onClearSkatch() {
            this.sprites = [];
            this.atTop = 'library';
            this.newSprite.isVisible = false;
        },
        disableLibSprites() {
            this.library = this.library.map((item) => {
                item.selected = false;
                return item;
            });
        },
        disableSkatchSprites() {
            this.sprites = this.sprites.map((item) => {
                item.selected = false;
                return item;
            });
        },
        onDeleteSelected() {
            this.library = this.library.filter((item) => {
                return item.selected === false;
            });
            this.library = this.library.map((item, index) => {
                const libPosition = this.getLibPosition(index);
                item.top = libPosition.top;
                item.left = libPosition.left;
                return item;
            });
            this.sprites = this.sprites.filter((item) => {
                return item.selected === false;
            });
            this.atTop = 'library';
        },
        onPutText(payload) {
            this.texts.push(payload);
        },
        onRemoveText(payload) {
            this.texts.splice(payload.index, 1);
        }
    },
    directives: {
        ClickOutside
    }
}
</script>

<style lang="scss">
$activeColor: #ffb100;
button {
    background-color: #00000091;
    color: $activeColor;
    padding: 5px 10px;
    box-shadow: 1px 1px 1px #000;
    border-radius: 3px;
    border: 1px solid $activeColor;
    margin-right: 2px;
    &:hover {
        border-color: lighten($color: $activeColor, $amount: 15);
        color: lighten($color: $activeColor, $amount: 15);
        outline: none;
    }
    &:focus {
        outline: none;
    }
    &:visited {
        border-color: darken($color: $activeColor, $amount: 15);
        color: darken($color: $activeColor, $amount: 15);
    }
    &:active {
        border-color: darken($color: $activeColor, $amount: 5);
        color: darken($color: $activeColor, $amount: 5);
        box-shadow: 0px 0px 3px $activeColor;
    }
}
.container {
    display: grid;
    grid-template-rows: 45px 20fr;
    grid-template-columns: 1fr;
    height: inherit;
    header {
        display: grid;
        grid-template-columns: 5fr 2fr;
        padding: 3px;
        height: inherit;
        align-items: center;
        grid-column-start: 1;
        grid-column-end: 3;
        .title {
            font-size: 18px;
            color: $activeColor;
            line-height: 18px;
            text-align: right;
            padding-right: 10px;
            span {
                font-size: 10px;
                color: darken($activeColor, $amount: 15)
            }
        }
        .controls {
            padding-left: 10px;
        }
    }
    .workarea {
        position: relative;
        .library {
            z-index: 100;
        }
        .sketch {
            z-index: 150;
        }
        .atTop {
            z-index: 300;
        }
    }
    .sprite {
        width: 100px;
        height: 100px;
        &.moveable {
            position: absolute;
        }
    }
       
}
</style>