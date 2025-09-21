# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is a Paris Metro-themed birthday party web application that allows guests to anonymously select a metro station. The application uses Node.js/Express backend with Socket.io for real-time updates.

## Key Commands

```bash
# Install dependencies
npm install

# Development (with auto-restart)
npm run dev

# Production
npm start
```

## Architecture

### Core Components
- **server.js**: Main Express server with Socket.io integration, handles station reservations with IP-based rate limiting
- **public/index.html**: Main user interface for station selection
- **public/admin.html**: Admin panel for managing reservations
- **public/debug.html & debug.js**: Debug interface for monitoring system state

### Data Flow
1. Client connects via WebSocket (Socket.io)
2. Server maintains in-memory state of:
   - `reservedStations` Map: station name → IP address
   - `ipReservations` Map: IP address → station name
3. Rate limiter enforces 1 selection per IP per 24h
4. Real-time updates broadcast to all connected clients

### Security & Rate Limiting
- Uses `helmet` for security headers
- `RateLimiterMemory` from `rate-limiter-flexible` for IP-based restrictions
- Admin endpoints protected by hardcoded key (`reset123` - should be changed in production)

## API Endpoints

### Admin Routes (require adminKey)
- `POST /api/release`: Release specific station
- `POST /api/reset-all`: Reset all reservations and rate limiter

### WebSocket Events
- `reserve-station`: Client attempts to reserve a station
- `station-reserved`: Broadcast when station is successfully reserved
- `request-stations`: Client requests current state

## Important Implementation Details

### Rate Limiter Reset Behavior
When stations are released or system is reset, the rate limiter memory is also cleared to allow users to make new selections immediately. This is handled in:
- `/api/release` endpoint: Clears rate limit for the IP that reserved the released station
- `/api/reset-all` endpoint: Completely resets the rate limiter instance

### Station List
The complete Paris Metro station list (300+ stations) is hardcoded in `metroStations` array in server.js:49-93. Includes all lines as of 2025.

### IP Detection
Server uses multiple header checks to determine real client IP (server.js:96-101):
- `x-forwarded-for`
- `x-real-ip`
- Direct connection address

## Development Notes

- Server runs on port 3000 by default (configurable via PORT env variable)
- WebSocket connections tracked in `connectedClients` Map
- All state is in-memory - restarting server clears all reservations
- Admin key is hardcoded as `reset123` - must be changed for production use