<script>
    import Popup from "./Popup.svelte";

    let { shelf_tag, box_id, checkout_callback } = $props();
</script>

<div class="shelf-entry">
    <div class="text">
        <h2>{shelf_tag}</h2>
        <p>{box_id === -1 ? "none" : box_id}</p>
    </div>

    <button onclick={checkout_callback}>Checkout</button>
</div>

<style>
    .text {
        display: flex;
        flex-direction: column;
        text-align: center;
    }

    h2 {
        margin: 0;
    }
    p {
        margin: 0;
    }
    .shelf-entry {
        height: 50px;
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
        padding: 16px;
        border: 1px solid #000000;
        border-radius: 8px;
        box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
    }

    .shelf-entry h2 {
        margin-bottom: 8px;
    }

    .shelf-entry button {
        padding: 8px 16px;
        background-color: #007BFF;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    .shelf-entry button:hover {
        background-color: #0056b3;
    }
</style>