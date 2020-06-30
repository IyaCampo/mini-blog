<template>
    <div>
        <h2>Blog Posts</h2>
        <form @submit.prevent="addPost" class="mb-3">
            <div class="form-group">
                <input
                    type="text"
                    class="form-control"
                    placeholder="Title"
                    v-model="post.title"
                />
            </div>
            <div class="form-group">
                <textarea
                    type="text"
                    class="form-control"
                    placeholder="Body"
                    v-model="post.body"
                ></textarea>
            </div>
            <input type="file" @change="onFileSelected" />
            <button
                class="btn btn-secondary float-right mb-2"
                @click="onUpload"
            >
                Upload
            </button>
            <button class="btn btn-primary btn-block mb-5">Save</button>
        </form>
        <nav aria-label="Page navigation example">
            <ul class="pagination">
                <li
                    class="page-item"
                    v-bind:class="[{ disabled: !pagination.prev_page_url }]"
                >
                    <a
                        href="#"
                        class="page-link"
                        @click="fetchPosts(pagination.prev_page_url)"
                        >Previous</a
                    >
                </li>

                <li class="page-item disabled">
                    <a href="#" class="page-link text-dark"
                        >Page {{ pagination.current_page }} of
                        {{ pagination.last_page }}</a
                    >
                </li>

                <li
                    class="page-item"
                    v-bind:class="[{ disabled: !pagination.next_page_url }]"
                >
                    <a
                        href="#"
                        class="page-link"
                        @click="fetchPosts(pagination.next_page_url)"
                        >Next</a
                    >
                </li>
            </ul>
        </nav>
        <div
            class="card card-body mb-2"
            v-for="post in posts"
            v-bind:key="post.id"
        >
            <img
                class="mb-2"
                src="https://wowslider.com/sliders/demo-40/data1/images/cherry_blossom.jpg"
            />
            <h3>{{ post.title }}</h3>
            <p>{{ post.body }}</p>
            <hr />
            <button @click="editPost(post)" class="btn btn-secondary mb-2">
                Edit
            </button>
            <button @click="deletePost(post.id)" class="btn btn-danger">
                Delete
            </button>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            posts: [],
            post: {
                id: "",
                title: "",
                body: ""
            },
            post_id: "",
            pagination: {},
            edit: false
        };
    },

    created() {
        this.fetchPosts();
    },

    methods: {
        fetchPosts(page_url) {
            let vm = this;
            page_url = page_url || "/api/posts";
            fetch(page_url)
                .then(res => res.json())
                .then(res => {
                    this.posts = res.data;
                    vm.makePagination(res.meta, res.links);
                })
                .catch(err => console.log(err));
        },
        makePagination(meta, links) {
            let pagination = {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            };
            this.pagination = pagination;
        },
        deletePost(id) {
            if (confirm("Are you sure?")) {
                fetch(`api/post/${id}`, {
                    method: "delete"
                })
                    .then(res => res.json())
                    .then(data => {
                        alert("Post Removed");
                        this.fetchPosts();
                    })
                    .catch(err => console.log(err));
            }
        },
        addPost() {
            if (this.edit === false) {
                fetch("api/post", {
                    method: "post",
                    body: JSON.stringify(this.post),
                    headers: {
                        "content-type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        this.post.title = "";
                        this.post.body = "";
                        alert("Post Added");
                        this.fetchPosts();
                    })
                    .catch(err => console.log(err));
            } else {
                fetch("api/post", {
                    method: "put",
                    body: JSON.stringify(this.post),
                    headers: {
                        "content-type": "application/json"
                    }
                })
                    .then(res => res.json())
                    .then(data => {
                        console.log("upd " + this.post.id);
                        this.post.title = "";
                        this.post.body = "";
                        this.post.post_id = null;
                        alert("Post Updated");
                        this.fetchPosts();
                        this.edit = false;
                    })
                    .catch(err => console.log(err));
            }
        },
        editPost(post) {
            this.edit = true;
            this.post.id = post.id;
            this.post.post_id = post.id;
            this.post.title = post.title;
            this.post.body = post.body;
        },
        onFileSelected(event) {
            this.selectedFile = event.target.files[0];
        },
        onUpload() {}
    }
};
</script>

<style></style>
