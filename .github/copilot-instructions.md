# ELUNO MIRROR - AI Agent Instructions

This project implements an automated matchmaking system for a PeerJS P2P card game. The system uses a central broker server to pair players automatically, replacing manual room creation/joining.

## Project Architecture

### Core Components
- **Broker Server** (`matchmaker.js`): Socket.io server that maintains a waiting queue and pairs players
- **Client Application** (`card4.html`): PeerJS-based card game with integrated matchmaking client
- **Deployment**: Designed for free hosting platforms (Glitch, Render, Heroku)

### System Flow
```
Player → Broker Queue → Match Found → P2P Connection → Direct Game Play
```

**Key Architecture Decisions:**
- **Stateless Broker**: Only handles matchmaking, no game state storage
- **Role Assignment**: First player in queue becomes Host, second becomes Guest
- **P2P Data Flow**: After matchmaking, game data flows directly between players
- **Dynamic Loading**: Socket.io loaded dynamically only when matchmaking starts

## Critical Development Workflows

### Local Development Setup
```bash
# Install dependencies
npm install

# Start broker server
npm start

# Development with auto-restart
npm run dev
```

### Testing Matchmaking
1. Start broker server
2. Open `card4.html` in two browser tabs
3. Click "FIND MATCH" in both tabs
4. System automatically pairs them and starts game

### Deployment Process
1. **Update Broker URL**: In `card4.html`, change `BROKER_URL` from localhost to deployed URL
2. **Deploy Broker**: Use Glitch (easiest), Render, or Heroku
3. **Share HTML**: Distribute updated `card4.html` to players
4. **No Server Needed**: Players just need the HTML file and internet connection

## Project-Specific Patterns & Conventions

### Configuration Management
- **Hardcoded URLs**: Broker URL is hardcoded in `card4.html` (line ~350)
- **Environment Variables**: Broker server uses `process.env.PORT` for deployment
- **No Config Files**: All configuration is inline for simplicity

### Error Handling Strategy
- **Broker Errors**: Displayed in UI with user-friendly messages
- **PeerJS Errors**: Logged to console, shown in matchmaking UI
- **Connection Timeouts**: 30-second queue timeout, 15-second connection timeout
- **Fallback States**: Always return to lobby on any failure

### UI State Management
Three main UI states in `card4.html`:
1. **Lobby Main**: Initial "FIND MATCH" button
2. **Searching**: Shows spinner, queue position, cancel button
3. **Connecting**: Shows match found, establishing P2P connection

### Network Event Flow
**Matchmaking Events:**
- `join_queue` → Client sends Peer ID
- `queue_update` → Server sends position updates
- `match_found` → Server assigns roles and opponent Peer ID
- `leave_queue` → Client cancels search

**Game Events (P2P):**
- `START_GAME` → Host sends initial game state
- `OPPONENT_MOVE` → Card play notification
- `OPPONENT_DRAW` → Draw card notification
- `MOON_ATTACK` → Special card effect
- `REQUEST_REMATCH` → Rematch request

### PeerJS Integration Pattern
```javascript
// Client-side pattern
peer = new Peer(undefined, { debug: 1 });
peer.on('open', (id) => socket.emit('join_queue', { peerId: id }));
peer.on('connection', (conn) => setupConnection());
```

### Socket.io Dynamic Loading
```javascript
// Only load when needed
if (typeof io === 'undefined') {
    const script = document.createElement('script');
    script.src = 'https://cdn.socket.io/4.7.2/socket.io.min.js';
    script.onload = () => initializeSocket();
}
```

## Integration Points & Dependencies

### External Dependencies
- **PeerJS**: P2P connection library (loaded from CDN in HTML)
- **Socket.io**: Real-time communication (loaded dynamically)
- **Express**: Web server framework
- **Socket.io Server**: WebSocket server for matchmaking

### Browser Requirements
- **WebRTC Support**: Required for P2P connections
- **AudioContext**: For sound effects (graceful degradation)
- **Canvas API**: For game rendering
- **Touch Events**: Mobile support

### Deployment Considerations
- **CORS**: Broker server allows all origins (`origin: "*"`)
- **Port**: Default 3000, uses `process.env.PORT` for production
- **Static Files**: Only `card4.html` needs to be served separately
- **No Database**: Queue is memory-only, no persistence needed

## Key Files & Their Roles

### `matchmaker.js` (161 lines)
- Express + Socket.io server
- Queue management with timeout cleanup
- Role assignment logic (Host/Guest)
- Health check endpoint

### `card4.html` (1494 lines)
- Complete game client with matchmaking integrated
- PeerJS connection management
- Canvas-based game rendering
- Audio management with sprite system
- Touch/mouse input handling

### `package.json`
- Minimal dependencies: express, socket.io
- Scripts: `start`, `dev` (nodemon)

### `README_MATCHMAKING.md`
- Comprehensive setup guide
- Deployment instructions for multiple platforms
- Architecture diagrams
- Troubleshooting guide

## Common Development Tasks

### Adding New Game Features
1. **Game Logic**: Modify `card4.html` game state management
2. **Network Events**: Add new event types to both client and server
3. **UI Updates**: Update lobby screens and game over states
4. **Testing**: Use two browser tabs for local testing

### Modifying Matchmaking
1. **Queue Logic**: Edit `waitingQueue` management in `matchmaker.js`
2. **Timeout Values**: Adjust `PLAYER_TIMEOUT` (currently 120000ms)
3. **Role Assignment**: Modify `matchPlayers()` function
4. **Error Handling**: Update socket event handlers

### Deploying to Production
1. **Broker**: Deploy `matchmaker.js` + `package.json` to hosting platform
2. **Client**: Update `BROKER_URL` in `card4.html`
3. **Distribution**: Share updated HTML file with players
4. **Monitoring**: Check `/health` endpoint for queue status

## Important Constants & Values

### Matchmaking Configuration
- `BROKER_URL`: Must be updated in `card4.html` for production
- `PLAYER_TIMEOUT`: 120000ms (2 minutes) queue timeout
- `PORT`: 3000 default, `process.env.PORT` for deployment

### Game Constants
- `SUITS`: ['fire', 'water', 'leaf', 'moon', 'mirror']
- `BASIC_SUITS`: ['fire', 'water', 'leaf']
- `COUNTER_MAP`: { fire: 'leaf', leaf: 'water', water: 'fire' }
- `CARD_RATIO`: 2.5 / 4.0

### Network Timeouts
- Socket.io connection: 10000ms
- PeerJS connection: 15000ms
- Queue cleanup: 10000ms interval

## Troubleshooting Patterns

### "Broker Offline" Error
- Check server is running: `curl http://localhost:3000/health`
- Verify `BROKER_URL` matches deployed server
- Check browser console for CORS errors

### Connection Failures
- Ensure WebRTC is supported and not blocked
- Check PeerJS CDN is accessible
- Verify both players have stable internet

### Matchmaking Issues
- Check broker logs for queue status
- Verify both players click "FIND MATCH" within 2 minutes
- Ensure no duplicate connections

## Security & Performance Notes

### Security
- **No sensitive data** passes through broker
- **Peer IDs** are temporary and session-specific
- **Game data** is P2P, broker only handles matchmaking
- **CORS** allows all origins (consider restricting in production)

### Performance
- **Memory-only** queue (no database)
- **Automatic cleanup** of expired connections
- **Minimal server** resource usage
- **P2P architecture** reduces server load

## Next Steps for AI Agents

When working on this codebase:
1. **Understand the flow**: Broker → Match → P2P → Game
2. **Test locally**: Always use two browser tabs
3. **Check deployment**: Verify broker URL is correct
4. **Monitor logs**: Use broker health endpoint
5. **Handle errors**: Graceful degradation is built-in

The system is designed for simplicity and reliability - prioritize these over complex features.