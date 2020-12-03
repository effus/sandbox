<template>
    <div class="svg-box" @click="onClick">
        <img :src="svgFixed" alt="svg" width="100" height="100"   />
    </div>
</template>

<script>
import svgToDataURL from 'svg-to-dataurl';
export default {
    name: 'svgBox',
    props: {
        sourse: {
            type: String,
            default: ''
        },
        id: {
            type: Number|Object
        }
    },
    computed: {
        svgFixed: function() {
            let svgSource = this.sourse.replace(/<svg>/g, "<svg xmlns='http://www.w3.org/2000/svg' width='500' height='500' viewBox='0 0 500 500'>");
            return svgToDataURL(svgSource) ;
        }
    },
    methods: {
        onClick() {
            this.$emit('select', {
                sourse: this.sourse,
                id: this.id
            });
        }
    }
}
</script>

<style lang="scss">
.svg-box {
    padding: 5px;
    //border: 1px solid #fff;
    background-color: #33333370;
    border-radius: 15px;
}
</style>