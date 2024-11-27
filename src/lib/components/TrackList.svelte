<script lang="ts">
  import { Button } from "$lib/components/ui/button/index.js";

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

  interface $$Props {
    tracks: Track[];
    songInfo: SongInfo | null;
    onBassPlay: () => void;
  }

  export let tracks: { id: number; label: string; songUrl: string; }[] = [];
  export const songInfo: { title: string; year: string; views: string; difficulty: string; } | null = null;

  let currentTrackIndex: number | null = null; // Tracks the currently playing index
  let audio: HTMLAudioElement | null = null; // For playing audio

  function playTrack(index: number) {
    if (audio) {
      audio.pause(); // Stop current audio
    }
    audio = new Audio(tracks[index].songUrl); // Load new track
    audio.play(); // Play the track
    currentTrackIndex = index; // Update state
  }

  function skipTrack() {
    if (currentTrackIndex !== null && currentTrackIndex < tracks.length - 1) {
      playTrack(currentTrackIndex + 1); // Play next track
    }
  }
</script>

<style>
  .track-list {
    margin-top: 20px;
    background-color: black;
    border-radius: 10px;
    padding: 20px;
  }

  .track {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    border-bottom: 1px solid #666;
  }

  .track.active {
    background-color: #222;
    color: #00ff00;
  }
</style>

<div class="track-list">
  {#each tracks as track, index}
    <div class="track {currentTrackIndex === index ? 'active' : ''}">
      <span>{track.id} - {track.label}</span>
      <div class="button-container">
        <Button
          variant="default"
          color={currentTrackIndex === index ? "success" : "primary"}
          on:click={() => playTrack(index)}
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="size-6"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"
            />
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M15.91 11.672a.375.375 0 0 1 0 .656l-5.603 3.113a.375.375 0 0 1-.557-.328V8.887c0-.286.307-.466.557-.327l5.603 3.112Z"
            />
          </svg>
        </Button>
        {#if currentTrackIndex === index}
          <Button
            variant="default"
            color="secondary"
            on:click={skipTrack}
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="size-6"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M3 8.689c0-.864.933-1.406 1.683-.977l7.108 4.061a1.125 1.125 0 0 1 0 1.954l-7.108 4.061A1.125 1.125 0 0 1 3 16.811V8.69ZM12.75 8.689c0-.864.933-1.406 1.683-.977l7.108 4.061a1.125 1.125 0 0 1 0 1.954l-7.108 4.061a1.125 1.125 0 0 1-1.683-.977V8.69Z"
              />
            </svg>
          </Button>
        {/if}
      </div>
    </div>
  {/each}
</div>
