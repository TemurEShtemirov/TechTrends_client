<template>
    <div>
        <header>
            <ul class="nav justify-content-center row">
                <li class="nav-item row col-12">
                    <a class="btn btn-primary mb-4 col-12" aria-current="page" href="#" @click="openAddPostModal">Add
                        Post</a>
                </li>
            </ul>
        </header>

        <h1>Blog Post Management</h1>

        <h2>Get All Posts</h2>

        <form class="d-flex">
            <input v-model="searchQuery" type="search" class="form-control" placeholder="Search by Title or Content">
            <button @click="clearSearch" class="btn btn-primary" type="button">Clear</button>

        </form>

        <div class="card-deck row container">
            <div v-for="post in posts" :key="post.id" class="card col-sm-12 col-md-6 col-lg-4 col-xl-3 col-xxl-3 m-3"
                style="width: 18rem;">
                <div class="card-body row">
                    <h5 class="card-title col-12 m-1">{{ post.title }}</h5>
                    <button @click="viewPostInfo(post)" class="btn btn-primary col-12 m-1">See Info</button>
                    <button @click="editPost(post)" class="btn btn-secondary col-12 m-1">Edit</button>
                    <button @click="deletePost(post)" class="btn btn-danger col-12 m-1">Delete</button>
                </div>
            </div>
        </div>

        <div class="modal" v-if="selectedPost" style="display: block;">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ selectedPost.title }}</h5>
                        <button type="button" class="close" @click="selectedPost = null">
                            <span>&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <p>{{ selectedPost.content }}</p>
                        <img v-if="selectedPost.image" :src="`http://localhost:8080/posts/` + selectedPost.image"
                            alt="Post Image" />
                    </div>
                </div>
            </div>
        </div>

        <div class="modal" v-if="editingPost" style="display: block;">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Edit Post</h5>
                        <button type="button" class="close" @click="editingPost = null">
                            <span>&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <form @submit.prevent="updatePost">
                            <div class="form-group">
                                <label for="editTitle">Title:</label>
                                <input type="text" v-model="editedPost.title" class="form-control" id="editTitle">
                            </div>
                            <div class="form-group">
                                <label for="editContent">Content:</label>
                                <textarea v-model="editedPost.content" class="form-control" id="editContent"></textarea>
                            </div>
                            <div class="form-group">
                                <label for="editImage">Image:</label>
                                <input type="file" @change="handleImageUpload" class="form-control" id="editImage"
                                    accept="image/png, image/jpeg" />
                            </div>
                            <button type="submit" class="btn btn-primary">Save Changes</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>

        <div class="modal" v-if="showAddPostModal" style="display: block;">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Add Post</h5>
                        <button type="button" class="close" @click="closeAddPostModal">
                            <span>&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <form @submit.prevent="addPost">
                            <div class="form-group">
                                <label for="addTitle">Title:</label>
                                <input type="text" v-model="newPost.title" class="form-control" id="addTitle">
                            </div>
                            <div class="form-group">
                                <label for="addContent">Content:</label>
                                <textarea v-model="newPost.content" class="form-control" id="addContent"></textarea>
                            </div>
                            <div class="form-group">
                                <label for="addImage">Image:</label>
                                <input type="file" @change="handleImageUpload" class="form-control" id="addImage"
                                    accept="image/png, image/jpeg" />
                            </div>
                            <button type="submit" class="btn btn-primary">Add Post</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const apiUrl = 'http://localhost:8080/posts/';

export default {
    data() {
        return {
            posts: [],
            selectedPost: null,
            editingPost: null,
            editedPost: {
                title: '',
                content: ''
            },
            showAddPostModal: false,
            newPost: {
                title: '',
                content: ''
            },
            imageFile: null,
            searchQuery: '', 
             clearSearch() {
                this.searchQuery = '';
            },
        };
    },
    computed: {
        filteredPosts() {
            return this.posts.filter(post => {
                const query = this.searchQuery.toLowerCase();
                return post.title.toLowerCase().includes(query) || post.content.toLowerCase().includes(query);
            });
        },
    },
    async mounted() {
        await this.getAllPosts();
    },
    methods: {
        async getAllPosts() {
            try {
                const response = await fetch(apiUrl);
                if (!response.ok) {
                    throw new Error('Network response was not ok');
                }

                const data = await response.json();
                this.posts = data.posts;
            } catch (error) {
                console.error('Error fetching posts:', error);
            }
        },
        viewPostInfo(post) {
            this.selectedPost = post;
        },
        editPost(post) {
            this.editedPost = { ...post };
            this.editingPost = post;
        },
        async updatePost() {
            try {
                const formData = new FormData();
                formData.append('title', this.editedPost.title);
                formData.append('content', this.editedPost.content);
                if (this.imageFile) {
                    formData.append('image', this.imageFile);
                }

                const response = await fetch(`${apiUrl}/${this.editedPost.id}`, {
                    method: 'PUT',
                    body: formData
                });

                if (!response.ok) {
                    throw new Error('Error updating post');
                }

                this.editingPost = null;
                this.editedPost = {
                    title: '',
                    content: ''
                };
                this.imageFile = null;
                await this.getAllPosts();
            } catch (error) {
                console.error('Error updating post:', error);
            }
        },
        async deletePost(post) {
            try {
                const postId = post.id;

                const response = await fetch(`${apiUrl}/${postId}`, {
                    method: 'DELETE'
                });

                if (!response.ok) {
                    throw new Error('Error deleting post');
                }

                this.posts = this.posts.filter(p => p.id !== postId);
            } catch (error) {
                console.error('Error deleting post:', error);
            }
        },

        openAddPostModal() {
            this.showAddPostModal = true;
        },

        closeAddPostModal() {
            this.showAddPostModal = false;
            this.newPost = {
                title: '',
                content: ''
            };
            this.imageFile = null;
        },

        async addPost() {
            try {
                const formData = new FormData();
                formData.append('title', this.newPost.title);
                formData.append('content', this.newPost.content);
                if (this.imageFile) {
                    formData.append('image', this.imageFile);
                }

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    body: formData
                });

                if (!response.ok) {
                    throw new Error('Network response was not ok');
                }

                this.closeAddPostModal();
                await this.getAllPosts();
            } catch (error) {
                console.error('Error adding post:', error);
            }
        },

        handleImageUpload(event) {
            this.imageFile = event.target.files[0];
        }
    }
};
</script>


<style>
/* Add any necessary styles */
</style>
