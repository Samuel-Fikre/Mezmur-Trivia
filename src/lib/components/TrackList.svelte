<script lang="ts">
  import { onMount } from 'svelte';
  import { Button } from "$lib/components/ui/button/index.js";

  interface Track {
    id: number;
    label: string;
    songUrl: string;
  }

  export let tracks: { id: number; label: string; songUrl: string; }[] = [];
  export const songInfo: { title: string; year: string; views: string; difficulty: string; } | null = null;
  export let onBassPlay: () => void;

  let currentTrackIndex: number | null = null;
  let isPlaying: boolean = false;
  let audioElements: HTMLAudioElement[] = [];
  let audio: HTMLAudioElement | null = null;

  function playTrack(index: number) {
    if (currentTrackIndex === index && isPlaying) {
      if (audio) {
        audio.pause();
        isPlaying = false;
      }
      return;
    }

    if (audio) {
      audio.pause();
    }
    audio = new Audio(tracks[index].songUrl);
    audioElements[index] = audio;
    audio.play();
    currentTrackIndex = index;
    isPlaying = true;

    if (tracks[index].label === "Bass") {
      onBassPlay();
    }

    audio.addEventListener('ended', () => {
      isPlaying = false;
      currentTrackIndex = null;
    });

    window.dispatchEvent(new CustomEvent('audioRefsUpdate', { 
      detail: audioElements.filter(a => a !== null)
    }));
  }

  function skipTrack() {
    if (currentTrackIndex !== null && currentTrackIndex < tracks.length - 1) {
      playTrack(currentTrackIndex + 1); // Play next track
    }
  }

  onMount(() => {
    return () => {
      audioElements.forEach(audio => {
        if (audio) {
          audio.pause();
          audio.currentTime = 0;
        }
      });
    };
  });
</script>

<style>
  .track-list {
    display: grid;
    gap: 0.75rem;
    padding: 0.75rem;
  }

  .track {
    background-color: rgba(31, 41, 55, 0.4);
    backdrop-filter: blur(8px);
    border: 1px solid rgba(75, 85, 99, 0.3);
    border-radius: 0.75rem;
    padding: 0.75rem 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: all 0.2s ease;
  }

  .track:hover {
    background-color: rgba(31, 41, 55, 0.5);
    border-color: rgba(75, 85, 99, 0.4);
  }

  .track.active {
    background-color: rgba(31, 41, 55, 0.6);
    border-color: rgba(234, 179, 8, 0.3);
  }

  .track-label {
    font-size: 0.875rem;
    font-weight: 500;
    color: rgba(255, 255, 255, 0.9);
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .track-icon {
    opacity: 0.7;
  }

  .button-container {
    display: flex;
    gap: 0.5rem;
  }

  @media (max-width: 768px) {
    .track {
      padding: 0.625rem 0.875rem;
    }
  }
</style>

<div class="track-list">
  {#each tracks as track, index}
    <div class="track {currentTrackIndex === index ? 'active' : ''}">
      <div class="track-label">
        <span class="track-icon">
          {#if track.label === "Drums"}
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
              <path d="M5.25 3.375c0-1.036.84-1.875 1.875-1.875h9.75c1.036 0 1.875.84 1.875 1.875v17.25c0 1.035-.84 1.875-1.875 1.875h-9.75c-1.036 0-1.875-.84-1.875-1.875V3.375Z" />
              <path d="m12.75 15.75-3.75-3.75 3.75-3.75" />
            </svg>
          {:else if track.label === "Bass"}
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
              <path d="M19.952 1.651a.75.75 0 0 1 .298.599V16.303a3 3 0 0 1-2.176 2.884l-1.32.377a2.553 2.553 0 1 1-1.403-4.909l2.311-.66a1.5 1.5 0 0 0 1.088-1.442V6.994l-9 2.572v9.737a3 3 0 0 1-2.176 2.884l-1.32.377a2.553 2.553 0 1 1-1.402-4.909l2.31-.66a1.5 1.5 0 0 0 1.088-1.442V5.25a.75.75 0 0 1 .544-.721l10.5-3a.75.75 0 0 1 .658.122Z" />
            </svg>
          {:else if track.label === "Instrument"}
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
              <path fill-rule="evenodd" d="M19.952 1.651a.75.75 0 0 1 .298.599V16.303a3 3 0 0 1-2.176 2.884l-1.32.377a2.553 2.553 0 1 1-1.403-4.909l2.311-.66a1.5 1.5 0 0 0 1.088-1.442V6.994l-9 2.572v9.737a3 3 0 0 1-2.176 2.884l-1.32.377a2.553 2.553 0 1 1-1.402-4.909l2.31-.66a1.5 1.5 0 0 0 1.088-1.442V5.25a.75.75 0 0 1 .544-.721l10.5-3a.75.75 0 0 1 .658.122ZM9 7.422V4.3L3.28 5.78a.75.75 0 0 1-.944-.71V2.933A.75.75 0 0 1 2.69 2.18L9 0l7.5 1.5-7.5 2.25V7.422Z" clip-rule="evenodd" />
            </svg>
          {:else}
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5">
              <path fill-rule="evenodd" d="M2.25 12c0-5.385 4.365-9.75 9.75-9.75s9.75 4.365 9.75 9.75-4.365 9.75-9.75 9.75S2.25 17.385 2.25 12Zm14.024-.983a1.125 1.125 0 0 1 0 1.966l-5.603 3.113A1.125 1.125 0 0 1 9 15.113V8.887c0-.857.921-1.4 1.671-.983l5.603 3.113Z" clip-rule="evenodd" />
            </svg>
          {/if}
        </span>
        <span>{track.label}</span>
      </div>
      <div class="button-container">
        <Button
          variant="ghost"
          size="sm"
          class="hover:bg-gray-700/50 hover:text-white {currentTrackIndex === index && isPlaying ? 'text-yellow-400' : 'text-gray-300'}"
          on:click={() => playTrack(index)}
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="w-5 h-5"
          >
            {#if currentTrackIndex === index && isPlaying}
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M14.25 9v6m-4.5 0V9M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"
              />
            {:else}
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
            {/if}
          </svg>
        </Button>
        {#if currentTrackIndex === index}
          <Button
            variant="ghost"
            size="sm"
            class="hover:bg-gray-700/50 hover:text-white text-gray-300"
            on:click={skipTrack}
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="w-5 h-5"
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
