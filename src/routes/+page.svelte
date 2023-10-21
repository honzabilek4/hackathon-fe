<script lang="ts">
  import { onMount } from "svelte";
  import io from "socket.io-client";

  $: data = [];

  const socket = io("ws://2dvkjqkl-3000.euw.devtunnels.ms");

  socket.on("update", (receivedData: any) => {	
		//@ts-ignore
    data = [...data, receivedData];
  });

  onMount(async () => {
    try {
      const response = await fetch(
        "https://2dvkjqkl-3000.euw.devtunnels.ms/transcribe"
      ); // Replace with your API endpoint
      if (response.ok) {
        data = await response.json();
      } else {
        throw new Error("Failed to fetch data");
      }
    } catch (error) {
      console.error(error);
    }
  });
</script>

<div class="container h-full mx-auto flex justify-center items-center">
  <div class="space-y-10 flex flex-col items-center">
    {#if data}
      <div>
        {#each data as item (item.id)}
          <p>[{item.timestamp}] {item.original} -> {item.translated}</p>
          <br />
        {/each}
      </div>
    {:else}
      <p>No data available.</p>
    {/if}
  </div>
</div>

<style lang="postcss">
  figure {
    @apply flex relative flex-col;
  }
  figure svg,
  .img-bg {
    @apply w-64 h-64 md:w-80 md:h-80;
  }
  .img-bg {
    @apply absolute z-[-1] rounded-full blur-[50px] transition-all;
    animation: pulse 5s cubic-bezier(0, 0, 0, 0.5) infinite,
      glow 5s linear infinite;
  }
  @keyframes glow {
    0% {
      @apply bg-primary-400/50;
    }
    33% {
      @apply bg-secondary-400/50;
    }
    66% {
      @apply bg-tertiary-400/50;
    }
    100% {
      @apply bg-primary-400/50;
    }
  }
  @keyframes pulse {
    50% {
      transform: scale(1.5);
    }
  }
</style>
