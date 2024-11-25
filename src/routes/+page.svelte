<script lang="ts">
  import "../app.css";
  import { onMount } from "svelte";
  import TrackList from "../lib/components/TrackList.svelte";
  import { Button } from "$lib/components/ui/button/index.js";
  import * as Pagination from "$lib/components/ui/pagination";

  // Interfaces for type definitions
  interface Track {
    id: number;
    label: string;
    songUrl: string;
  }

  interface SongInfo {
    title: string;
    year: string;
    views: string;
    difficulty: string;
  }

  interface Song {
    title: string;
    year: string;
    songs: string[]; // Array of song URLs
  }

  // Reactive variables
  let tracks: Track[] = [];
  let songInfo: SongInfo | null = null;
  let database: Song[] = []; // Explicitly typed as an array of Song objects
  let userGuess = ""; // User's input for guessing the song
  let suggestions: string[] = []; // Suggestions for the input box
  let feedbackMessage = ""; // Feedback for the user

  // Pagination-specific variables
  const perPage = 1; // Number of songs per page
  const count = 100; // Total number of pages (assumes `database` contains songs)
  let currentPage = 1; // Start with the first page

  // Lifecycle method to fetch songs and load the first song
  onMount(async () => {
    await fetchSongs();
    loadSong(currentPage - 1); // Zero-based index for the current page
  });

  // Fetch song data from the backend
  async function fetchSongs() {
    try {
      const response = await fetch("http://localhost:3000/api/songs");
      database = await response.json();
    } catch (error) {
      console.error("Error fetching songs:", error);
    }
  }

  // Load the current song based on the index
  function loadSong(index: number) {
    if (index < database.length) {
      const song = database[index];
      tracks = [
        { id: 1, label: "Drums", songUrl: song.songs[0] },
        { id: 2, label: "Bass + Piano", songUrl: song.songs[1] },
        { id: 3, label: "Synth + Strings", songUrl: song.songs[2] },
        { id: 4, label: "Combined", songUrl: song.songs[3] },
      ];
      songInfo = {
        title: song.title,
        year: song.year,
        views: "185M",
        difficulty: "Hard (par 4)",
      };
      feedbackMessage = ""; // Clear feedback when loading a new song
      userGuess = ""; // Clear input field
      suggestions = []; // Clear suggestions
    }
  }

  // Handle user input for guessing
  function handleInput(event: Event) {
    const input = (event.target as HTMLInputElement).value;
    userGuess = input;

    // Generate suggestions dynamically based on input
    suggestions = database
      .map((song) => song.title)
      .filter((title) => title.toLowerCase().includes(input.toLowerCase()));

    // Show suggestions only if there's input
    if (input === "") {
      suggestions = [];
    }
  }

  // Check the user's guess
  function handleGuess() {
    if (userGuess.trim().toLowerCase() === songInfo?.title.toLowerCase()) {
      feedbackMessage = "🎉 You got it!";
    } else {
      feedbackMessage = "❌ You don’t got it!";
    }
  }

  // Handle suggestion click to auto-fill input
  function handleSuggestionClick(suggestion: string) {
    userGuess = suggestion;
    suggestions = []; // Hide suggestions when a suggestion is clicked
  }

  // Handle pagination changes
  function handlePageChange(page: number) {
    currentPage = page;
    loadSong(currentPage - 1); // Zero-based index for loading songs
  }
</script>

<main>
  <h1>Mezmur Trivia</h1>
  <TrackList {tracks} {songInfo} />
  <div style="margin-top: 30px;  text-align: center;">
    <input
      type="text"
      bind:value={userGuess}
      placeholder="Guess the song title..."
      on:input={handleInput}
      style="padding: 10px; width: 60%; border: 1px solid #ddd; border-radius: 5px; color: #333; background-color: #fff;"
    />
    <Button on:click={handleGuess}>Submit Guess</Button>
    {#if suggestions.length > 0}
      <ul
        style="list-style: none; padding: 0; margin-top: 10px; 
        text-align: left; color: #333; display: inline-block; width: 60%; background: #fff; border: 1px solid #ddd; border-radius: 5px;"
      >
        {#each suggestions as suggestion}
          <button
            style="padding: 5px 10px; cursor: pointer; background: none; border: none; color:black; text-align: left; width: 100%;"
            on:click={() => handleSuggestionClick(suggestion)}
          >
            {suggestion}
          </button>
        {/each}
      </ul>
    {/if}
    {#if feedbackMessage}
      <p
        style="margin-top: 20px; font-size: 18px; color: {feedbackMessage.includes('got it') ? 'green' : 'red'};"
      >
        {feedbackMessage}
      </p>
    {/if}
  </div>
  <div>
    
  </div>
  
</main>



<div class="p-4 mt-0 bg-black text-white">
  <Pagination.Root count={database.length} perPage={perPage} let:pages let:currentPage>
    <Pagination.Content>
      <Pagination.Item >
        <Pagination.PrevButton on:click={() => handlePageChange(currentPage ?? 1 - 1)} />
      </Pagination.Item>
      {#each pages as page (page.key)}
        {#if page.type === "ellipsis"}
          <Pagination.Item>
            <Pagination.Ellipsis />
          </Pagination.Item>
        {:else}
          <Pagination.Item>
            <Pagination.Link
              {page}
              isActive={currentPage == page.value}
              class={`px-4 py-2 border rounded-md ${
                currentPage == page.value
                  ? "bg-black text-white "
                  : "text-white border-gray-300 hover:bg-blue-100"
              }`}
              on:click={() => handlePageChange(page.value)}
            >
              {page.value}
            </Pagination.Link>
          </Pagination.Item>
        {/if}
      {/each}
      <Pagination.Item>
        <Pagination.NextButton 
        on:click={() => handlePageChange(currentPage ?? 1 + 1)} />
      </Pagination.Item>
    </Pagination.Content>
  </Pagination.Root>
</div>


<style>
  main {
    color: #ffffff;
    background-color: #333;
    padding: 20px;
    font-family: Arial, sans-serif;
  }

  h1 {
    text-align: center;
    color: #f5a623;
  }
</style>
