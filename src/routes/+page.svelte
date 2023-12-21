<script>
    import io from 'socket.io-client';

    // Open WebSocket connection with the server to communicate moves
    const socket = io('http://localhost:3000');

    socket.on('testing', ()=> {

    });
</script>