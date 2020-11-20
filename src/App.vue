<template>
    <div id="app">
        <router-view/>
    </div>
</template>

<script>
export default {
    name: 'App',
    data() {
        return {

        }
    },
    components: {},
    created() {},
    mounted() {},
    methods: {}
}
</script>

<style lang="scss">
@import './assets/css/resetCss.css';
#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: $red_c;
}
html,body,#app{
    margin:0;
    padding:0;
    height:100%;
    width:100%;
}
</style>
