<script lang="ts">
    import '../app.css';
    let { children } = $props();
</script>

<div class="app">
    <div class="gradient"></div>
    <main class="container">
        {@render children()}
    </main>
</div>

<style>
    .app {
        min-height: 100vh;
        background: var(--color-background);
        position: relative;
    }
    .gradient {
        position: absolute;
        inset: 0;
        background: var(--gradient-soft);
        pointer-events: none;
    }
    main.container {
        position: relative;
        z-index: 1;
        padding: 1.25rem 1rem 3rem;
    }
    @media (min-width: 768px) {
        main.container {
            padding: 2rem 1rem 4rem;
        }
    }
</style>
