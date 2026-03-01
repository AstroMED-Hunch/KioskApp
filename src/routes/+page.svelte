<script lang="ts">
    import Popup from "$lib/Popup.svelte";
    import {onMount} from "svelte";
    import { slide, blur } from "svelte/transition";
    import ShelfEntry from "$lib/ShelfEntry.svelte";

    let show_entered_popup = $state(false);
    let entering_box_id = $state(-1);
    let status = $state("idle");
    let shelf_location = $state("");
    let shelf_full_confirmed = $state(false);
    let show_entered_popup_v = $derived(show_entered_popup || status !== "idle");
    let currently_detected_face = $state("");

    let shelves_json = $state([]);

    let sendRegisterRequest = () => {};
    onMount(() => {
        let socket = new WebSocket("ws://localhost:3000");

        let shelves_response = fetch("http://localhost:8000/getShelves").then(res => res.json()).then(json => {
            console.log("what is this guy yapping abt")
            shelves_json = json.shelves;
        });

        socket.onopen = () => {
            console.log("WebSocket connection established");
            socket.send(JSON.stringify({type: "kioskConnected"}));
            sendRegisterRequest = () => {
                socket.send(JSON.stringify({type: "registerBox", message: entering_box_id}));
            }
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
                shelf_full_confirmed = false;
            } else if (jsonMsg.type === "boxLocation") {
                shelf_location = jsonMsg.message;
                show_entered_popup = false;
                status = "location_received";
            } else if (jsonMsg.type === "boxUpdate") {
                let shelves_response = fetch("http://localhost:8000/getShelves").then(res => res.json()).then(json => {
                    console.log("what is this guy yapping abt")
                    shelves_json = json.shelves;
                });
            } else if (jsonMsg.type === "faceRecognitionUpdate") {
                currently_detected_face = jsonMsg.message;
            }
        }
    });
</script>

<!-- Box Recognition Popups -->
<Popup show={show_entered_popup_v} onClose={() => show_entered_popup = false}>
    <div class="popup-stack">
        {#key status || show_entered_popup}
            {#if status === "location_received"}
            <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                <h1>Box Registered!</h1>
                <h3>Please Place the box at:</h3>
                <h2>{shelf_location}</h2>
                <button onclick={() => status = "idle"}>Okay!</button>
            </div>
            {:else if status === "face_recognition_boxentry"}
                <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                    <h1>{
                        currently_detected_face === "err_nodetect" ? "No face detected!" :
                            currently_detected_face === "err_multiple" ? "Multiple faces detected!" :
                                "Are you "+currently_detected_face+"?"
                    }</h1>
                    <h3>{
                        currently_detected_face === "err_nodetect" ? "Please ensure your face is visible to the camera." :
                            currently_detected_face === "err_multiple" ? "Please ensure only your face is visible to the camera." :
                                "If this is correct, please press continue."
                    }</h3>
                    {#if currently_detected_face !== "err_nodetect" && currently_detected_face !== "err_multiple"}
                    <div class="options">
                        <button onclick={() => sendRegisterRequest()}>Continue</button>
                    </div>
                    {/if}
                </div>
            {:else if status === "shelves_full" && !shelf_full_confirmed}
                <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                    <h1>All shelves are full!</h1>
                    <h3>Please wait for a shelf to be freed up.</h3>
                    <button onclick={() => {shelf_full_confirmed=true; show_entered_popup = false; status="idle"}}>Okay!</button>
                </div>
            {:else if show_entered_popup}
                <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                    <h1>A new box is visible!</h1>
                    <h3>Would you like to register?</h3>
                    <p class="box-id">Box: {entering_box_id}</p>
                    <div class="options">
                        <button onclick={() => sendRegisterRequest()}>Yes</button>
                        <button onclick={() => show_entered_popup = false}>No</button>
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

<div class="shelf-storage">
    {#key shelves_json}
        {#each shelves_json as shelf}
            <ShelfEntry shelf_tag={shelf.tag} box_id={shelf.box_id} box_name={shelf.box_pretty_name} checkout_callback={() => {}} />
        {/each}
    {/key}
</div>

<style>
    .shelf-storage {
        display: flex;
        flex-direction: column;
        gap: 16px;
        margin-right: 0;
    }

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
        color: #393939;
    }
</style>