<script lang="ts">
  import { currentUser, pb } from "./pocketbase";
  import { onMount, onDestroy } from "svelte";
  import Profile from "./Profile.svelte";
  let comment: string = "";
  let formData = new FormData();
  let postlist = [];
  let commentlist = [];
  let fileName = "";
  let getitems = 5;
  let selectedPost = null;
  let active = -1;
  let title = "";
  let ErrorString = "";
  let CommentComment = "";
  let CommentFilename = "";

  function selectPost(post, index) {
    selectedPost = post;
    active = index;
  }

  function isVideo(src) {
    const videoExtensions = ["mp4", "webm", "mov"];
    const fileExtension = src.split(".").pop().toLowerCase();
    return videoExtensions.includes(fileExtension);
  }

  function adjustHeight(node) {
    node.style.height = "1px";
    node.style.height = node.scrollHeight + "px";

    node.addEventListener("input", () => {
      node.style.height = "1px";
      node.style.height = node.scrollHeight + "px";
    });

    return {
      destroy() {
        node.removeEventListener("input", () => {
          node.style.height = "1px";
          node.style.height = node.scrollHeight + "px";
        });
      },
    };
  }

  function formatDate(date) {
    const monthNames = [
      "Jan",
      "Feb",
      "Mar",
      "Apr",
      "May",
      "Jun",
      "Jul",
      "Aug",
      "Sep",
      "Oct",
      "Nov",
      "Dec",
    ];

    const hours = date.getHours().toString().padStart(2, "0");
    const minutes = date.getMinutes().toString().padStart(2, "0");
    const month = monthNames[date.getMonth()];
    const day = date.getDate();
    const year = date.getFullYear();

    return `${hours}:${minutes} · ${month} ${day}, ${year}`;
  }
  pb.autoCancellation(false);

  async function fetchPosts() {
    try {
      const { items: records } = await pb
        .collection("posts")
        .getList(1, getitems, {
          sort: "-created",
        });

      const promises = records.map(async (posts) => {
        const user = await pb.collection("flowUsers").getOne(posts.owner, {
          params: { _ts: Date.now() },
        });
        
        return {
          avatar: user.avatar,
          owner: posts.owner,
          id: posts.id,
          username: user.username,
          title: posts.title,
          postField: posts.postField,
          created: posts.created,
          image:
            "https://mikael.software/pb/api/files/posts/" +
            posts.id +
            "/" +
            posts.image,
        };
      });

      postlist = await Promise.all(promises);
    } catch (error) {
      console.error("Error fetching posts:", error);
    }
  }
  async function fetchComments() {
    try {
      const records = await pb.collection("postscomments").getFullList({
        sort: "+created",
      });

      const promises = records.map(async (comments) => {
        const user = await pb.collection("flowUsers").getOne(comments.owner, {
          params: { _ts: Date.now() },
        });

        return {
          owner: comments.owner,
          avatar: user.avatar,
          id: comments.id,
          username: user.username,
          commenttext: comments.commenttext,
          created: comments.created,
          parentpost: comments.parentpost,
          image:
            "https://mikael.software/pb/api/files/postscomments/" +
            comments.id +
            "/" +
            comments.image,
        };
      });

      commentlist = await Promise.all(promises);
    } catch (error) {
      console.error("Error fetching posts:", error);
    }
  }

  function signOut() {
    pb.authStore.clear();
    window.location.reload();
  }

  function handleFileInput(e) {
    const fileInput = e.target;

    for (let file of fileInput.files) {
      formData.append("image", file);
    }
    fileName = fileInput.files.length > 0 ? fileInput.files[0].name : "";
  }

  function handleCommentFileInput(e) {
    const fileInput = e.target;

    for (let file of fileInput.files) {
      formData.append("image", file);
    }
    CommentFilename = fileInput.files.length > 0 ? fileInput.files[0].name : "";
  }
  function hashCode(str) {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = str.charCodeAt(i) + ((hash << 5) - hash);
    }
    return hash;
  }

  function intToRGB(i) {
    const c = (i & 0x00ffffff).toString(16).toUpperCase();
    return "#" + "00000".substring(0, 6 - c.length) + c;
  }

  function stringToColor(str) {
    return intToRGB(hashCode(str));
  }
  async function post(e) {
    e.preventDefault();

    const validationResult = validateInput();
    if (!validationResult.valid) {
      ErrorString = validationResult.errorMessage;
      return;
    }

    formData.append("postField", comment);
    formData.append("title", title);
    formData.append("owner", $currentUser.id);

    console.log("Uploading:", formData);

    try {
      const createdPost = await pb.collection("posts").create(formData);
      resetFields();
      await fetchPosts();
    } catch (err) {
      ErrorString = "An error occurred while creating the post";
      console.error(err);
      resetFields();
    }
  }

  function validateInput() {
    if (title.length < 3) {
      return {
        valid: false,
        errorMessage: "Title should be at least 3 characters long",
      };
    }

    return {
      valid: true,
    };
  }

  async function getAvatar(a) {
    const record = await pb.collection("flowUsers").getOne(a, {});
    return record.avatar;
  }

  async function postComment(e) {
    e.preventDefault();

    formData.append("commenttext", CommentComment);
    formData.append("owner", $currentUser.id);
    formData.append("parentpost", selectedPost.id);

    console.log("Uploading:", formData);

    try {
      const createdComment = await pb
        .collection("postscomments")
        .create(formData);
      resetFields();
      await fetchComments();
    } catch (err) {
      resetFields();
      console.error(err);
    }
  }

  function resetFields() {
    comment = "";
    title = "";
    CommentComment = "";
    document.getElementById("fileInput").value = "";
    formData = new FormData();
    fileName = "";
    ErrorString = "";
  }

  onMount(() => {
    fetchPosts();
    fetchComments();
  });

  function showMore() {
    getitems += 5;
    fetchPosts();
  }
</script>

<div class="page-wrapper">
  <div class="feedscreen">
    <div class="bgcard" />
    <div class="feedcard">
      <div class="feednavbar">
        <button class="navbarbutton" on:click={signOut}>Sign out</button>
        <Profile />
      </div>
      <div class="feedcontent">
        <div class="feedpost">
          <form on:submit={post}>
            <div class="input-wrapper">
              <textarea
                placeholder="Title"
                bind:value={title}
                rows="1"
                use:adjustHeight
                maxlength="20"
              />
              <textarea
                placeholder="Epic post 😎"
                bind:value={comment}
                rows="1"
                use:adjustHeight
                maxlength="100"
              />
            </div>
            <div>
              <input
                type="file"
                id="fileInput"
                on:change={(e) => handleFileInput(e)}
                style="display: none;"
              /><button
                type="button"
                onclick="document.getElementById('fileInput').click();"
              >
                🖼️
              </button>
              {fileName}<br />
              <p style="padding:0px; margin:0px">File size less than 5mb accepted</p>
            </div>
            <button class="postbutton" type="submit">Post</button>
          </form>
          <p style="color:red">{ErrorString}</p>
        </div>
        <div class="feedcommentscard">
          {#each postlist as posts, index (posts)}
            <div
              class="feedcomments {index === active ? 'active' : ''}"
              on:click={() => selectPost(posts, index)}
            >
              <span>
                <p
                  style="margin: 0px;display: inline;font-size:12px;font-style: italic;"
                >
                  {formatDate(new Date(posts.created))} ·
                </p>
                <img
                style="border-radius:40px" 
                height=25px
                width= 25px
                  src="https://mikael.software/pb/api/files/flowUsers/{posts.owner}/{posts.avatar}"
                />
                <p
                  style="color: {stringToColor(
                    posts.username
                  )}; font-weight: bolder;display: inline;"
                >
                  {posts.username}:
                </p>
              </span>
              <p style="margin-bottom:4px; font-weight:bolder">{posts.title}</p>
              {#if isVideo(posts.image)}
                <p
                  style="margin:0px;padding:0px;border:solid 1px gray; width:fit-content; border-radius: 6px;"
                >
                  🎥
                </p>
              {:else}
                <img
                  style="margin:0px;padding:0px;border:solid 1px gray; width:fit-content; border-radius: 6px"
                  src={posts.image}
                  alt=""
                  width="25"
                  height="25"
                  on:error={(e) => (e.target.style.display = "none")}
                />
              {/if}
            </div>
          {/each}
          <button class="postbutton" on:click={showMore}>Show more</button>
        </div>
      </div>
    </div>
  </div>
</div>
{#if selectedPost}
  <div class="media-container">
    {#if isVideo(selectedPost.image)}
      <p class="media-comment">{selectedPost.postField}</p>
      <video
        class="commentvideo"
        src={selectedPost.image}
        width="90%"
        controls
        on:error={(e) => (e.target.style.display = "none")}
      />
    {:else}
      <img
        class="commentimage"
        src={selectedPost.image}
        alt=""
        width="90%"
        on:error={(e) => (e.target.style.display = "none")}
      />
    {/if}

    {#each commentlist as comments}
      {#if comments.parentpost === selectedPost.id}
        <div class="commentdiv">
          <span style="background-image: linear-gradient(to top, #cfd9df 0%, #e2ebf0 100%); display: inline-block; padding: 5px; border-radius: 5px;">
            <img
                style="border-radius:20px;border: solid 2px gray" 
                height=50px
                width= 50px
                  src="https://mikael.software/pb/api/files/flowUsers/{comments.owner}/{comments.avatar}"
                  alt="yeet"
                />
            <p
              style="margin: 0px;display: inline;font-size:12px;font-style: italic;"
            >
              {formatDate(new Date(comments.created))} ·
            </p>
            
            <p
              style="color: {stringToColor(
                comments.username
              )}; font-weight: bolder;display: inline;"
            >
              {comments.username}
            </p>
          </span>
          <p style="margin:0px;padding:0px">{comments.commenttext}</p>
          {#if isVideo(comments.image)}
            <video
              class="commentvideo"
              src={comments.image}
              width="90%"
              controls
              on:error={(e) => (e.target.style.display = "none")}
            />
          {:else}
            <img
              class="commentimage"
              src={comments.image}
              alt=""
              width="90%"
              on:error={(e) => (e.target.style.display = "none")}
            />
          {/if}
        </div>
      {/if}
    {/each}
    <div
      style="box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px; padding: 3px; margin-bottom: 5px"
    >
      <form on:submit={postComment}>
        <div class="input-wrapper">
          <textarea
            placeholder="Comment"
            bind:value={CommentComment}
            rows="1"
            use:adjustHeight
            maxlength="100"
          />
        </div>
        <div>
          <input
            type="file"
            id="fileInput"
            on:change={(e) => handleCommentFileInput(e)}
            style="display: none;"
          /><button
            type="button"
            onclick="document.getElementById('fileInput').click();"
          >
            🖼️
          </button>
          {fileName}<br />
        </div>
        <button class="postbutton" type="submit">Post</button>
      </form>
    </div>
  </div>
{/if}

<style>
  :root {
  }
  .page-wrapper {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    justify-content: space-between;
  }
  .bgcard {
    top: 0;
    position: fixed;
    width: 335px;
    height: 100vh;
    z-index: -2;
    background-color: #d1d5db;
  }
  .feedscreen {
  }
  .feedcard {
    width: 350px;
    border-radius: 4px;
    position: relative;
    height: 100%;
  }

  .feednavbar {
    width: 55px;
    height: 100%;
    background-color: #d1d5db;
    position: fixed;
    padding-top: 5px;
    padding-left: 5px;
    gap: 3px;
  }

  .feedcontent {
    width: calc(100% - 100px);
    border-radius: 40px;
    display: flex;
    flex-direction: column;
    margin-left: 80px;
    padding: 0;
  }
  .media-container {
    position: fixed;
    top: 0px;
    left: 340px;
    width: 400px;
    max-height: calc(90vh);
    margin-top: 1px;
    background-color: #f5f5f5;
    padding: 10px;
    border-top-right-radius: 3px;
  border-bottom-right-radius: 3px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
    overflow-y: auto;
    overflow-wrap: break-word;
  }
  .feedpost {
    margin: 0;
    height: auto;
    padding: 10px;
    background-color: rgb(230, 230, 230);
  }

  .feedcommentscard {
    background-color: rgb(230, 230, 230);
    margin: 0;
  }
  .feedcomments {
    background-image: linear-gradient(120deg, #fdfbfb 0%, #ebedee 100%);
    width: calc(100%); /* Subtracting the border width */
    height: auto;

    overflow: auto;
    overflow-wrap: break-word;
    display: flex;
    flex-direction: column;

    padding: 10px;
    border: solid 1px rgb(202, 202, 202);
    box-sizing: border-box;
  }

  .feedcomments:hover {
    box-shadow: rgba(192, 192, 192, 0.25) 0px 15px 30px -6px inset,
      rgba(192, 192, 192, 0.3) 0px 15px 30px -6px inset;
    cursor: pointer;
  }
  .feedcomments.active {
    box-shadow: rgba(129, 129, 129, 0.25) 0px 15px 30px -6px inset,
      rgba(109, 103, 103, 0.3) 0px 15px 30px -6px inset;
  }
  .commentimage {
    background-color: white;
    border: solid rgb(202, 202, 202) 1px;
    border-radius: 20px;
  }
  .commentvideo {
    border-radius: 20px;
  }
  input {
    flex: 1;
    margin-right: 1px;
    font-size: larger;
    margin-left: 0;
  }

  /* CSS */
  .navbarbutton {
    align-items: center;
    appearance: none;
    background-color: wheat;
    background-image: linear-gradient(1deg, #8689b3, #6b6b6b 99%);
    background-size: calc(100% + 20px) calc(100% + 20px);
    border-radius: 100px;
    border-width: 0;
    box-shadow: none;
    box-sizing: border-box;
    color: #ffffff;
    cursor: pointer;
    display: inline-flex;
    font-family: CircularStd, sans-serif;
    font-size: 1rem;
    height: auto;
    justify-content: center;
    line-height: 1.5;
    margin-bottom: 3px;
    position: relative;
    text-align: center;
    text-decoration: none;
    transition: background-color 0.2s, background-position 0.2s;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
    vertical-align: top;
    white-space: nowrap;
  }

  .navbarbutton:active,
  .navbarbutton:focus {
    outline: none;
  }

  .navbarbutton:hover {
    background-position: -20px -20px;
  }

  .navbarbutton:focus:not(:active) {
    box-shadow: rgba(40, 170, 255, 0.25) 0 0 0 0.125em;
  }
  textarea {
    font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
    font-size: larger;
    width: 100%;
    box-sizing: border-box;
    resize: none; /* Prevent manual resizing */
    overflow: hidden; /* Hide the scrollbar */
    border: none;
    outline: none;
  }
  .input-wrapper {
    display: inline-block;
    max-width: 500px;
    width: 100%;
    box-sizing: border-box;
  }
  input[type="text"] {
    width: 100%;
    box-sizing: border-box;
  }

  /* CSS */
  .postbutton {
    right: 0;
    margin-top: 3px;
    background-color: #ffffff;
    border: 1px solid rgb(209, 213, 219);
    border-radius: 0.5rem;
    box-sizing: border-box;
    color: #111827;
    font-family: "Inter var", ui-sans-serif, system-ui, -apple-system, system-ui,
      "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif,
      "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol",
      "Noto Color Emoji";
    font-size: 0.875rem;
    font-weight: 600;
    line-height: 1.25rem;
    padding: 0.3rem 0.5rem;
    text-align: center;
    text-decoration: none #d1d5db solid;
    text-decoration-thickness: auto;
    box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
    cursor: pointer;
    user-select: none;
    -webkit-user-select: none;
    touch-action: manipulation;
  }

  .postbutton:hover {
    background-color: rgb(249, 250, 251);
  }

  .postbutton:focus {
    outline: 2px solid transparent;
    outline-offset: 2px;
  }

  .postbutton:focus-visible {
    box-shadow: none;
  }
  .footer {
    bottom: 0px;
    height: 100px;
    width: 100%;
    background-color: #d1d5db;
  }

  .commentdiv {
    margin-bottom: 4px;
    border-top: solid 1px orange;
  }
</style>
