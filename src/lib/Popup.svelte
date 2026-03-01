<script>
    import { scale, fade } from "svelte/transition";
    let { children, show, onClose } = $props();
</script>

{#if show}
    <div class="popup-backdrop" transition:fade={{duration: 300}} onclick={onClose} role="presentation"></div>
    <div class="popup-content" transition:scale={{ duration: 300 }}>
        {@render children()}
    </div>
{/if}

<style>
    .popup-backdrop {
        position: fixed;
        inset: 0;
        z-index: 40;
        opacity: 0.5;
        background: black;
    }

    .popup-content {
        position: fixed;
        z-index: 50;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        padding: 16px;

        background-color: #ffffff;
        border: 1px solid #000000;

        border-radius: 8px;
        box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
    }
</style>