<script>
  // To communicate with parent
  import { createEventDispatcher } from "svelte";
  import axios from "axios";

  export let editingPost;
  
  // Updates the DOM when the props have been changed, introduces reactivity
  $: title = editingPost.title;
  $: body = editingPost.body;
  let loading = false;

  let dispatch = createEventDispatcher();

  const baseURL = "https://ndb99xkpdk.execute-api.eu-west-2.amazonaws.com/dev";

  async function onSubmit(event) {
    event.preventDefault();

    loading = true;
    if (title.trim() === 0 || body.trim() === 0) {
      return;
    }
    const newPost = { title, body };

    let URL;
    let method;

    if (editingPost.id) {
      URL = `${baseURL}/post/${editingPost.id}`;
      method = "PUT";
    } else {
      URL = `${baseURL}/post`;
      method = "POST";
    }

    console.log("URL:", URL, "Method:", method);

    const res = await axios({
      method: method,
      url: URL,
      data: newPost
    });

    //const res = await axios.post(`${baseURL}/post`, newPost);

    const post = await res.data;

    // reset the form fields
    // editingPost.title = "";
    // editingPost.body = "";

    // Dispatch custom event so that parent can capture it as props
    dispatch("postCreated", post);

    loading = false;
    // console.log("Posting data:", post);
  }

 
</script>

<style>

</style>

{#if !loading}
  <form on:submit={onSubmit}>
    <div class="input-field">
      <label for="title">Title</label>
      <input type="text" bind:value={editingPost.title} />
    </div>
    <div class="input-field">
      <label for="body">Body</label>
      <input type="text" bind:value={editingPost.body} />
    </div>
    <button type="Submit" class="waves-effect waves-light btn">
      {editingPost.id ? 'Update' : 'Add'}
    </button>
  </form>
{:else}
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
{/if}
