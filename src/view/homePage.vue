<template>
    <div class="home-panel">
        <MapView></MapView>
    </div>
</template>

<script>
import config from '@/config.js';

//引入vuex
import { mapMutations, mapState } from 'vuex'

import MapView from "@/components/MapView";

export default {
    name: 'Home',
    props: [],
    data() {
        return {
        }
    },
    computed: {
    },
    components: {
        MapView
    },
    mounted() {

    },
    methods: {
    },
}
</script>

<style lang="scss" scope>
.home-panel{
    height: 100%;
    width:100%;
}
</style>
