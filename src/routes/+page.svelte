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
    let currently_detected_face = $state("err_nodetect");
    let shelf_currently_checking_out = $state("");
    let connected = $state(false);

    let shelves_json = $state([]);

    type PillEntry = { pill_type: string; quantity: number };
    let pill_scan_result = $state<PillEntry[]>([]);
    let pill_scan_captured = $state(false);

    let sendRegisterRequest = () => {};
    let sendRegisterBoxExit = () => {};
    let sendCapturePills = () => {};
    let checkout_shelf = (shelf_tag: string) => {};
    
    let handleFaceRecContinue = () => {
        if (status === "face_recognition_boxentry") {
            sendRegisterRequest();
        } else if (status === "face_recognition_boxexit") {
            sendRegisterBoxExit();
        }
    }

    function connect() {
        let socket = new WebSocket("ws://localhost:3000");

        socket.onopen = () => {
            console.log("WebSocket connection established");
            connected = true;
            socket.send(JSON.stringify({type: "kioskConnected"}));
            sendRegisterRequest = () => {
                socket.send(JSON.stringify({type: "registerBox", message: entering_box_id}));
            }
            sendRegisterBoxExit = () => {
                socket.send(JSON.stringify({type: "registerBoxExit", message: shelf_currently_checking_out}));
            }
            sendCapturePills = () => {
                pill_scan_captured = false;
                pill_scan_result = [];
                socket.send(JSON.stringify({type: "capturePills"}));
            }
            checkout_shelf = (shelf_tag: string) => {
                shelf_currently_checking_out = shelf_tag;
                let confirm_checkout = confirm("Are you sure you want to check out the box at shelf "+shelf_tag+"?");
                 if (confirm_checkout) {
                    socket.send(JSON.stringify({type: "registerBoxExit", message: shelf_tag}));
                } else {
                    shelf_currently_checking_out = "";
                }
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
                if (jsonMsg.message !== "pill_checkup") {
                    pill_scan_result = [];
                    pill_scan_captured = false;
                }
            } else if (jsonMsg.type === "boxLocation") {
                shelf_location = jsonMsg.message;
                show_entered_popup = false;
                status = "location_received";
            } else if (jsonMsg.type === "boxUpdate") {
                fetch("http://localhost:8000/getShelves").then(res => res.json()).then(json => {
                    shelves_json = json.shelves;
                }).catch(err => console.error("Failed to fetch shelves:", err));
            } else if (jsonMsg.type === "faceRecognitionUpdate") {
                currently_detected_face = jsonMsg.message;
            } else if (jsonMsg.type === "pillScanResult") {
                pill_scan_result = jsonMsg.message;
                pill_scan_captured = true;
            }
        }

        socket.onclose = () => {
            console.log("WebSocket connection closed. Reconnecting in 3s...");
            connected = false;
            setTimeout(connect, 3000);
        }

        socket.onerror = (err) => {
            console.error("WebSocket error:", err);
            socket.close();
        }
    }

    onMount(() => {
        connect();

        fetch("http://localhost:8000/getShelves").then(res => res.json()).then(json => {
            // console.log("what is this guy yapping abt")
            shelves_json = json.shelves;
        }).catch(err => console.error("Failed to fetch shelves:", err));
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
            {:else if status === "face_recognition_boxentry" || status === "face_recognition_boxexit"}
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
                        <button onclick={handleFaceRecContinue}>Continue</button>
                    </div>
                    {/if}
                </div>
            {:else if status === "pill_checkup"}
                <div out:slide={{duration: 300, axis:"x"}} in:blur={{duration: 400}} class="popup-centered">
                    <h1>Pill Verification</h1>
                    {#if !pill_scan_captured}
                        <p class="pill-hint">Point the camera at the pills inside the box, then capture.</p>
                        <button onclick={() => sendCapturePills()}>Capture Pills</button>
                    {:else}
                        <div class="pill-results">
                            {#if pill_scan_result.length === 0}
                                <p class="pill-hint">No pills detected.</p>
                            {:else}
                                <table class="pill-table">
                                    <thead>
                                        <tr><th>Pill Type</th><th>Qty</th></tr>
                                    </thead>
                                    <tbody>
                                        {#each pill_scan_result as entry}
                                            <tr>
                                                <td>{entry.pill_type}</td>
                                                <td class="pill-qty">{entry.quantity}</td>
                                            </tr>
                                        {/each}
                                    </tbody>
                                </table>
                            {/if}
                        </div>
                        <div class="options">
                            <button onclick={() => sendCapturePills()}>Re-capture</button>
                            <button onclick={() => sendRegisterRequest()}>Confirm &amp; Register</button>
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

<div class="page-wrapper">
    <header class="site-header">
        <span class="header-label">STORAGE MANAGEMENT SYSTEM</span>
        <span class="header-divider"></span>
        <span class="header-sub">SHELF CONTROL INTERFACE</span>
        <span class="status-indicator" style="background-color: {connected ? 'var(--color-primary)' : 'red'};"></span>
    </header>

    <div class="shelf-storage">
        {#key shelves_json}
            {#each shelves_json as shelf}
                <ShelfEntry shelf_tag={shelf.tag} box_id={shelf.box_id} box_name={shelf.box_pretty_name} checkout_callback={() => {checkout_shelf(shelf.tag)}} />
            {/each}
        {/key}
    </div>
</div>

<style>
    .page-wrapper {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
        padding: 32px 48px;
        gap: 24px;
    }

    .site-header {
        display: flex;
        align-items: center;
        gap: 16px;
        padding-bottom: 16px;
        border-bottom: 1px solid var(--color-secondary);
        margin-bottom: 8px;
    }

    .header-label {
        font-family: 'Space Mono', monospace;
        font-size: 0.85rem;
        letter-spacing: 0.2em;
        color: var(--color-primary);
        text-transform: uppercase;
    }

    .header-divider {
        flex: 1;
        height: 1px;
        background: linear-gradient(to right, var(--color-secondary), transparent);
    }

    .header-sub {
        font-family: 'Space Mono', monospace;
        font-size: 0.65rem;
        letter-spacing: 0.15em;
        color: var(--color-secondary);
        text-transform: uppercase;
    }

    .shelf-storage {
        display: flex;
        flex-direction: column;
        gap: 12px;
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
        gap: 12px;
        white-space: nowrap;
        min-width: max-content;
    }

    .options {
        display: flex;
        align-items: center;
        gap: 16px;
        margin-top: 4px;
    }

    h1 {
        margin: 0;
        font-size: 1.4rem;
        color: var(--color-primary);
        letter-spacing: 0.06em;
        text-transform: uppercase;
    }

    h2 {
        margin: 0;
        font-size: 1.8rem;
        color: var(--color-primary);
        letter-spacing: 0.1em;
        text-shadow: 0 0 20px rgba(102, 252, 241, 0.6);
    }

    h3 {
        margin: 0;
        font-size: 0.9rem;
        color: var(--color-text);
        font-family: 'Roboto', sans-serif;
        font-weight: 400;
        letter-spacing: 0.03em;
    }

    .box-id {
        font-family: 'Space Mono', monospace;
        font-size: 0.85rem;
        color: var(--color-secondary);
        letter-spacing: 0.08em;
        margin: 0;
    }

    button {
        padding: 8px 24px;
        background-color: transparent;
        color: var(--color-primary);
        border: 1px solid var(--color-primary);
        cursor: pointer;
        font-family: 'Space Mono', monospace;
        font-size: 0.75rem;
        letter-spacing: 0.12em;
        text-transform: uppercase;
        transition: background-color 0.2s, color 0.2s, box-shadow 0.2s;
    }

    button:hover {
        background-color: var(--color-primary);
        color: var(--color-bg);
        box-shadow: 0 0 16px rgba(102, 252, 241, 0.45);
    }
    
    .status-indicator {
        width: 10px;
        height: 10px;
        border-radius: 50%;
        display: inline-block;
        margin-left: 10px;
    }

    .mono-highlight {
        font-family: 'Space Mono', monospace;
        color: var(--color-primary);
        font-weight: 700;
    }

    .pill-hint {
        color: var(--color-secondary);
        font-size: 0.8rem;
        margin-bottom: 8px;
        text-align: center;
    }

    .pill-results {
        width: 100%;
        max-height: 200px;
        overflow-y: auto;
        margin-bottom: 12px;
        border: 1px solid var(--color-secondary);
        padding: 8px;
    }

    .pill-table {
        width: 100%;
        border-collapse: collapse;
        font-size: 0.8rem;
    }

    .pill-table th {
        text-align: left;
        border-bottom: 1px solid var(--color-secondary);
        padding: 4px;
        color: var(--color-secondary);
        font-family: 'Space Mono', monospace;
    }

    .pill-table td {
        padding: 4px;
        color: var(--color-text);
        border-bottom: 1px solid rgba(102, 252, 241, 0.1);
    }

    .pill-qty {
        text-align: right;
        font-family: 'Space Mono', monospace;
    }
</style>
