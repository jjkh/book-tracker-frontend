<script lang="ts">
  import { onMount } from "svelte";
  import { request } from "graphql-request";
  import Popup from "./Popup.svelte";

  interface SearchResults {
    findBooks: SearchResult[];
  }

  interface SearchResult {
    id: number;
    openLibraryId: string;
    title: string;
    author: string;
    publishYear: number;
    pageCount: number;
  }

  const query = `
    mutation {
      findBook(title: "art of electronics") {
        id
        openLibraryId
        title
        author
        publishYear
        pageCount
      }
    }
  `;
  let count: number = 0;
  let response: string = "not done!";
  let updateSize = false;

  onMount(() => {
    request<SearchResults>("http://localhost:8000/graph", query).then(
      (data) => {
        response = JSON.stringify(data, undefined, 4);
        updateSize = true;
      }
    );

    const interval = setInterval(() => count++, 1000);
    return () => {
      clearInterval(interval);
    };
  });
</script>

<Popup>
  <pre>{response}</pre>
</Popup>
<div class="App">
  <header class="App-header">
    <p>Page has been open for <code>{count}</code> seconds.</p>
  </header>
</div>

<style>
  :global(body) {
    margin: 0;
    font-family: Arial, Helvetica, sans-serif;
  }

  .App {
    background: #0002;
    padding: 4px 8px;
    border-radius: 4px;
  }

  .App p {
    margin: 0.4rem;
  }

  .App-header {
    background-color: #f9f6f6;
    color: #333;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
</style>
