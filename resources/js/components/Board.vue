<template>
    <div class="container">
        <div v-if="user">
            <h3>{{ board.board.name }}</h3>
            <b-button size="sm" v-if="!category_panel" variant="outline-primary" @click="show_category_panel"><i class="fas fa-plus"></i> Category</b-button>
            <b-input-group v-if="category_panel" class="col-5 mx-auto">
                <b-form-input @keydown.enter="add_category()" v-model="category_name" type="text" class="form-control" placeholder="Enter your new category name"></b-form-input>
                <b-input-group-append>
                <b-button variant="outline-primary" @click="add_category()">Add new category</b-button>
                </b-input-group-append>
            </b-input-group>

            <div class="row px-3">
                <Category v-for="category in board.categories" :key="category.id" :thisCategory="category" :links="board.links" @del_category="delete_category(category.id)"/>
            </div>
        </div>
        <div v-else>
            <h3>Vous n'êtes pas connecté</h3>
        </div>
    </div>   
</template>

<script>
// import boards from '../json/boards.json'

import Category from './Category.vue'

    export default {
        name: 'Board',
        components: {
            Category
        },
        props: {
            
        },
        data() {
            return {
                category_panel: false,
                // category_name: '',
                user: null,
                board: {
                    board: null,
                    categories: [],
                    links: []
                },
            }
        },
        created(){
            this.get_currentUser()
        },
        methods: {
            show_category_panel(){
                this.category_panel = !this.category_panel
            },
            get_currentUser(){
                let data = {
                    'user_id': currentUser,
                }
                window.axios.post(`api/user`, {data})
                    .then( response => {
                        this.user = response.data.user
                        this.get_board(response.data.user.current_board)
                    })
                    .catch( err => {
                        this.user = null;
                    })
            },
            get_board(id){
                let data = {
                    'board_id': id
                }
                window.axios.post(`api/categories/get`, {data})
                    .then( response => {
                        this.board.board = response.data.board
                        this.board.categories = response.data.categories
                    }, e => console.log(e) )
                    // .then( () => this.board.categories.map( x => this.category_ids.push(x.id)), e => console.log(e))
                    .then( () => {
                        let data = {
                            'ids': this.board.categories.map( x => x.id)
                        }
                        if (data.ids.length > 0) {
                            window.axios.post('api/links/get', {data})
                            .then( response => this.board.links = response.data.links, e => console.log(e) )
                        }
                    })
            },
            add_category() {

                if(this.category_name.length > 0) {
                    let data = {
                        'title': this.category_name,
                        'board_id': this.user.current_board
                    }
                    console.log(data)
                    // ajax call to the api route
                    window.axios.post('api/categories/store', {data})
                    .then(response => {
                        this.board.categories = response.data.categories;
                    }, e => console.log(e));
                    // On remet les inputs à zéro
                    this.category_name = ''
                    this.category_panel = false
                }
            },
    		delete_category(id) {

       			window.axios.post('api/categories/destroy', {id})
                .then( response => {
                    this.board.categories = this.board.categories.filter( x => x.id != id) 
                })
                .catch(e=> console.log(e.response));
    		}
        }
    }

</script>


<style>

</style>