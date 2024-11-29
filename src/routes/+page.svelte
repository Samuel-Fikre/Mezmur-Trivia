<script lang="ts">
  import "../app.css";
  import { onMount } from "svelte";
  import TrackList from "../lib/components/TrackList.svelte";
  import { Button } from "$lib/components/ui/button/index.js";
  import * as Pagination from "$lib/components/ui/pagination";
  import * as Card from "$lib/components/ui/card/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import { toast } from "svelte-sonner";
  import * as Accordion from "$lib/components/ui/accordion/index.js";

  interface Track {
    id: number;
    label: string;
    songUrl: string;
  }

  interface SongInfo {
    title: string;
    year: string;
    hint: string;
    views: string;
    difficulty: string;
  }

  interface Song {
    title: string;
    year: string;
    description: string;
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
  let showInstructions = false;

  onMount(async () => {
    await fetchSongs();
    loadSong(currentPage - 1);
  });

  async function fetchSongs() {
    try {
      const response = await fetch("https://mezmurbackend.onrender.com/api/songs");
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
        { id: 2, label: "Bass", songUrl: song.songs[1] },
        { id: 3, label: "Instrument", songUrl: song.songs[2] },
        { id: 4, label: "Combined", songUrl: song.songs[3] },
      ];
      songInfo = {
        title: song.title,
        year: song.year,
        hint: song.description,
        views: "0",
        difficulty: "Medium"
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
      feedbackMessage = "got";
    } else {
      feedbackMessage = "not";
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

  function handleBassPlay() {
    toast.success("Bass Track Playing", {
      description: "Volume up to hear the bass line! 🎸",
      duration: 3000
    });
  }

  function toggleInstructions() {
    showInstructions = !showInstructions;
  }
</script>

<main class="h-screen p-4 sm:p-6 md:p-8 flex flex-col">
  <div class="flex justify-center h-[10vh]">
    <img class="h-full object-contain -mt-10" src="https://utfs.io/f/ANNyZZJHi12wg4TA4w63j0iDgPdnJFK9BYayENlqe67mcptS" alt="">
  </div>
  <Card.Root class="mx-auto w-full max-w-[95%] sm:max-w-lg h-[85vh] flex flex-col bg-gray-800 text-white">
    <Card.Header class="p-3 sm:p-4">
      <div class="flex items-center justify-between">
        <Card.Title class="text-base sm:text-lg text-yellow-300">Guess the Song</Card.Title>
        <button 
          class="text-gray-400 hover:text-white transition-colors" 
          on:click={toggleInstructions}
          aria-label="Game Instructions"
        >
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 5.25h.008v.008H12v-.008Z" />
          </svg>
        </button>
      </div>
      <Card.Description class="text-xs sm:text-sm text-gray-400">
        Listen to the clips and guess the title of the song!
      </Card.Description>
    </Card.Header>
    <Card.Content class="flex-1 overflow-y-auto p-3 sm:p-4">
      {#if showInstructions}
        <div class="mb-4">
          <Accordion.Root class="w-full">
            <Accordion.Item value="how-to-play">
              <Accordion.Trigger class="text-sm text-yellow-300">How to Play</Accordion.Trigger>
              <Accordion.Content class="text-sm text-gray-300">
                <ul class="list-disc pl-4 space-y-2">
                  <li>Listen to different track parts of the song (Drums, Bass, Instrument)</li>
                  <li>Use the hints and year information provided</li>
                  <li>Type your guess in the input field</li>
                  <li>Submit your guess to see if you're correct!</li>
                  <li>If you want to reach Here is My Telegram Handle @sami_f_k</li>
                </ul>
              </Accordion.Content>
            </Accordion.Item>
            <Accordion.Item value="tips">
              <Accordion.Trigger class="text-sm text-yellow-300">Tips & Tricks</Accordion.Trigger>
              <Accordion.Content class="text-sm text-gray-300">
                <ul class="list-disc pl-4 space-y-2">
                  <li>Start with the Combined track for the full experience</li>
                  <li>Pay attention to the year and hint provided</li>
                  <li>Try different parts of the song if you're stuck</li>
                </ul>
              </Accordion.Content>
            </Accordion.Item>
          </Accordion.Root>
        </div>
      {/if}

      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2 p-2 sm:p-3 border border-gray-700 rounded-md bg-gray-900 text-white">
        <div>
          <p class="text-xs text-gray-400">Year</p>
          <p class="text-sm sm:text-base font-semibold">{songInfo?.year || "Unknown"}</p>
        </div>
        <div>
          <p class="text-xs text-gray-400">Hint</p>
          <p class="text-sm sm:text-base font-semibold">{songInfo?.hint || "No hint available"}</p>
        </div>
      </div>

      <TrackList {tracks} {songInfo} onBassPlay={handleBassPlay} />
      <div class="mt-3 text-center">
        <Input
          bind:value={userGuess}
          placeholder="Guess the song title..."
          on:input={handleInput}
          class="mb-2 text-sm bg-gray-700 text-white border-gray-600 placeholder-gray-400"
        />
        <Button on:click={handleGuess} class="w-full text-sm bg-yellow-500 hover:bg-yellow-600 text-black">
          Submit Guess
        </Button>
        {#if suggestions.length > 0}
          <ul class="list-none mt-2 p-2 max-h-[15vh] overflow-y-auto bg-gray-700 border border-gray-600 rounded-md">
            {#each suggestions as suggestion}
              <button
                class="w-full text-left py-1.5 px-2 text-xs sm:text-sm text-white hover:bg-gray-600"
                on:click={() => handleSuggestionClick(suggestion)}
              >
                {suggestion}
              </button>
            {/each}
          </ul>
        {/if}
        {#if feedbackMessage}
          <div class="mt-2 flex items-center justify-center gap-2 text-sm sm:text-base">
            {#if feedbackMessage === "got"}
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="lightgreen" class="size-6">
                <path fill-rule="evenodd" d="M5.166 2.621v.858c-1.035.148-2.059.33-3.071.543a.75.75 0 0 0-.584.859 6.753 6.753 0 0 0 6.138 5.6 6.73 6.73 0 0 0 2.743 1.346A6.707 6.707 0 0 1 9.279 15H8.54c-1.036 0-1.875.84-1.875 1.875V19.5h-.75a2.25 2.25 0 0 0-2.25 2.25c0 .414.336.75.75.75h15a.75.75 0 0 0 .75-.75 2.25 2.25 0 0 0-2.25-2.25h-.75v-2.625c0-1.036-.84-1.875-1.875-1.875h-.739a6.706 6.706 0 0 1-1.112-3.173 6.73 6.73 0 0 0 2.743-1.347 6.753 6.753 0 0 0 6.139-5.6.75.75 0 0 0-.585-.858 47.077 47.077 0 0 0-3.07-.543V2.62a.75.75 0 0 0-.658-.744 49.22 49.22 0 0 0-6.093-.377c-2.063 0-4.096.128-6.093.377a.75.75 0 0 0-.657.744Zm0 2.629c0 1.196.312 2.32.857 3.294A5.266 5.266 0 0 1 3.16 5.337a45.6 45.6 0 0 1 2.006-.343v.256Zm13.5 0v-.256c.674.1 1.343.214 2.006.343a5.265 5.265 0 0 1-2.863 3.207 6.72 6.72 0 0 0 .857-3.294Z" clip-rule="evenodd" />
              </svg>
              <span class="text-lightgreen">You got it!</span>
            {:else}
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="red" class="size-6">
                <path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25Zm-1.72 6.97a.75.75 0 1 0-1.06 1.06L10.94 12l-1.72 1.72a.75.75 0 1 0 1.06 1.06L12 13.06l1.72 1.72a.75.75 0 1 0 1.06-1.06L13.06 12l1.72-1.72a.75.75 0 1 0-1.06-1.06L12 10.94l-1.72-1.72Z" clip-rule="evenodd" />
              </svg>
              <span class="text-red-500">You don't got it!</span>
            {/if}
          </div>
        {/if}
      </div>
    </Card.Content>
    <Card.Footer class="bg-gray-900 p-2 sm:p-3">
      <Pagination.Root count={database.length} perPage={perPage} let:pages let:currentPage>
        <Pagination.Content class="flex flex-wrap justify-center gap-1">
          <Pagination.Item>
            <Pagination.PrevButton class="text-xs sm:text-sm" on:click={() => handlePageChange((currentPage || 1) - 1)} />
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
                  class={`px-2 py-1 text-xs sm:text-sm rounded-md ${
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
            <Pagination.NextButton class="text-xs sm:text-sm" on:click={() => handlePageChange((currentPage || 1) + 1)} />
          </Pagination.Item>
        </Pagination.Content>
      </Pagination.Root>
    </Card.Footer>
  </Card.Root>
</main>

<style>
  main {
    color: #eaeaea;
    background-color: #1c1c1c;
    font-family: Arial, sans-serif;
  }

  button {
    cursor: pointer;
  }

  :global(.card-content) {
    scrollbar-width: thin;
    scrollbar-color: #4a4a4a #2d2d2d;
  }

  :global(.card-content::-webkit-scrollbar) {
    width: 8px;
  }

  :global(.card-content::-webkit-scrollbar-track) {
    background: #2d2d2d;
  }

  :global(.card-content::-webkit-scrollbar-thumb) {
    background-color: #4a4a4a;
    border-radius: 4px;
  }
</style>
