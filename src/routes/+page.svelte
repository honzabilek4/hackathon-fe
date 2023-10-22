<script lang="ts">
  import { onMount } from "svelte";
  import io from "socket.io-client";
  // import { Table } from "@skeletonlabs/skeleton";
  import InPlaceEdit from "../components/InPlaceEdit.svelte";
  import { RadioItem, tableMapperValues } from "@skeletonlabs/skeleton";
  import type { TableSource } from "@skeletonlabs/skeleton";

  $: data = [];

  let tableSimple: TableSource;

  const socket = io("wss://2dvkjqkl-3000.euw.devtunnels.ms");

  socket.on("update", (receivedData: any) => {
    //@ts-ignore
    data = [...data, receivedData];
  });

  const getTranscript = async () => {
    try {
      const response = await fetch(
        "https://2dvkjqkl-3000.euw.devtunnels.ms/full-transcript"
      );
      if (response.ok) {
        data = await response.json();
      } else {
        throw new Error("Failed to fetch transcriptions");
      }
    } catch (error) {
      console.error(error);
    }
  };

  function updateItem(item) {
    console.log(item);
    try {
      fetch(`https://2dvkjqkl-3000.euw.devtunnels.ms/transcribe/${item.id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          translated: item.translated,
          original: item.original,
        }),
      });
    } catch (e) {
      console.log(e);
    }
  }

  function scrollToBottom() {
    const page = document.getElementById("page");
    page.scrollTo(0, page?.scrollHeight || 0);
    setTimeout(scrollToBottom, 500);
  }

  onMount(async () => {
    await getTranscript();
    tableSimple = {
      head: ["name", "original", "translated"],
      body: tableMapperValues(data, ["name", "original", "translated"]),
    };
		scrollToBottom();
  });
</script>

<div>
  <!-- {#if tableSimple}
    <Table source={tableSimple} interactive={true}/>
  {/if} -->

  <div class="container h-full mx-auto flex justify-center items-center">
    <div class="space-y-10 flex flex-col items-center text-2xl">
      {#if data}
        <div>
          {#each data as item (item.id)}
            <p>
              [{item.name}: {item.timestamp}]
              <InPlaceEdit
                class="w-full"
                bind:value={item.original}
                on:submit={() => updateItem(item)}
              />
              ---
              <InPlaceEdit class="w-full" bind:value={item.translated} />
              <br />
            </p>
          {/each}
        </div>
      {:else}
        <p>No data available.</p>
      {/if}
    </div>
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
