<script lang="ts">
    import Popup from "$lib/Popup.svelte";
    import {onMount} from "svelte";

    let show_popup = $state(false);
    let entering_box_id = $state(-1);

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
                show_popup = true;
            } else if (jsonMsg.type === "boxEnterCancel") {
                entering_box_id = -1;
                show_popup = false;
            }
        }
    });
</script>

<!-- Box Recognition -->
<Popup show={show_popup} onClose={() => show_popup = false}>
    <div class="popup-centered">
        <h1>A new box is visible!</h1>
        <h3>Would you like to register?</h3>
        <p class="box-id">Box: {entering_box_id}</p>
        <div class="options">
            <button onclick={() => console.log("Yes")}>Yes</button>
            <button onclick={() => console.log("No")}>No</button>
        </div>
    </div>
</Popup>

<style>
    .popup-centered {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
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