<script lang="ts">
  import { currentUser, pb } from "./pocketbase";
  import Feed from "./Feed.svelte";
  let username: string;
  let password: string;
  let passwordConfirm: string;
  let feedbackString = "";
  let email: string;
  let signingIn: boolean = false;

  async function login() {
    feedbackString = "";
    try {
      console.log("Authenticating...");
      await pb.collection("flowUsers").authWithPassword(username, password);
      console.log("Authentication successful");
    } catch (error) {
      console.error("Authentication error:", error);
      feedbackString = "Wrong password or user doesn't exist";
    }
  }

  async function signUp() {
    feedbackString = "";
    try {
      const data = {
        email,
        username,
        password,
        passwordConfirm,
        name: username,
      };
      const createdUser = await pb.collection("flowUsers").create(data);
      signingIn = false;
      feedbackString = "Success";
    } catch (err) {
      console.error(err);
      feedbackString = "User already exists or password was too short";
    }
  }

  function signOut() {
    pb.authStore.clear();
    window.location.reload();
  }
</script>

{#if $currentUser}<Feed />{/if}

{#if !$currentUser && signingIn === false}

  <div class="screendiv"><div class="card"><h1>flow</h1>
    <form on:submit|preventDefault>
      <input placeholder="Username" type="text" bind:value={username} />
      <input placeholder="Password" type="password" bind:value={password} />
      <button on:click={login}>Login</button>
    </form><p>{feedbackString}</p>
    <button style="margin:1px"on:click={() => {signingIn = true; feedbackString = ""; }}>Create account</button>
    
  </div></div>
{/if}
{#if !$currentUser && signingIn === true}

  <div class="screendiv"><div class="card"><h1>flow</h1>
    
    <form on:submit|preventDefault>
      <input placeholder="Email (optional)" type="text" bind:value={email} />
      <input placeholder="Username" type="text" bind:value={username} />
      <input
        placeholder="Password (characters >= 8)"
        type="password"
        bind:value={password}
      />
      <input
        placeholder="Confirm password"
        type="password"
        bind:value={passwordConfirm}
      />

      <button on:click={signUp}>SignUp</button>
    </form>

    {feedbackString}
    <button on:click={() => { signingIn = false; feedbackString = ""; }}>Back</button>

  </div></div>
{/if}

<style>
  h1{
    color:antiquewhite
  }
  .screendiv {
    display: flex;
    
    align-items: center;
    flex-direction: column;
    height: 100vh;
    
    box-shadow: rgba(0, 0, 0, 0.16) 0px 1px 4px;
  }
  .card{
    margin-top:15%;
    box-shadow: rgba(0, 0, 0, 0.24) 0px 3px 8px;
    padding:10px;
    border-radius: 10px;
    background-color: #2a353f;
  }
  form {
    display: flex;
    flex-direction: column;
    padding: 2px;
    margin: 2px;
  }
  input {
    margin: 2px;
  }



/* CSS */
button {
  appearance: none;
  background-color: #FAFBFC;
  border: 1px solid rgba(27, 31, 35, 0.15);
  border-radius: 6px;
  box-shadow: rgba(27, 31, 35, 0.04) 0 1px 0, rgba(255, 255, 255, 0.25) 0 1px 0 inset;
  box-sizing: border-box;
  color: #24292E;
  cursor: pointer;
  display: inline-block;
  font-family: -apple-system, system-ui, "Segoe UI", Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji";
  font-size: 14px;
  font-weight: 500;
  line-height: 20px;
  list-style: none;
  padding: 6px 16px;
  position: relative;
  transition: background-color 0.2s cubic-bezier(0.3, 0, 0.5, 1);
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  white-space: nowrap;
  word-wrap: break-word;
}

button:hover {
  background-color: #F3F4F6;
  text-decoration: none;
  transition-duration: 0.1s;
}

button:disabled {
  background-color: #FAFBFC;
  border-color: rgba(27, 31, 35, 0.15);
  color: #959DA5;
  cursor: default;
}

button:active {
  background-color: #EDEFF2;
  box-shadow: rgba(225, 228, 232, 0.2) 0 1px 0 inset;
  transition: none 0s;
}

button:focus {
  outline: 1px transparent;
}

button:before {
  display: none;
}

button:-webkit-details-marker {
  display: none;
}
</style>
