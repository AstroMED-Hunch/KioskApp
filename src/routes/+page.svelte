<script lang="ts">
    import Popup from "$lib/Popup.svelte";
    import {onMount} from "svelte";
    import { slide, blur } from "svelte/transition";

    let show_entered_popup = $state(false);
    let entering_box_id = $state(-1);
    let status = $state("idle");
    let show_entered_popup_v = $derived(show_entered_popup || status !== "idle");

    onMount(() => {
        let socket = new WebSocket("ws://localhost:3000");

        socket.onopen = () => {
            console.log("WebSocket connection established");
            socket.send(JSON.stringify({type: "kioskConnected"}));
        }

        socket.onmessage = (e) => {
            let jsonMsg = JSON.parse(e.data);
            console.log(jsonMsg);
            if (jsonMsg.type === "boxEnterConfirmation") {
                entering_box_id = jsonMsg.message;
                show_entered_popup = true;
            } else if (jsonMsg.type === "boxEnterCancel") {
                entering_box_id = -1;
                show_entered_popup = false;
            } else if (jsonMsg.type === "statusUpdate") {
                status = jsonMsg.message;
            }
        }
    });
</script>

<!-- Box Recognition -->
<Popup show={show_entered_popup_v} onClose={() => show_entered_popup = false}>
    <div class="popup-stack">
        {#key status || show_entered_popup}
            {#if show_entered_popup}
                <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                    <h1>A new box is visible!</h1>
                    <h3>Would you like to register?</h3>
                    <p class="box-id">Box: {entering_box_id}</p>
                    <div class="options">
                        <button onclick={() => console.log("Yes")}>Yes</button>
                        <button onclick={() => console.log("No")}>No</button>
                    </div>
                </div>
            {:else if status === "multiple_boxes"}
                <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                    <h1>Multiple boxes are visible!</h1>
                    <h3>Please adjust the camera to only show one box.</h3>
                </div>
            {/if}
        {/key}
    </div>
</Popup>

<style>
    .popup-stack {
        overflow: hidden;
        display: grid;
        grid-template-areas: "overlay";
    }
    .popup-stack > * {
        grid-area: overlay;
    }
    .popup-centered {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
        white-space: nowrap;
        min-width: max-content;
    }
    .options {
        display: flex;
        align-items: center;
        gap: 16px;
    }
    h1 {
        margin: 0;
        font-size: 1.5em;
    }
    h3 {
        margin: 0;
        font-size: 1em;
        color: #393939
    }
</style>