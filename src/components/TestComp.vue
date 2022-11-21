<template>
    <div class="root">
        <div class="title">
            Список пользователей
        </div>
        <div class="search">
            <div class="search__field">
                <img src="../assets/img/icons_search.svg" alt="">
                <input type="text" :placeholder="placeholder" v-model="search">
            </div>
            <div class="clearfilters">
                <img src="../assets/img/clean.svg" alt="">
                <button @click="clearFilter()" type="button">Очистить фильтр</button>    
            </div>
        </div>
        <div class="userstable">
            <div class="variants-sort">
                Сортировка:
                <button @click="sortUsers('date')" :class="['default', { 'active': curSortActive == 'date' }]">Дата регистрации</button>
                <button @click="sortUsers('rating')" :class="['default', { 'active': curSortActive == 'rating' }]">Рейтинг</button>
            </div>
            <div class="table">
                <div class="table__flex">
                    <div class="table__flex-item" >Имя пользователя</div>
                    <div class="table__flex-item" >E-mail</div>
                    <div class="table__flex-item" >Дата регистрации</div>
                    <div class="table__flex-item" >Рейтинг</div>    
                    <div class="table__flex-item-icon"></div>
                </div>
                <div class="table__flex" v-for="(user, index) in filterByNameEmail" :key="index">
                    <div class="table__flex-item user" >{{user.username}}</div>
                    <div class="table__flex-item date" >{{user.email}}</div>
                    <div class="table__flex-item date" >{{user.parsed_date}}</div>
                    <div class="table__flex-item date" >{{user.rating}} </div>
                    <div class="table__flex-item-icon"> 
                        <img src="../assets/img/cancel.svg" alt="" @click="toggleModal(index)"> 
                     </div>
                </div>
            </div>
        </div>
        <div class="modal" v-show="isModalOpen">
            <div class="modal-content">
                <div class="modal-content-confirm">Вы уверены, что хотите удалить пользователя?</div>
                    <div class="modal-content-confirm-buttons">
                        <div class="modal-content-confirm-buttons-yes"><button  @click="removeUser()">Да</button></div>
                        <div class="modal-content-confirm-buttons-no"><button  @click="toggleModal()">Нет</button></div>
                    </div>
                  
                
            </div>
        </div>
    </div>
</template>

<script>

import axios from 'axios';
export default {
    name: 'TestComp',
    data() {
        return{
            isModalOpen: false,
            placeholder: 'Поиск по имени или e-mail',
            users: [],
            sortDirection: false,
            search: "",
            indexToDelete: null,
            curPage: 0,
            pageCount: 1,
            perPage: 5,
            curSortActive: ""
        }
    },
    mounted: function() {
        this.getUsers();
    },
    computed: {
        filterByNameEmail() {
            return this.users.filter((user) =>
                user.username.toLowerCase().includes(this.search.toLowerCase())
                || user.email.toLowerCase().includes(this.search.toLowerCase()));
        }
    },
    methods: {
        toggleModal(index = null){
            this.isModalOpen = !this.isModalOpen;
            this.indexToDelete = index;
        },
        removeUser() {
            this.users.splice(this.indexToDelete, 1);
            this.indexToDelete = null;
            this.isModalOpen = !this.isModalOpen;
        },
        getUsers(){
            let self = this;

            axios.get('https://5ebbb8e5f2cfeb001697d05c.mockapi.io/users')
                .then(function (response) {

                    self.users = response.data.map((user) => {

                        let date = user.registration_date ? new Date(user.registration_date) : new Date();
                        let day = date.getUTCDate().toString().length == 1 ? "0" + date.getUTCDate() : date.getUTCDate();
                        let month = (date.getUTCMonth() + 1).toString().length == 1 ? "0" + (date.getUTCMonth() + 1) : date.getUTCMonth() + 1;
                        let parsedDate = `${day}.${month}.${date.getUTCFullYear().toString().split("").splice(2, 2).join("")}`;

                        return {
                            id: user.id,
                            email: user.email,
                            rating: user.rating,
                            username: user.username,
                            registration_date: user.registration_date,
                            parsed_date: parsedDate
                        }
                    });

                    
                })
                .catch(function (error) {
                    console.log(error);
                });
        },
        clearFilter() {
            this.search = "";
        },
        changeDirection() {
            this.sortDirection = !this.sortDirection;
        },
        sortUsers(type) {
            switch (type) {
                case "rating":
                    this.curSortActive = "rating";
                    this.users.sort((a, b) => {
                        if (a.rating > b.rating) {
                            return this.sortDirection ? 1 : -1;
                        } else if (a.rating < b.rating) {
                            return this.sortDirection ? -1 : 1;
                        }
                        return 0;
                    });
                    break;
                default:
                    this.curSortActive = "date";
                    this.users.sort((a, b) => {
                        if (new Date(a.registration_date) > new Date(b.registration_date)) {
                             return this.sortDirection ? 1 : -1;
                        } else if (new Date(a.registration_date) < new Date(b.registration_date)) {
                            return this.sortDirection ? -1 : 1;
                        }
                        return 0;
                    });
                    break;
            }

            this.changeDirection();
        }
    }
}

</script>