<script>
  import { createEventDispatcher, onMount } from "svelte";

  export let value,
    required = true;

  let textarea;

  function handleKeyDown(event) {
    if ((event.metaKey || event.ctrlKey) && event.key === "Enter") {
      textarea.blur();
    }
  }  

  const dispatch = createEventDispatcher();
  let editing = false,
    original;

  onMount(() => {
    original = value;
  });

  function edit() {
    editing = true;
  }

  function submit() {
    if (value != original) {
      dispatch("submit", value);
      
    }

    editing = false;
  }

  function keydown(event) {
    if (event.key == "Escape") {
      event.preventDefault();
      value = original;
      editing = false;
    }
  }

  function focus(element) {
    element.focus();
  }
</script>

{#if editing}
  <form on:submit|preventDefault={submit} on:keydown={keydown} class="block">
    <textarea
      bind:this={textarea}
      on:keydown={handleKeyDown}
      class="text-2xl w-full border whitespace-pre border-gray-300 bg-transparent
      focus:border-blue-500 focus:ring focus:ring-blue-200 p-2 rounded-md outline-none"
      bind:value
      on:blur={submit}
      {required}
      use:focus
      autogrow
    />
  </form>
{:else}
  <div on:click={edit}>
    {value}
  </div>
{/if}

<style>
  input {
    border: none;
    background: none;
    font-size: inherit;
    color: inherit;
    font-weight: inherit;
    text-align: inherit;
    box-shadow: none;
  }
</style>
