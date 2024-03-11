<script>
  import meetups from "./Meetups/meetups-store.js";
  import Header from "./UI/Header.svelte";
  import MeetupGrid from "./Meetups/MeetupGrid.svelte";
  // import Button from "./UI/Button.svelte";
  import EditMeetup from "./Meetups/EditMeetup.svelte";
  import MeetupDetail from "./Meetups/MeetupDetail.svelte";
  import LoadingSpinner from "./UI/LoadingSpinner.svelte";
  import Error from "./UI/Error.svelte";

  let editMode;
  let page = "overview";
  let pageData = {};
  let editedId;
  let isLoading = true;
  let error;

  fetch("https://svelte-demo-d7a07-default-rtdb.firebaseio.com/meetups.json")
    .then((res) => {
      if (!res.ok) {
        throw new Error("Fetching meetup failed. Try again later!");
      }
      return res.json();
    })
    .then((data) => {
      const loadedMeetups = [];
      for (const key in data) {
        loadedMeetups.push({
          ...data[key],
          id: key,
        });
      }
      setTimeout(() => {
        isLoading = false;
        meetups.setMeetups(loadedMeetups.reverse());
      }, 1000);
    })
    .catch((err) => {
      error = err
      isLoading = false;
      console.log(err);
    });

  function savedMeetup(e) {
    editMode = null;
    editedId = null;
  }
  function cancelEdit() {
    editMode = null;
    editedId = null;
  }
  function showDetails(e) {
    page = "details";
    pageData.id = e.detail;
  }
  function closeDetails() {
    page = "overview";
    pageData = {};
  }
  function startEdit(e) {
    editMode = "edit";
    editedId = e.detail;
  }
  function clearError(){
    error = null;
  }
</script>

{#if error}
<Error message={error.message} on:cancel={clearError} />
{/if}

<Header />

<main>
  {#if page === "overview"}
    {#if editMode === "edit"}
      <EditMeetup id={editedId} on:save={savedMeetup} on:cancel={cancelEdit} />
    {/if}

    {#if isLoading}
      <LoadingSpinner />
    {:else}
      <MeetupGrid
        meetups={$meetups}
        on:showdetails={showDetails}
        on:edit={startEdit}
        on:add={() => {
          editMode = "edit";
        }}
      />
    {/if}
  {:else}
    <MeetupDetail id={pageData.id} on:close={closeDetails} />
  {/if}
</main>

<style>
  main {
    margin-top: 5rem;
  }
</style>
