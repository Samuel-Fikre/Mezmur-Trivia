<script lang="ts">
  import "../app.css";
  import { onMount } from "svelte";
  import TrackList from "../lib/components/TrackList.svelte";
  import { Button } from "$lib/components/ui/button/index.js";
  import * as Pagination from "$lib/components/ui/pagination";
  import * as Card from "$lib/components/ui/card/index.js";
  import { Input } from "$lib/components/ui/input/index.js";

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
    songs: string[];
  }

  let tracks: Track[] = [];
  let songInfo: SongInfo | null = null;
  let database: Song[] = [];
  let userGuess = "";
  let suggestions: string[] = [];
  let feedbackMessage = "";

  const perPage = 1;
  let currentPage = 1;

  onMount(async () => {
    await fetchSongs();
    loadSong(currentPage - 1);
  });

  async function fetchSongs() {
    try {
      const response = await fetch("http://localhost:3000/api/songs");
      database = await response.json();
    } catch (error) {
      console.error("Error fetching songs:", error);
    }
  }

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
      feedbackMessage = "";
      userGuess = "";
      suggestions = [];
    }
  }

  function handleInput(event: Event) {
    const input = (event.target as HTMLInputElement).value;
    userGuess = input;
    suggestions = database
      .map((song) => song.title)
      .filter((title) => title.toLowerCase().includes(input.toLowerCase()));

    if (input === "") {
      suggestions = [];
    }
  }

  function handleGuess() {
    if (userGuess.trim().toLowerCase() === songInfo?.title.toLowerCase()) {
      feedbackMessage = "🎉 You got it!";
    } else {
      feedbackMessage = "❌ You don’t got it!";
    }
  }

  function handleSuggestionClick(suggestion: string) {
    userGuess = suggestion;
    suggestions = [];
  }

  function handlePageChange(page: number) {
    currentPage = page;
    loadSong(currentPage - 1);
  }
</script>

<main>
  <div class="flex justify-center">
    <img class="h-28" src="https://utfs.io/f/ANNyZZJHi12wg4TA4w63j0iDgPdnJFK9BYayENlqe67mcptS" alt="">
  </div>
  <Card.Root class="mx-auto max-w-lg mt-8 bg-gray-800 text-white">
    <Card.Header>
      <Card.Title class="text-xl text-yellow-300">Guess the Song</Card.Title>
      <Card.Description class="text-gray-400">
        Listen to the clips and guess the title of the song!
      </Card.Description>
    </Card.Header>
    <Card.Content>
      <TrackList {tracks} {songInfo} />
      <div class="mt-6 text-center">
        <Input
          bind:value={userGuess}
          placeholder="Guess the song title..."
          on:input={handleInput}
          class="mb-4 bg-gray-700 text-white border-gray-600 placeholder-gray-400"
        />
        <Button on:click={handleGuess} class="w-full bg-yellow-500 hover:bg-yellow-600 text-black">
          Submit Guess
        </Button>
        {#if suggestions.length > 0}
          <ul class="list-none mt-2 p-2 bg-gray-700 border border-gray-600 rounded-md">
            {#each suggestions as suggestion}
              <button
                class="w-full text-left py-2 px-4 text-white hover:bg-gray-600"
                on:click={() => handleSuggestionClick(suggestion)}
              >
                {suggestion}
              </button>
            {/each}
          </ul>
        {/if}
        {#if feedbackMessage}
          <p
            class="mt-4 text-lg"
            style="color: {feedbackMessage.includes('got it') ? 'lightgreen' : 'red'};"
          >
            {feedbackMessage}
          </p>
        {/if}
      </div>
    </Card.Content>
    <Card.Footer class="bg-gray-900 p-4">
      <Pagination.Root count={database.length} perPage={perPage} let:pages let:currentPage>
        <Pagination.Content>
          <Pagination.Item>
            <Pagination.PrevButton on:click={() => handlePageChange((currentPage || 1) - 1)} />
          </Pagination.Item>
          {#each pages as page (page.key)}
            {#if page.type === "ellipsis"}
              <Pagination.Item>
                <Pagination.Ellipsis class="text-gray-500" />
              </Pagination.Item>
            {:else}
              <Pagination.Item>
                <Pagination.Link
                  {page}
                  isActive={currentPage == page.value}
                  class={`px-3 py-1 rounded-md ${
                    currentPage == page.value
                      ? "bg-yellow-500 text-black"
                      : "hover:bg-gray-700 text-gray-300"
                  }`}
                  on:click={() => handlePageChange(page.value)}
                >
                  {page.value}
                </Pagination.Link>
              </Pagination.Item>
            {/if}
          {/each}
          <Pagination.Item>
            <Pagination.NextButton on:click={() => handlePageChange((currentPage || 1) + 1)} />
          </Pagination.Item>
        </Pagination.Content>
      </Pagination.Root>
    </Card.Footer>
  </Card.Root>
</main>

<style>
  main {
    color: #eaeaea;
    background-color: #1a1a1a;
    padding: 20px;
    font-family: Arial, sans-serif;
    min-height: 100vh;
  }

  

  button {
    cursor: pointer;
  }
</style>
