<template>
    <b-container fluid="lg">

        <main v-if="users.length" class="list">
            <b-row align-h="center" align-v="center">
                <b-col lg="8" v-for="user in usersList" :key="user.id">
                    <b-card
                        @click="showPosts(user)"
                        v-b-modal.modal-center
                        :title="user.name"
                        bg-variant="dark" 
                        text-variant="white" 
                        class="text-center"
                    />
                </b-col>
            </b-row>


            <b-button 
                align-h="center"
                variant="success"
                pill
                v-if="renderShowMoreButtonUsers"
                @click="showMore('USERS')"
            >
                Mostrar Mais
            </b-button>
        </main>

        <!-- // TODO -->
        <p v-if="errors">Deu erro!</p>

        <b-modal
            ref="v-modal"
            id="modal-center"
            centered
            :title="selectedUserName"
            ok-only
            @hidden="hideModal"
            @ok="hideModal"
        >
            <div v-for="post in postItems" :key="post.id">
                <h2>
                    {{ post.title }}
                </h2>

                <p>
                    {{ post.body }}
                </p>

                <b-button @click="hidePost(post.id)">
                    Ocultar
                </b-button>
            </div>

            <b-button
                v-if="renderShowMoreButtonPosts"
                @click="showMore('POSTS')"
                pill
                variant="success"
            >
                Mostrar Mais
            </b-button>

        </b-modal>

    </b-container>
</template>

<script>
import axios from 'axios';

export default {
    name: '',
    data() {
        return {
            users: [],
            posts: [],
            hiddenPostsIds: [],
            selectedUser: null,
            totalUsers: 3,
            totalPosts: 3,
            showLoader: false,
            errors: false,
        };
    },
    computed: {
        usersList() {
            return this.users.filter((user,i) => this.isWithinRange(i, this.totalUsers));
        },
        postItems() {
            return this.posts.filter((post,i) => {
                if (this.isWithinRange(i, this.totalPosts) && !this.isPostHidden(post)) {
                    return post;
                }
            });
        },
        renderShowMoreButtonUsers() {
            return this.totalUsers <= this.users.length;
        },
        renderShowMoreButtonPosts() {
            return this.totalPosts <= this.posts.length;
        },
        selectedUserName() {
            return this.selectedUser?.name || ``;
        }
    },
    methods: {
        // Fetch
        async getUsers() {
            await axios.get('https://jsonplaceholder.typicode.com/users')
                .then(({ data }) => this.users = data)
                .catch(err => {
                    console.error(err);
                    this.errors = true;
                });
        },
        async getUserPosts(userId) {
            const url = `https://jsonplaceholder.typicode.com/posts?userId=${userId}`;

            await axios.get(url)
                .then(({ data }) => this.posts = data)
                .catch(err => console.error(err));
        },
        // Modal
        showModal() {
            this.hiddenPostsIds = [];
            this.totalPosts = 3;

            this.$refs['v-modal'].show();
        },
        hideModal() {
            this.$refs['v-modal'].hide();
        },
        // Loader
        toggleLoader() {
            this.showLoader = !this.showLoader;
        },
        // Post
        async showPosts(user) {
            this.selectedUser = user;
            this.toggleLoader();

            await this.getUserPosts(user.id);

            this.showModal();
            this.toggleLoader();
        },
        hidePost(postId) {
            const hiddenPosts = this.hiddenPostsIds;
            hiddenPosts.push(postId);

            this.hiddenPostsIds = [...new Set(hiddenPosts)];
        },
        isPostHidden(post){
            return this.hiddenPostsIds.some(id => id === post.id);
        },
        // Helpers
        showMore(type) {
            if (type === 'USERS') {
                this.totalUsers += 3;
                return;
            }
            
            this.totalPosts += 3;
        },
        isWithinRange(index, total) {
            return (index + 1) <= total;
        },
    },
    async mounted() {
        this.toggleLoader();

        await this.getUsers();
   
        this.toggleLoader();
    }
};
</script>
