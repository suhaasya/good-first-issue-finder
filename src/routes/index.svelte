<script lang="ts" context="module">
  import type { Load } from '@sveltejs/kit';

  export const load: Load = async ({ fetch, url }) => {
    const globalQuery = 'is:open label:"EddieHub:good-first-issue" no:assignee';
    const orgQuery = 'is:open label:"good first issue" org:EddieHubCommunity no:assignee';
    let globalParam = false;
    try {
      globalParam = JSON.parse(url.searchParams.get('global'));
    } catch (e) {
      globalParam = false;
    }
    let postBody: queryBody = null;

    if (!globalParam) postBody = { query: orgQuery };
    else {
      postBody = { query: globalQuery };
    }

    const res = await fetch('/api/get-issues', { method: 'POST', body: JSON.stringify(postBody) });

    if (res.ok) {
      const data = await res.json();
      return {
        props: {
          checked: globalParam,
          data: data as SearchResponse,
        },
      };
    }
    const data = await res.json();
    return {
      error: data.message,
    };
  };
</script>

<script lang="ts">
  import IssueCard from '$lib/components/issue-card.svelte';
  import Search from '$lib/components/search.svelte';
  import Toggle from '$lib/components/toggle.svelte';
  import Filter from '$lib/components/filter.svelte';
  import { selectedLabels } from '$lib/stores/selected-labels.store';
  import type { queryBody, SearchResponse } from '../global';
  import { goto } from '$app/navigation';
  export let data: SearchResponse;

  $: searchString = '';
  $: filteredLabels = data.edges;
  $: searchItems = data.edges;

  export let checked: boolean;

  $: $selectedLabels, filterLabels();
  const filterLabels = () => {
    filteredLabels = data.edges.filter((dataset) =>
      $selectedLabels.every((label) =>
        dataset.node.labels.edges.some((edge) => edge.node.name === label),
      ),
    );
  };

  const performSearch = () => {
    searchItems = data.edges.filter((el) =>
      el.node.title.toLowerCase().includes(searchString.toLowerCase()),
    );
  };

  $: intersectedArray = filteredLabels.filter((item) => searchItems.includes(item));

  const onChangeHandler = async () => {
    if (checked) {
      await goto('?global=true');
      performSearch();
      return;
    }
    await goto('?global=false');
    performSearch();
  };
</script>

<div class="mb-8 flex flex-col items-center justify-center">
  <div class="mb-4">
    <Toggle
      labelLeft="Organization"
      on:change={() => {
        onChangeHandler();
      }}
      bind:checked
      labelRight="Global"
      id="org-toggle"
    />
  </div>
  <Search bind:searchTerm={searchString} on:keyup={() => performSearch()} />
  <Filter tags={data.labels} />
</div>
{#if intersectedArray.length > 0}
  <div class="mb-4 space-y-4">
    {#each intersectedArray as node}
      <IssueCard issue={node.node} />
    {/each}
  </div>
{:else}
  <div class="text-center">Unfortuately there was no issue found</div>
{/if}
