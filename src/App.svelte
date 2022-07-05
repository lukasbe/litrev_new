<script lang="ts">
  import { onMount } from "svelte";
  import Tree from "./Tree.svelte";
  import { fly } from 'svelte/transition';

  interface Aspect {
    ID: number;
    NAME: string;
    PARENT_ID: number | null;
  }

  interface Article{
    id: number;
    title: string;
    abstract: string;
    authors: string;
    conference: string;
    year: number;
    aspect_ids: string;
    aspect_names: string;
  }

  const aspectsHost = "https://litrev.abendstille.at/postgrest/aspects";
  let aspects: Aspect[];

  let errorMsg;
  let selectedNode;

  let articles: Article[];

  onMount(async () => await loadAspects());

  const loadAspects = async () => {
    try {
      const res = await fetch(aspectsHost);
      if (res.ok) {
        aspects = await (res.json() as Promise<Aspect[]>);
        return;
      }
      errorMsg = `An Error occurred while fetching aspects`;
    } catch (err) {
      console.error(err);
      errorMsg = `An Error occurred while fetching aspects`;
    }
  };

  const getTree = (aspects: Aspect[]) => {
    const topLevel = aspects.filter((a) => !a.PARENT_ID);
    const tree = topLevel.map((t) => {
      return getChildren(aspects, t.ID, true);
    });
    const root = {
      id: 0,
      label: "Aspects",
      children: tree,
    };
    return root;
  };

  const getChildren = (aspects: Aspect[], id, topLevel) => {
    const node = aspects.find((a) => a.ID == id);
    let children = null;
    if (aspects.find((a) => a.PARENT_ID == id)) {
      children = [];
      aspects
        .filter((a) => a.PARENT_ID == id)
        .forEach((a) => {
          children.push(getChildren(aspects, a.ID, false));
        });
    }
    const treeNode = {
      id: node.ID,
      label: node.NAME,
      children,
    };
    return treeNode;
  };

  const nodeSelect = async (e: CustomEvent) => {
    if(e.detail.id == 0) return;
    console.log(e.detail);
    selectedNode = e.detail;
    const fetchedArticles = await fetch('https://litrev.abendstille.at/postgrest/rpc/get_articles_for_aspect_as_json', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({p_aspect_id: selectedNode.id})
    });
    const artResp = await fetchedArticles.json();
    articles = artResp[0].j;
    console.log(articles);
  };
</script>


<main class:disabled={selectedNode}>
  {#if aspects}
    <Tree on:select={async(e) => await nodeSelect(e)} tree={getTree(aspects)} />
    {#if selectedNode && articles}
      <dialog open in:fly={{y: 200, duration: 500}} out:fly={{y: -200, duration: 500}}>
        <div class="nav">
          <button class="close" on:click={() => {
            selectedNode = null;
            articles = null;
            }}>x</button>
        </div>
        {selectedNode.label}
        <ul>
          {#each articles as article}
            <li class="article">
              {article.authors} ({article.year}). {article.title} <i class="conf">{article.conference}</i>
            </li>
          {/each}
        </ul>
      </dialog>
    {/if}
  {/if}
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }

  main{
    height: 100%;
  }

  .disabled {
    background-color: darkorange;
    background-image: linear-gradient(
      130deg,
      #ff7a18,
      #af002d 41.07%,
      #319197 76.05%
    );
    pointer-events: none;
  }


  dialog {
    position: fixed;
    width: 90vw;
    height: auto;
    margin: auto;
    top: 5%;
    pointer-events: all;
    max-height: 80vh;
    overflow-y: auto;
  }

  dialog .nav {
    display: flex;
    justify-content: end;
  }

  dialog .nav button {
    background: none;
    border: 0;
    cursor: pointer;
  }

  dialog li{
    margin-block: .6rem;
  }

  .close{
    font-size: larger;
    font-weight: 700;
  }

</style>
