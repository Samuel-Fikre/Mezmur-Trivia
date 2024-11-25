<script lang="ts">
  import { Button } from "$lib/components/ui/button/index.js";

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
      <Button
        variant="default"
        color={currentTrackIndex === index ? "success" : "primary"}
        on:click={() => playTrack(index)}
      >
        {currentTrackIndex === index ? "Playing..." : "Play"}
      </Button>
      {#if currentTrackIndex === index}
        <Button
          variant="default"
          color="secondary"
          on:click={skipTrack}
        >
          Skip
        </Button>
      {/if}
    </div>
  {/each}
</div>
