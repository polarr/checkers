<script>
    import io from 'socket.io-client';

    // Open WebSocket connection with the server to communicate moves
    const socket = io('http://localhost:3000');
    let socketId;

    socket.on("connect", ()=> {
        console.log(socket.id);
        socketId = socket.id;
    });

    socket.on('testing', ()=> {

    });
</script>

<div class = "wrapper">
    <nav class = "navbar">
        <span style = "margin-right: auto; font-weight: bold;">
            Online Checkers
        </span>
        <span>
            {socketId ? "User ID: " + socketId : "Not Connected"}
        </span>
    </nav>
    <slot {socket}></slot>
</div>

<style lang="scss">
    .wrapper {
        width: 100vw;
        height: 100vh;

        display: flex;
        flex-direction: column;
        align-items: stretch;

        font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    .navbar {
        background-color: #eee;
        padding: 1rem;

        display: flex;
        justify-content: right;
        
        > span + span {
            padding-left: 1rem;
        }
    }
</style>