<template>
    <div id="app">
        <b-modal v-model="modal">
            <h3>Описание</h3>
            {{desc.title}}
            <div style="display: flex; flex-direction: row; margin-top: 10px; overflow-y: auto">
                <div class="mr-3" v-for="(el,ind) in desc.images" v-bind:key="ind">
                    <img style="width: 100px; height: 100px; border-radius: 10px;"
                         :src="'http://188.225.18.212/img/'+el">
                </div>
            </div>
            <h3>Критерии</h3>
            <div style="margin-bottom: 10px" v-for="(el,index) in current.criteria" v-bind:key="index">
                <div style="display: flex; flex-direction: row; justify-content: space-between">
                    <div style="font-weight: 600">
                        {{el.name}}
                    </div>
                    <input type="number" v-model="el.rating"/>
                </div>
                <div style="display: flex; flex-direction: row; margin-top: 10px; overflow-y: auto">
                    <div class="mr-3" v-for="(el,ind) in el.images" v-bind:key="ind">
                        <img style="width: 100px; height: 100px; border-radius: 10px;"
                             :src="'http://188.225.18.212/img/'+el">
                    </div>
                </div>

            </div>
            <b-button class="mr-2" variant="success" @click="accept">Принять</b-button>
            <b-button class="mr-2" variant="warning" @click="back">На доработку</b-button>
            <b-button class="mr-2" variant="danger" @click="reject">Отклонить</b-button>
        </b-modal>
        <b-container>
            <b-table responsive :fields="fields" :items="data" @row-clicked="openModal"></b-table>
        </b-container>
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
                current: {},
                fields: [
                    {key: 'name', label: 'Название'},
                    {key: 'user', label: 'Пользователь'},
                    {key: 'address', label: 'Адрес'},
                    {key: 'status', label: 'Статус'},
                    {key: 'date_start', label: 'Дата'},
                ],
            }
        },
        async mounted() {
            let {data} = await this.$axios.get('http://188.225.18.212/event/-/admin');
            console.log(data)
            this.data = data;
        },
        methods: {
            openModal(row) {
                this.modal = true;
                this.current = row;
            },
            async accept() {
                console.log(this.current)
                let {data} = await this.$axios.put('http://188.225.18.212/event/' + this.current._id + '/admin', {
                    ...this.current,
                    status: 'accepted'
                });
                console.log(data)
            },
            async reject() {
                console.log(this.current)
                let {data} = await this.$axios.put('http://188.225.18.212/event/' + this.current._id + '/admin', {
                    ...this.current,
                    status: 'rejected'
                });
                console.log(data)
            },
            async back() {
                console.log(this.current)
                let {data} = await this.$axios.put('http://188.225.18.212/event/' + this.current._id + '/admin', {
                    ...this.current,
                    status: 'draft'
                });
                console.log(data)
            },

        },
        computed: {
            desc() {
                return this.current.description || {}
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
