<script>
    import Popup from "./Popup.svelte";

    let { shelf_tag, box_id, box_name, checkout_callback } = $props();
</script>

<div class="shelf-entry" data-augmented-ui="tl-clip br-clip border">
    <div class="text">
        <h2>{shelf_tag}</h2>
        <p>Contents: {(box_id === -1 ? "none" : "Box #"+box_id) + (box_name == "" ? "" : " ("+box_name+")")}</p>
    </div>

    <button onclick={checkout_callback}>Checkout</button>
</div>

<style>
    .text {
        display: flex;
        flex-direction: column;
    }

    h2 {
        margin: 0;
        color: var(--color-primary);
        font-size: 1rem;
        letter-spacing: 0.08em;
        text-transform: uppercase;
    }

    p {
        margin: 0;
        font-size: 0.8rem;
        color: var(--color-secondary);
        font-family: 'Roboto', sans-serif;
        letter-spacing: 0.03em;
    }

    .shelf-entry {
        height: 50px;
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        padding: 16px;
        background-color: var(--color-panel);
        --aug-tl: 14px;
        --aug-br: 14px;
        --aug-border-bg: var(--color-secondary);
        --aug-border-size: 1px;
        transition: --aug-border-bg 0.2s;
    }

    .shelf-entry:hover {
        --aug-border-bg: var(--color-primary);
    }

    .shelf-entry button {
        padding: 6px 18px;
        background-color: transparent;
        color: var(--color-primary);
        border: 1px solid var(--color-primary);
        cursor: pointer;
        font-family: 'Space Mono', monospace;
        font-size: 0.7rem;
        letter-spacing: 0.12em;
        text-transform: uppercase;
        transition: background-color 0.2s, color 0.2s, box-shadow 0.2s;
    }

    .shelf-entry button:hover {
        background-color: var(--color-primary);
        color: var(--color-bg);
        box-shadow: 0 0 12px rgba(102, 252, 241, 0.5);
    }
</style>
