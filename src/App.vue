<template>
    <div id="app">
        <b-modal v-model="modal">
            <b-button variant="success" @click="accept">Принять</b-button>
        </b-modal>
        <b-table :items="data" @row-clicked="openModal"></b-table>

    </div>
</template>

<script>

    export default {
        name: 'App',
        components: {},
        data() {
            return {
                data: [],
                modal: false,
                current: {}
            }
        },
        async mounted() {
            let {data} = await this.$axios.get('http://192.168.43.108:5000/event');
            this.data = data;
        },
        methods: {
            openModal(row) {
                this.modal = true;
                this.current = row;
            },
            async accept() {
                let {data} = await this.$axios.put('http://192.168.43.108:5000/event/' + this.current._id, {status: 'accepted'});
                console.log(data)
            }
        }

    }
</script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
