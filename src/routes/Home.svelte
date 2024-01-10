<script lang="ts">
    import type { Socket } from "socket.io-client";

    export let socket: Socket;

    let time: string;
    let code: string;
    let created = false;
    let joinError = false;
    const createGame = () => {
        socket.emit("create-room", {time: parseInt(time)});
    };

    const joinGame = () => {
        socket.emit("join-room", {code});
    };

    socket?.on("created-room", async ({code})=> {
        created = code;
        await navigator.clipboard.writeText(code);
    });

    socket?.on("join-room-failure", ({error})=> {
        joinError = error;
    });
</script>

<article>
    <h1>Welcome to Online Checkers</h1>
    {#if !created}
    <div class = "create-join-wrapper">
        <div>
            <input bind:value={time} placeholder="Time Control (minutes)">
            <button on:click={createGame}>Create a game</button>
        </div>
        <div>
            <input bind:value={code} placeholder="Code">
            <button on:click={joinGame}>Join with code</button>
            <span style="color: red;">{joinError || ""}</span>
        </div>
    </div>
    {:else}
    <div style="text-align: center">
        Code copied to clipboard!<br>
        {created}
    </div>
    {/if}
</article>

<style lang="scss">
    article {
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;

        .create-join-wrapper {
            display: flex;
            justify-content: space-evenly;

            > div {
                display: flex;
                flex-direction: column;
                padding: 1rem;
                border: 1px solid black;
                text-align: center;

                input+button {
                    margin-top: 1rem;
                }
            }
        }
    }
</style>