<script>
    import { fly, scale, fade } from 'svelte/transition';

    import path from '~/lib/router';

    export let route = undefined;
    export let entry = undefined;
    export let onboarding = undefined;
    export let home = undefined;
    export let menu = undefined;
    export let modal = undefined;
    export let transparent = undefined;

    $: matchedRoute = $path.split('?')[0] === route;
</script>

<style>
    div {
        position: absolute;
        top: 0px;
        left: 0px;
        width: 100%;
        height: 100%;
        background-color: transparent;
    }

    div.transparent {
        background: none;
    }
</style>

{#if matchedRoute}
    {#if entry}
        <div class:transparent in:scale="{{ duration: 380, start: 0.95, oapcity: 0.9 }}">
            <slot />
        </div>
    {:else if onboarding}
        <div class:transparent in:fly="{{ x: 360, duration: 280, opacity: 0 }}">
            <slot />
        </div>
    {:else if modal}
        <div class:transparent>
            <slot />
        </div>
    {:else if home}
        <div class:transparent>
            <slot />
        </div>
    {:else if menu}
        <div class:transparent transition:fly="{{ x: 360, duration: 280, opacity: 0 }}">
            <slot />
        </div>
    {/if}
{/if}
