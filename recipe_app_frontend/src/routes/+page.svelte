<script lang="ts">
    import Header from '../lib/components/Header.svelte';
    import SearchBar from '../lib/components/SearchBar.svelte';
    import RecipeGrid from '../lib/components/RecipeGrid.svelte';
    import RecipeDetail from '../lib/components/RecipeDetail.svelte';
    import { fetchRecipes } from '../lib/services/recipes';
    import type { Recipe } from '../lib/services/recipes';

    let search = '';
    let loading = true;
    let error: string | null = null;
    let allRecipes: Recipe[] = [];
    let filtered: Recipe[] = [];
    let selected: Recipe | null = null;

    async function load() {
        loading = true;
        error = null;
        try {
            allRecipes = await fetchRecipes();
            applyFilter();
        } catch (e) {
            error = e instanceof Error ? e.message : 'Failed to load recipes';
        } finally {
            loading = false;
        }
    }

    function applyFilter() {
        const q = search.trim().toLowerCase();
        if (!q) {
            filtered = allRecipes;
            return;
        }
        filtered = allRecipes.filter(r => {
            const hay = [
                r.title,
                r.description ?? '',
                (r.tags ?? []).join(' '),
                (r.ingredients ?? []).map(i => i.name).join(' ')
            ].join(' ').toLowerCase();
            return hay.includes(q);
        });
    }

    function onSearch(value: string) {
        search = value;
        applyFilter();
    }

    function openDetail(recipe: Recipe) {
        selected = recipe;
        // Focus will be handled inside modal
    }

    function closeDetail() {
        selected = null;
    }

    import { onMount } from 'svelte';
    onMount(() => {
        load();
    });
</script>

<svelte:head>
    <title>Recipe Explorer</title>
    <meta name="description" content="Browse and search recipes in a modern Svelte app" />
</svelte:head>

<div class="page">
    <Header />

    <section class="actions">
        <SearchBar value={search} on:change={(e) => onSearch(e.detail)} placeholder="Search recipes by name or ingredients..." />
    </section>

    {#if loading}
        <section class="state">
            <div class="skeleton hero"></div>
            <div class="grid sm-2 lg-3">
                {#each [...Array(6).keys()] as i (i)}
                    <div class="card skeleton" style="height: 260px;"></div>
                {/each}
            </div>
        </section>
    {:else if error}
        <section class="state">
            <div class="alert error">
                <strong>Failed to load recipes</strong>
                <p>{error}</p>
                <button class="btn" on:click={load}>Retry</button>
            </div>
        </section>
    {:else}
        {#if filtered.length === 0}
            <section class="state">
                <div class="empty card">
                    <div class="icon">🔎</div>
                    <h3>No results</h3>
                    <p>We couldn't find any recipes matching "<strong>{search}</strong>". Try different keywords.</p>
                </div>
            </section>
        {:else}
            <RecipeGrid {filtered} on:select={(e) => openDetail(e.detail)} />
        {/if}
    {/if}

    <RecipeDetail open={!!selected} recipe={selected} on:close={closeDetail} />
</div>

<style>
    .page {
        display: flex;
        flex-direction: column;
        gap: 1.25rem;
    }
    .actions {
        position: sticky;
        top: .5rem;
        z-index: 5;
        background: transparent;
        padding-bottom: .25rem;
    }
    .state {
        display: grid;
        gap: 1rem;
    }
    .hero {
        height: 120px;
    }
    .alert {
        padding: 1rem;
        border-radius: var(--radius-md);
        border: 1px solid rgba(17,24,39,0.12);
        background: white;
        box-shadow: var(--shadow-sm);
        display: grid;
        gap: .5rem;
        align-items: start;
    }
    .alert.error {
        border-color: rgba(239,68,68,0.3);
        background: linear-gradient(0deg, rgba(239,68,68,0.05), white);
    }
    .empty {
        text-align: center;
        padding: 2rem 1rem;
    }
    .empty .icon {
        font-size: 2rem;
        margin-bottom: .25rem;
    }
</style>
