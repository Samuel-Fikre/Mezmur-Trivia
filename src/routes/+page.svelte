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
  import { ScrollArea } from "$lib/components/ui/scroll-area/index.js";
  import { Separator } from "$lib/components/ui/separator/index.js";

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
    lyrics?: string[];
    singer?: string;
    singerFacts?: string[];
  }

  let tracks: Track[] = [];
  let songInfo: SongInfo | null = null;
  let database: Song[] = [];
  let userGuess = "";
  let suggestions: string[] = [];
  let feedbackMessage = "";
  let gameMode = "song"; // Default game mode
  let currentLyric = ""; // For lyrics game mode
  let currentSingerFact = ""; // For singer knowledge game mode

  const perPage = 1;
  let currentPage = 1;
  let showInstructions = false;
  let audioElements: HTMLAudioElement[] = [];
  let allSongs: Song[] = [];

  // Game mode options
  const gameModes = [
    { id: "song", label: "Guess the Song" },
    { id: "lyrics", label: "Guess by Lyrics" },
    { id: "singer", label: "Know Your Singer" }
  ];

  onMount(() => {
    loadGameMode(gameMode);
    
    window.addEventListener('audioRefsUpdate', ((e: CustomEvent) => {
      audioElements = e.detail;
    }) as EventListener);
    
    return () => {
      window.removeEventListener('audioRefsUpdate', ((e: CustomEvent) => {
        audioElements = e.detail;
      }) as EventListener);
    };
  });

  async function loadGameMode(mode: string) {
    gameMode = mode;
    resetGame();
    
    switch (mode) {
      case "song":
        await fetchSongs();
        loadSong(currentPage - 1);
        break;
      case "lyrics":
        await fetchLyricsQuiz();
        loadLyricsQuestion(currentPage - 1);
        break;
      case "singer":
        await fetchSingerQuiz();
        loadSingerQuestion(currentPage - 1);
        break;
    }
  }
 
  async function fetchSongs() {
    try {
      const response = await fetch("https://guessmezmur-backend.onrender.com/api/songs");
      allSongs = await response.json();
      database = shuffleArray(allSongs.filter(song => song.songs && song.songs.length >= 4));
    } catch (error) {
      console.error("Error fetching songs:", error);
    }
  }

  async function fetchLyricsQuiz() {
    try {
      const response = await fetch("http://localhost:3001/api/lyrics-quiz");
      if (!response.ok) throw new Error("Failed to fetch lyrics quiz");
      allSongs = await response.json();
      database = shuffleArray(allSongs);
    } catch (error) {
      console.error("Error fetching lyrics quiz:", error);
      toast.error("Failed to load lyrics quiz");
    }
  }

  async function fetchSingerQuiz() {
    try {
      const response = await fetch("http://localhost:3001/api/singer-quiz");
      if (!response.ok) throw new Error("Failed to fetch singer quiz");
      allSongs = await response.json();
      database = shuffleArray(allSongs);
    } catch (error) {
      console.error("Error fetching singer quiz:", error);
      toast.error("Failed to load singer quiz");
    }
  }

  // Fisher-Yates shuffle algorithm
  function shuffleArray<T>(array: T[]): T[] {
    const shuffled = [...array];
    for (let i = shuffled.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    return shuffled;
  }

  function loadSong(index: number) {
    if (index < database.length) {
      const song = database[index];
      if (song.songs && song.songs.length >= 4) {
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
      } else {
        console.warn("Skipping song due to missing tracks:", song.title);
        loadSong(index + 1);
      }
    }
  }

  function loadLyricsQuestion(index: number) {
    if (database[index] && database[index].lyrics && database[index].lyrics.length > 0) {
      const randomLyricIndex = Math.floor(Math.random() * database[index].lyrics.length);
      currentLyric = database[index].lyrics[randomLyricIndex];
      songInfo = {
        title: database[index].title,
        year: database[index].year,
        hint: "Can you guess the song from these lyrics?",
        views: "",
        difficulty: "medium"
      };
    }
  }

  function loadSingerQuestion(index: number) {
    if (database[index] && database[index].singerFacts && database[index].singerFacts.length > 0) {
      const randomFactIndex = Math.floor(Math.random() * database[index].singerFacts.length);
      currentSingerFact = database[index].singerFacts[randomFactIndex];
      songInfo = {
        title: database[index].singer || "",
        year: "",
        hint: "Can you guess the singer from this fact?",
        views: "",
        difficulty: "medium"
      };
    }
  }

  function resetGame() {
    userGuess = "";
    suggestions = [];
    feedbackMessage = "";
    currentLyric = "";
    currentSingerFact = "";
    currentPage = 1;
  }

  function handleInput(event: Event) {
    const input = (event.target as HTMLInputElement).value;
    userGuess = input;
    suggestions = allSongs
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
    stopAllTracks();
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

  function stopAllTracks() {
    audioElements.forEach(audio => {
      if (audio) {
        audio.pause();
        audio.currentTime = 0;
      }
    });
  }
</script>

<main class="min-h-screen overflow-x-hidden relative">
  <!-- Background Elements -->
  <div class="fixed inset-0 bg-gradient-to-b from-[#2e1065] via-[#4c1d95] to-[#1e1b4b]"></div>
  <div class="fixed inset-0 bg-[url('/noise.svg')] opacity-10 mix-blend-soft-light"></div>
  
  <!-- Wave Pattern -->
  <div class="fixed inset-0 flex items-center justify-center pointer-events-none overflow-hidden">
    <div class="absolute w-[200%] left-1/2 -translate-x-1/2 wave-container">
      <img src="/wave.svg" alt="" class="w-full h-auto" />
    </div>
  </div>

  <div class="relative min-h-screen w-full">
    <div class="w-full max-w-3xl px-4 py-6 md:py-12 mx-auto">
      <!-- Logo -->
      <div class="flex justify-center mb-8 md:mb-12 animate-float">
        <img 
          class="w-82 md:w-44 h-14 md:h-16 object-contain opacity-90 hover:opacity-100 transition-opacity" 
          src="https://vo8mu4xic1.ufs.sh/f/yh10ZRrvFx0z0xKjLL3Uzc4F8ydXNomx76YbCtQ29sWrVwel" 
          alt="Mezmur Trivia"
        />
      </div>

      <!-- Game Header -->
      <div class="text-center mb-6 md:mb-8 space-y-2">
        <h1 class="text-xl md:text-2xl font-medium text-gradient">
          {#if gameMode === "song"}
            Guess the Song
          {:else if gameMode === "lyrics"}
            Guess by Lyrics
          {:else}
            Know Your Singer
          {/if}
        </h1>
        <p class="text-xs md:text-sm text-gray-400/80 px-4">
          {#if gameMode === "song"}
            Listen to the clips and guess the title of the song!
          {:else if gameMode === "lyrics"}
            Read the lyrics and guess the song title!
          {:else}
            Read about the singer and guess who it is!
          {/if}
        </p>
      </div>

      <!-- Game Mode Selector -->
      <div class="flex flex-wrap justify-center gap-2 mb-6 md:mb-8 px-2">
        {#each gameModes as mode}
          <Button
            variant={gameMode === mode.id ? "default" : "ghost"}
            size="sm"
            class="{gameMode === mode.id ? 'glass-button bg-white/20' : 'glass-button'} 
                   transition-all min-w-[120px] md:min-w-[140px] py-2 md:py-2.5 text-sm md:text-base"
            on:click={() => loadGameMode(mode.id)}
          >
            {mode.label}
          </Button>
        {/each}
      </div>

      <!-- Game Content -->
      <div class="space-y-4 md:space-y-6 px-2">
        <!-- Info Card -->
        {#if gameMode === "song"}
          <div class="glass-card p-3 md:p-4">
            <div class="grid grid-cols-2 gap-3 md:gap-4">
              <div>
                <p class="text-xs text-gray-400/80 mb-0.5 md:mb-1">Year</p>
                <p class="text-sm md:text-base font-medium">{songInfo?.year || "Unknown"}</p>
              </div>
              <div>
                <p class="text-xs text-gray-400/80 mb-0.5 md:mb-1">Hint</p>
                <p class="text-sm md:text-base font-medium">{songInfo?.hint || "No hint available"}</p>
              </div>
            </div>
          </div>
        {/if}

        <!-- Game Interface -->
        {#if gameMode === "song"}
          <div class="glass-card overflow-hidden">
            {#if tracks.length > 0}
              <TrackList {tracks} {songInfo} onBassPlay={handleBassPlay} />
            {/if}
          </div>
        {:else if gameMode === "lyrics"}
          <div class="glass-card p-6 md:p-8 text-center">
            <h3 class="text-lg md:text-xl font-medium text-gradient mb-3">Coming Soon!</h3>
            <p class="text-sm md:text-base text-gray-300/90">
              Soon you'll be able to test your knowledge of mezmur lyrics.
              Stay tuned for this exciting feature!
            </p>
          </div>
        {:else}
          <div class="glass-card p-6 md:p-8 text-center">
            <h3 class="text-lg md:text-xl font-medium text-gradient mb-3">Coming Soon!</h3>
            <p class="text-sm md:text-base text-gray-300/90">
              Get ready to showcase your knowledge about favourite mezmur singers.
              This feature is currently under development!
            </p>
          </div>
        {/if}

        <!-- Input Section -->
        <div class="space-y-3">
          {#if suggestions.length > 0}
            <ScrollArea class="h-28 md:h-32 glass-card">
              <div class="p-1">
                {#each suggestions as suggestion}
                  <button
                    class="w-full text-left px-3 py-2.5 md:py-2 text-sm text-gray-300 hover:bg-white/10 active:bg-white/20 rounded-md transition-colors"
                    on:click={() => handleSuggestionClick(suggestion)}
                  >
                    {suggestion}
                  </button>
                {/each}
              </div>
            </ScrollArea>
          {/if}

          <div class="flex gap-2">
            <Input
              bind:value={userGuess}
              placeholder={gameMode === "singer" ? "Enter singer name..." : "Enter song title..."}
              on:input={handleInput}
              class="glass-morphism bg-transparent border-white/20 text-white placeholder:text-gray-500 h-11 md:h-12 text-sm md:text-base"
            />
            <Button 
              on:click={handleGuess}
              class="glass-button px-6 md:px-8 h-11 md:h-12 text-sm md:text-base font-medium"
            >
              Guess
            </Button>
          </div>

          {#if feedbackMessage}
            <div class="flex justify-center items-center gap-2 py-2">
              {#if feedbackMessage === "got"}
                <div class="flex items-center gap-2 text-green-400">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.857-9.809a.75.75 0 00-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 10-1.06 1.061l2.5 2.5a.75.75 0 001.137-.089l4-5.5z" clip-rule="evenodd" />
                  </svg>
                  <span class="text-sm md:text-base font-medium">Correct!</span>
                </div>
              {:else}
                <div class="flex items-center gap-2 text-red-400">
                  <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor" class="w-5 h-5">
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.28 7.22a.75.75 0 00-1.06 1.06L8.94 10l-1.72 1.72a.75.75 0 101.06 1.06L10 11.06l1.72 1.72a.75.75 0 101.06-1.06L11.06 10l1.72-1.72a.75.75 0 00-1.06-1.06L10 8.94 8.28 7.22z" clip-rule="evenodd" />
                  </svg>
                  <span class="text-sm md:text-base font-medium">Try again!</span>
                </div>
              {/if}
            </div>
          {/if}
        </div>
      </div>

      <!-- Pagination -->
      <div class="mt-8 flex justify-center">
        <div class="flex items-center gap-2">
          <Button
            variant="ghost"
            size="icon"
            class="glass-button p-2 disabled:opacity-50 disabled:cursor-not-allowed"
            disabled={currentPage === 1}
            on:click={() => handlePageChange(currentPage - 1)}
          >
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-5 h-5">
              <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
            </svg>
          </Button>

          <div class="text-sm text-gray-300/90">
            {currentPage} / {database.length}
          </div>

          <Button
            variant="ghost"
            size="icon"
            class="glass-button p-2 disabled:opacity-50 disabled:cursor-not-allowed"
            disabled={currentPage === database.length}
            on:click={() => handlePageChange(currentPage + 1)}
          >
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-5 h-5">
              <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
            </svg>
          </Button>
        </div>
      </div>
    </div>

    <!-- Help Button -->
    <button 
      class="fixed bottom-4 md:bottom-6 right-4 md:right-6 glass-button p-2.5 md:p-3 backdrop-blur-sm transition-all shadow-lg"
      on:click={toggleInstructions}
      aria-label="Game Instructions"
    >
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-5 h-5 md:w-6 md:h-6">
        <path stroke-linecap="round" stroke-linejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 5.25h.008v.008H12v-.008Z" />
      </svg>
    </button>
  </div>

  <!-- Instructions Modal -->
  {#if showInstructions}
    <div class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
      <div class="glass-card w-full max-w-[340px] md:max-w-md">
        <div class="p-5 md:p-6">
          <h2 class="text-lg md:text-xl font-medium text-gradient mb-4">How to Play</h2>
          <div class="space-y-4">
            <div>
              <h3 class="font-medium text-yellow-400/80 mb-2">Game Modes</h3>
              <ul class="space-y-2.5 text-sm text-gray-300/90">
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>Song Mode: Listen to different instrumental parts (drums, bass, instruments) and guess the mezmur title. Each part reveals different aspects of the song!</span>
                </li>
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>Lyrics Mode (Coming Soon): Challenge yourself by guessing songs from their meaningful lyrics</span>
                </li>
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>Singer Mode (Coming Soon): Test your knowledge about favourite mezmur singers through interesting facts and their music snippets</span>
                </li>
              </ul>
            </div>
            <div>
              <h3 class="font-medium text-yellow-400/80 mb-2">Tips</h3>
              <ul class="space-y-2.5 text-sm text-gray-300/90">
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>Pay attention to the year and hints provided - they're valuable clues!</span>
                </li>
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>Try different parts of the song - sometimes the bass or drums might be the key to recognition</span>
                </li>
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>The Year is in the ian Calendar (EC)</span>
                </li>
                <li class="flex gap-2">
                  <span class="text-yellow-400/60">•</span>
                  <span>Use the suggestions dropdown to help with exact song titles</span>
                </li>
              </ul>
            </div>
          </div>
          <button 
            class="mt-6 w-full glass-button py-2.5 font-medium text-sm md:text-base"
            on:click={toggleInstructions}
          >
            Got it!
          </button>
        </div>
      </div>
    </div>
  {/if}
</main>

<style>
  :global(body) {
    -webkit-tap-highlight-color: transparent;
  }

  :global(.scrollarea-viewport) {
    -webkit-overflow-scrolling: touch !important;
    overflow-y: auto !important;
  }

  @media (max-width: 768px) {
    :global(.scrollarea-viewport) {
      scroll-behavior: smooth;
      overscroll-behavior-y: contain;
      &::-webkit-scrollbar {
        display: none;
      }
      -ms-overflow-style: none;
      scrollbar-width: none;
    }
  }
</style>
