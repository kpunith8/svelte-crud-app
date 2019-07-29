<script>
  import { onMount } from "svelte";
  import axios from "axios";
  import PostForm from "../components/PostForm.svelte";

  const baseURL = "https://ndb99xkpdk.execute-api.eu-west-2.amazonaws.com/dev";

  let posts = [];
  let postLimit = 5;
  let editingPost = {
    body: "",
    title: "",
    id: null
  };

  onMount(async () => {
    const res = await axios.get(`${baseURL}/posts/${postLimit}`);
    posts = await res.data;
  });

  function editPost(post) {
    console.log("Post to edit:", post);
    editingPost = post;
  }

  async function deletePost(id) {
    let deleteRequest = await axios.delete(`${baseURL}/post/id`, id);
    let res = await deleteRequest.data;
    if (res) {
      posts = posts.filter(p => p.id !== id);
    }
  }

  function addPost({ detail: post }) {
    if (posts.find(p => p.id === post.id)) {
      const index = posts.findIndex(p => p.id === post.id);
      let postsUpdated = posts;
      postsUpdated.splice(index, 1, post);

      posts = postsUpdated;
    } else {
      posts = [post, ...posts];
    }

    editingPost = {
      body: "",
      title: "",
      id: null
    };
  }

  async function setLimit() {
    const res = await axios.get(`${baseURL}/posts/${postLimit}`);
    const data = await res.data;
    posts = data;
  }
</script>

<style>
  .btn-delete {
    color: red !important;
  }

  .card-title {
    margin-bottom: 0;
  }
  .timestamp {
    color: #999;
    margin-bottom: 10px;
  }
</style>

<div class="row">
  <div class="col s6">
    <PostForm on:postCreated={addPost} {editingPost} />
  </div>

  <div class="col s3" style="margin:35px">
    <p>Limit number of posts</p>
    <input type="number" bind:value={postLimit} />
    <button on:click={setLimit} class="waves-effect waves-light btn">
      Set
    </button>
  </div>
</div>
<div class="row">
  {#if posts.length === 0}
    <div class="preloader-wrapper big active">
    <div class="spinner-layer spinner-blue-only">
      <div class="circle-clipper left">
        <div class="circle" />
      </div>
      <div class="gap-patch">
        <div class="circle" />
      </div>
      <div class="circle-clipper right">
        <div class="circle" />
      </div>
    </div>
  </div>
  {:else}
    {#each posts as post}
      <div class="col s6">
        <div class="card">
          <div class="card-content">
            <p class="card-title">{post.title}</p>
            <p class="timestamp">{post.createdAt}</p>
            <p>{post.body}</p>
          </div>
          <div class="card-action">
            <a href="#" on:click={() => editPost(post)}>Edit</a>
            <a href="#" on:click={() => deletePost(post.id)} class="btn-delete">
              Delete
            </a>
          </div>
        </div>
      </div>
    {/each}
  {/if}
</div>
