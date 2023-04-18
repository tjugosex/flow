<script lang="ts">
  import { currentUser, pb } from "./pocketbase";
  let showProfileCard = false;
  let fileName = "";
  let formData = new FormData();
  function toggleProfileCard() {
    showProfileCard = !showProfileCard;
  }

  function handleFileInput(e) {
    const fileInput = e.target;
    for (let file of fileInput.files) {
      formData.append("avatar", file);
    }
    fileName = fileInput.files.length > 0 ? fileInput.files[0].name : "";
  }
  async function handleSubmit(e) {
    e.preventDefault();
    try {
        if (fileName != "")
        {
            const updateAvatar = await pb.collection("flowUsers").update($currentUser.id, formData);
        }

    }
    catch(err){

    }
  }
</script>

<button class="navbarbutton" on:click={toggleProfileCard}>Profile</button>
{#if showProfileCard}
  <div class="profilecard">
    <img
      width="50px"
      height="50px"
      src="https://mikael.software/pb/api/files/flowUsers/{$currentUser.id}/{$currentUser.avatar}"
    />
    <form on:submit={handleSubmit}>
      <input
        type="file"
        id="fileInput"
        on:change={(e) => handleFileInput(e)}
        style="display: none;"
      />
      <button
        type="button"
        onclick="document.getElementById('fileInput').click();"
      >
        Change
      </button>
      {fileName}
      <button type="submit">Submit</button>
    </form>
    <div>Username: {$currentUser.username}</div>

    <div>Email: {$currentUser.email}</div>
  </div>
{/if}

<style>
  .profilecard {
    position: absolute;
    width: 200px;
    height: 300px;
    background-color: #ffffff;
    opacity: 0.98;
    top: 4px;
    left: 85px;
    display: flex;
    flex-direction: column;
    gap: 5px;
    padding: 3px;
    border: solid 1px;
    z-index: 10;
  }
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
    width: 70px;
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
</style>
