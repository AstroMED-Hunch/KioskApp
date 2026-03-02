<script>
    import { scale, fade } from "svelte/transition";
    let { children, show, onClose } = $props();
</script>

{#if show}
    <div class="popup-backdrop" transition:fade={{duration: 300}} onclick={onClose} role="presentation"></div>
    <div class="popup-content" transition:scale={{ duration: 300 }} data-augmented-ui="tl-clip tr-clip br-clip bl-clip border">
        {@render children()}
    </div>
{/if}

<style>
    .popup-backdrop {
        position: fixed;
        inset: 0;
        z-index: 40;
        background: rgba(11, 12, 16, 0.85);
    }

    .popup-content {
        position: fixed;
        z-index: 50;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        padding: 36px 48px;
        background-color: var(--color-panel);
        --aug-tl: 20px;
        --aug-tr: 20px;
        --aug-br: 20px;
        --aug-bl: 20px;
        --aug-border-bg: var(--color-primary);
        --aug-border-size: 1px;
        box-shadow:
            0 0 40px rgba(102, 252, 241, 0.12),
            inset 0 0 40px rgba(102, 252, 241, 0.04);
    }
</style>
