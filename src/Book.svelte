<script lang="ts">
    export let title: string;
    export let openLibraryId: string;
    export let coverId: string | null;
    export let author: string | null;
    export let publishYear: number | null;

    $: ol_uri = `https://openlibrary.org/works/${openLibraryId}`;
    $: image_uri = `https://covers.openlibrary.org/b/id/${coverId}-M.jpg`;
</script>

<div class="book">
    <h3><a href="{ol_uri}">{title} ({publishYear})</a></h3>
    {#if coverId}
        <img src="{image_uri}" width="150px" alt="{title}" />
    {:else}
        <b>No cover!</b>
    {/if}
    <h4>By {author}</h4>
</div>

<style>
    .book {
        background-color: rgba(0, 0, 0, 0.2);
        padding: 0 20px;
        width: 150px;
        text-align: center;
    }
    h3,
    h4 {
        overflow: hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2; /* number of lines to show */
        line-clamp: 2;
        -webkit-box-orient: vertical;
        align-items: stretch;
    }

    h3 {
        font-size: 1rem;
    }
    h4 {
        font-size: 0.8rem;
    }

    img {
        height: 200px;
    }
</style>
