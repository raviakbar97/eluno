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
- **Host Authority**: Host generates game state and controls deck logic
- **Graceful Degradation**: Audio fails silently, network errors return to lobby

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

### Testing Broker Health
```bash
# Check if broker is running
curl http://localhost:3000/health

# Expected response:
# {"status":"ok","queueSize":0,"uptime":123.45}
```

## Project-Specific Patterns & Conventions

### Configuration Management
- **Hardcoded URLs**: Broker URL is hardcoded in `card4.html` (line ~370) as `BROKER_URL`
- **Environment Variables**: Broker server uses `process.env.PORT` for deployment (default: 3000)
- **No Config Files**: All configuration is inline for simplicity
- **Asset URLs**: All game assets loaded from GitHub raw URLs (https://github.com/viqihakbarrr-cyber/kkk/blob/main/...)

### Error Handling Strategy
- **Broker Errors**: Displayed in UI with user-friendly messages
- **PeerJS Errors**: Logged to console, shown in matchmaking UI
- **Connection Timeouts**: 120-second queue timeout, 15-second connection timeout
- **Fallback States**: Always return to lobby on any failure
- **Audio Failure**: Continues without sound if audio loading fails

### UI State Management
Three main UI states in `card4.html`:
1. **Lobby Main**: Initial "FIND MATCH" button with broker status
2. **Searching**: Shows spinner, queue position, cancel button
3. **Connecting**: Shows match found, establishing P2P connection

### Network Event Flow
**Matchmaking Events:**
- `join_queue` → Client sends Peer ID
- `queue_update` → Server sends position updates
- `match_found` → Server assigns roles and opponent Peer ID
- `leave_queue` → Client cancels search
- `timeout` → Server notification of queue timeout

**Game Events (P2P):**
- `START_GAME` → Host sends initial game state (deck, hands, top card)
- `OPPONENT_MOVE` → Card play notification with critical hit info
- `OPPONENT_DRAW` → Draw card notification
- `PARRY_SUCCESS` → Mirror card parry
- `REQUEST_REMATCH` → Rematch request
- `REMATCH_ACCEPTED` → Rematch accepted

### PeerJS Integration Pattern
```javascript
// Client-side pattern
peer = new Peer(undefined, { debug: 1 });
peer.on('open', (id) => socket.emit('join_queue', { peerId: id }));
peer.on('connection', (conn) => setupConnection());
peer.on('error', (err) => handleError());
```

### Socket.io Dynamic Loading
```javascript
// Only load when needed
if (typeof io === 'undefined') {
    const script = document.createElement('script');
    script.src = 'https://cdn.socket.io/4.7.2/socket.io.min.js';
    script.onload = () => initializeSocket();
    script.onerror = () => handleError();
}
```

### Host Authority Pattern
- **Host generates**: Deck, both hands, top card, turn order
- **Host validates**: All moves, draws, special effects
- **Guest receives**: Initial state, then reacts to host decisions
- **Critical hits**: Determined by host using COUNTER_MAP

### PeerJS Integration Pattern
```javascript
// Client-side pattern
peer = new Peer(undefined, { debug: 1 });
peer.on('open', (id) => socket.emit('join_queue', { peerId: id }));
peer.on('connection', (conn) => setupConnection());
peer.on('error', (err) => handleError());
```

### Socket.io Dynamic Loading
```javascript
// Only load when needed
if (typeof io === 'undefined') {
    const script = document.createElement('script');
    script.src = 'https://cdn.socket.io/4.7.2/socket.io.min.js';
    script.onload = () => initializeSocket();
    script.onerror = () => handleError();
}
```

### Host Authority Pattern
- **Host generates**: Deck, both hands, top card, turn order
- **Host validates**: All moves, draws, special effects
- **Guest receives**: Initial state, then reacts to host decisions
- **Critical hits**: Determined by host using COUNTER_MAP

## Integration Points & Dependencies

### External Dependencies
- **PeerJS**: P2P connection library (loaded from CDN in HTML, version 1.5.2)
- **Socket.io**: Real-time communication (loaded dynamically from CDN, version 4.7.2)
- **Express**: Web server framework (version ^4.18.2)
- **Socket.io Server**: WebSocket server for matchmaking (version ^4.7.2)

### Browser Requirements
- **WebRTC Support**: Required for P2P connections
- **AudioContext**: For sound effects (graceful degradation)
- **Canvas API**: For game rendering
- **Touch Events**: Mobile support
- **Modern Browser**: Chrome 60+, Firefox 55+, Safari 11+

### Deployment Considerations
- **CORS**: Broker server allows all origins (`origin: "*"`)
- **Port**: Default 3000, uses `process.env.PORT` for production
- **Static Files**: Only `card4.html` needs to be served separately
- **No Database**: Queue is memory-only, no persistence needed
- **Asset Hosting**: Game assets must be accessible via HTTPS (GitHub raw URLs)

## Key Files & Their Roles

### `matchmaker.js` (161 lines)
- Express + Socket.io server
- Queue management with timeout cleanup (120s)
- Role assignment logic (Host/Guest)
- Health check endpoint (`/health`)
- Periodic cleanup every 10 seconds
- Memory-only queue storage

### `card4.html` (1658 lines)
- Complete game client with matchmaking integrated
- PeerJS connection management (version 1.5.2)
- Canvas-based game rendering
- Audio management with sprite system (SFX_URL, SFX_SPRITES)
- Touch/mouse input handling
- Game state machine (MENU, PLAYING, GAMEOVER)
- **Current BROKER_URL**: `https://beagle-causal-gorilla.ngrok-free.app` (line 370)

### `package.json`
- Minimal dependencies: express ^4.18.2, socket.io ^4.7.2
- Scripts: `start` (node matchmaker.js), `dev` (nodemon matchmaker.js)
- No build process required

### `README_MATCHMAKING.md`
- Comprehensive setup guide
- Deployment instructions for multiple platforms
- Architecture diagrams
- Troubleshooting guide

### `deploy-instructions.txt`
- Quick reference for deployment
- Step-by-step platform instructions

### `Dockerfile`
- Empty file (not implemented)

## Common Development Tasks

### Adding New Game Features
1. **Game Logic**: Modify `card4.html` game state management
2. **Network Events**: Add new event types to both client and server
3. **UI Updates**: Update lobby screens and game over states
4. **Testing**: Use two browser tabs for local testing
5. **Host Authority**: Remember host generates all random elements

### Modifying Matchmaking
1. **Queue Logic**: Edit `waitingQueue` management in `matchmaker.js`
2. **Timeout Values**: Adjust `PLAYER_TIMEOUT` (currently 120000ms)
3. **Role Assignment**: Modify `matchPlayers()` function
4. **Error Handling**: Update socket event handlers
5. **Cleanup Interval**: Adjust 10000ms cleanup interval

### Deploying to Production
1. **Broker**: Deploy `matchmaker.js` + `package.json` to hosting platform
2. **Client**: Update `BROKER_URL` in `card4.html` (line ~350)
3. **Distribution**: Share updated HTML file with players
4. **Monitoring**: Check `/health` endpoint for queue status
5. **HTTPS**: Ensure broker uses HTTPS for secure connections

## Important Constants & Values

### Matchmaking Configuration
- `BROKER_URL`: Must be updated in `card4.html` (line 370) for production
- `PLAYER_TIMEOUT`: 120000ms (2 minutes) queue timeout
- `PORT`: 3000 default, `process.env.PORT` for deployment
- `CLEANUP_INTERVAL`: 10000ms (10 seconds) cleanup interval

### Game Constants
- `SUITS`: ['fire', 'water', 'leaf', 'mirror']
- `BASIC_SUITS`: ['fire', 'water', 'leaf']
- `COUNTER_MAP`: { fire: 'leaf', leaf: 'water', water: 'fire' }
- `CARD_RATIO`: 2.5 / 4.0
- `SFX_URL`: Audio sprite from GitHub raw URLs
- `ASSETS_URL`: Card images from GitHub raw URLs

### Network Timeouts
- Socket.io connection: 10000ms
- PeerJS connection: 15000ms
- Queue cleanup: 10000ms interval
- Matchmaking timeout: 15000ms (connection wait)

## Troubleshooting Patterns

### "Broker Offline" Error
- Check server is running: `curl http://localhost:3000/health`
- Verify `BROKER_URL` matches deployed server
- Check browser console for CORS errors
- Ensure server is listening on correct port

### Connection Failures
- Ensure WebRTC is supported and not blocked
- Check PeerJS CDN is accessible
- Verify both players have stable internet
- Check for firewall blocking WebRTC ports

### Matchmaking Issues
- Check broker logs for queue status
- Verify both players click "FIND MATCH" within 2 minutes
- Ensure no duplicate connections
- Check for expired Peer IDs

### Audio/Asset Loading Failures
- Verify GitHub raw URLs are accessible via HTTPS
- Check browser console for 404 errors
- Game continues without audio if loading fails

## Security & Performance Notes

### Security
- **No sensitive data** passes through broker
- **Peer IDs** are temporary and session-specific
- **Game data** is P2P, broker only handles matchmaking
- **CORS** allows all origins (consider restricting in production)
- **No authentication** - designed for casual play

### Performance
- **Memory-only** queue (no database)
- **Automatic cleanup** of expired connections
- **Minimal server** resource usage
- **P2P architecture** reduces server load
- **Dynamic loading** reduces initial page load

## Game-Specific Patterns

### Card Game Rules
- **Basic cards**: Fire, Water, Leaf (9 values each)
- **Special cards**: Mirror (parry)
- **Counter system**: Fire → Leaf → Water → Fire
- **Critical hits**: Playing counter to top card
- **Mirror effect**: Extra turn, parry opponent's move

### Host Responsibilities
- Generate deck (30 cards)
- Deal hands (6 cards each)
- Generate top discard
- Determine turn order
- Validate all moves
- Handle mirror card effects

### Network Data Flow
1. **Matchmaking**: Broker exchanges Peer IDs
2. **Game Start**: Host sends full state to guest
3. **Turns**: Players send moves to each other
4. **Effects**: Host calculates and broadcasts results
5. **Win Condition**: First to empty hand wins



### Deployment Considerations
- **CORS**: Broker server allows all origins (`origin: "*"`)
- **Port**: Default 3000, uses `process.env.PORT` for production
- **Static Files**: Only `card4.html` needs to be served separately
- **No Database**: Queue is memory-only, no persistence needed
- **Asset Hosting**: Game assets must be accessible via HTTPS (GitHub raw URLs)

## Common Development Tasks

### Adding New Game Features
1. **Game Logic**: Modify `card4.html` game state management
2. **Network Events**: Add new event types to both client and server
3. **UI Updates**: Update lobby screens and game over states
4. **Testing**: Use two browser tabs for local testing
5. **Host Authority**: Remember host generates all random elements

### Modifying Matchmaking
1. **Queue Logic**: Edit `waitingQueue` management in `matchmaker.js`
2. **Timeout Values**: Adjust `PLAYER_TIMEOUT` (currently 120000ms)
3. **Role Assignment**: Modify `matchPlayers()` function
4. **Error Handling**: Update socket event handlers
5. **Cleanup Interval**: Adjust 10000ms cleanup interval

### Deploying to Production
1. **Broker**: Deploy `matchmaker.js` + `package.json` to hosting platform
2. **Client**: Update `BROKER_URL` in `card4.html` (line ~350)
3. **Distribution**: Share updated HTML file with players
4. **Monitoring**: Check `/health` endpoint for queue status
5. **HTTPS**: Ensure broker uses HTTPS for secure connections

## Next Steps for AI Agents

When working on this codebase:
1. **Understand the flow**: Broker → Match → P2P → Game
2. **Test locally**: Always use two browser tabs
3. **Check deployment**: Verify broker URL is correct
4. **Monitor logs**: Use broker health endpoint
5. **Handle errors**: Graceful degradation is built-in
6. **Respect host authority**: All random elements generated by host
7. **Test edge cases**: Timeouts, disconnections, rematches

The system is designed for simplicity and reliability - prioritize these over complex features.