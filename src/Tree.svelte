<script context="module">
  // retain module scoped expansion state for each tree node
  const _expansionState = {
    /* treeNodeId: expanded <boolean> */
  };
</script>

<script>
  import { createEventDispatcher } from "svelte/internal";

  //	import { slide } from 'svelte/transition'
  export let tree;
  const { label, children, id } = tree;
  const dispatch = createEventDispatcher();

  let expanded = _expansionState[label] || false;
  const toggleExpansion = () => {
    console.log(children);
    expanded = _expansionState[label] = !expanded;
  };
  $: arrowDown = expanded;
</script>

<ul>
  <!-- transition:slide -->
  <li>
    {#if children}
      <span on:click={toggleExpansion}>
        <span class="arrow" class:arrowDown>&#x25b6</span>
      </span>
      <button
        on:click={() => {
          console.log("label: " + label);
          dispatch("select", { label, id });
        }}>{label}</button
      >
      {#if expanded}
        {#each children as child}
          <svelte:self
            on:select={(e) => dispatch("select", e.detail)}
            tree={child}
          />
        {/each}
      {/if}
    {:else}
      <span>
        <span class="no-arrow" />
        <button
          on:click={() => {
            console.log("label: " + label);
            dispatch("select", { label, id });
          }}>{label}</button
        >
      </span>
    {/if}
  </li>
</ul>

<style>

  button{
    background: none;
    border: 0;
    box-shadow: 2px 2px 2px 2px black;
    cursor: pointer;
  }

  button:active{
    box-shadow: 2px 2px 2px 2px gray;
  }

  ul {
    margin: 0;
    list-style: none;
    padding-left: 1.2rem;
    user-select: none;
  }
  .no-arrow {
    padding-left: 1rem;
  }
  .arrow {
    cursor: pointer;
    display: inline-block;
    /* transition: transform 200ms; */
  }
  .arrowDown {
    transform: rotate(90deg);
  }

  li {
    margin-block: 1.3rem;
  }
</style>
