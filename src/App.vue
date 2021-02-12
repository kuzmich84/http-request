<template>
    <div class="container">
        <AppAlert :alert="alert" @close="alert = null"></AppAlert>
        <form class="card" @submit.prevent="createPerson">
            <h2>Работа с базой данных</h2>

            <div class="form-control">
                <label for="name">Введите имя</label>
                <input type="text" id="name" v-model.trim="name">
            </div>
            <button class="btn primary" :disabled="name.length===0">Создать человека</button>
        </form>
        <AppLoader v-if="loading"></AppLoader>
        <AppPeopleList
                v-else
                :people="people"
                @load="loadPeople"
                @remove="removePerson"
        ></AppPeopleList>
    </div>
</template>

<script>
    import AppPeopleList from "./components/AppPeopleList";
    import axios from 'axios';
    import AppAlert from "./components/AppAlert";
    import AppLoader from "./components/AppLoader";

    export default {
        data: () => ({
            name: '',
            people: [],
            alert: null,
            loading: false,
        }),
        mounted() {
            this.loadPeople();
        },
        methods: {
            async createPerson() {
                //https://vue-with-http-51891-default-rtdb.firebaseio.com/people.json

                const response = await fetch('https://vue-with-http-51891-default-rtdb.firebaseio.com/people.json', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        firstName: this.name
                    })
                });

                const firebaseData = await response.json();

                this.people.push({
                    firstName: this.name,
                    id: firebaseData.name,
                });

                this.name = '';
            },
            async loadPeople() {
                try {
                    this.loading = true;
                    const {data} = await axios.get('https://vue-with-http-51891-default-rtdb.firebaseio.com/people.json');
                    if (!data) {
                        throw new Error('Список людей пуст')
                    }
                    this.people = Object.keys(data).map(key => {
                        return {
                            id: key,
                            ...data[key]
                        }
                    })
                    this.loading = false;
                } catch (e) {
                    this.alert = {
                        type: 'danger',
                        title: 'Ошибка',
                        text: e.message,
                    }
                }


            },
            async removePerson(id) {
                try {
                    const name = this.people.find(person => person.id === id).firstName;
                    await axios.delete(`https://vue-with-http-51891-default-rtdb.firebaseio.com/people/${id}.json`);
                    this.people = this.people.filter((person) => person.id !== id);
                    this.alert = {
                        type: 'primary',
                        title: 'Успешно',
                        text: `Пользователь c именем ${name} был удален`
                    }

                } catch (e) {

                }

            }

        },
        components: {
            AppPeopleList,
            AppAlert,
            AppLoader
        }
    }
</script>

<style>

</style>
