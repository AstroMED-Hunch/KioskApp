<script lang="ts">
    import Popup from "$lib/Popup.svelte";

    let show_popup = $state(false);

    const callback = () => {
        show_popup = true;
    }
</script>

<button onclick={callback}>test</button>
<Popup show={show_popup} onClose={() => show_popup = false}>
    <h1>Hello, World!</h1>
</Popup>