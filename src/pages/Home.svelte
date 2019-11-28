<script>
  // lifecycle
  import { onMount } from "svelte";
  import PostForm from "../components/PostForm.svelte";
  const apiBaseUrl =
    "https://ndb99xkpdk.execute-api.eu-west-2.amazonaws.com/dev";
  let posts = [];
  let postLimit = 6;

  let editingPost = {
    body: "",
    title: "",
    id: null
  };
  onMount(async () => {
    const res = await fetch(apiBaseUrl + "/posts");
    posts = await res.json();
  });

  function addPost({ detail: post }) {
    if (posts.find(p => p.id === post.id)) {
      const index = post.findIndex(p => {
        p.id === post.id;
        let postUpdated = post;
        postUpdated.splice(index, 1, post);
        posts = postUpdated;
      });
    } else posts = [post, ...posts];
    let editingPost = {
      body: "",
      title: "",
      id: null
    };
  }

  function editPost(post) {
    editingPost = post;
  }

  function deletePost(id) {
    if (comfirm("sure?")) {
      fetch(`${apiBaseUrl}/post/${id}`, {
        method: "DELETE"
      })
        .then(res => {
          return res.json();
        })
        .then(() => {
          posts = posts.filter(p => {
            p.id !== id;
          });
        });
    }
  }

  function setLimit() {
    fetch(`${apiBaseUrl}/post/${postLimit}`)
      .then(res => {
        return res.json();
      })
      .then(postData => {
        posts = postData;
      });
  }
</script>

<style>
  .delete-btn {
    color: red !important;
  }
  .card .card-content .card-title {
    margin-bottom: 0;
  }
  .card .card-content p.timestamp {
    color: #999;
    margin-bottom: 10px;
  }
</style>

<div class="row">
  <div class="col s6">
    <PostForm on:postCreated={addPost} {editingPost} />
  </div>
  <div class="col s3">
    <p>Limit number of posts</p>
    <input type="number" bind:value={postLimit} />
    <button on:click={setLimit} class="waves-effect waves-light btn">
      Set
    </button>

  </div>
</div>
<div class="row">
  {#if posts.length === 0}
    <h3>Loading posts...</h3>
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
            <a href="#" class="delete-btn" on:click={() => deletePost(post.id)}>
              Delete
            </a>
          </div>
        </div>
      </div>
    {/each}
  {/if}
</div>
