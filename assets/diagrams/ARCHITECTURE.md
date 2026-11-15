# System Architecture Diagram

This file contains a text-based representation of the TerraScope system architecture. A visual diagram (PNG/SVG) will be added in future updates.

```
┌─────────────────────────────────────────────────────────────────┐
│                     TerraScope Mobile App                        │
│                      (Flutter + Dart)                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Auth Screen  │  │ Field Select │  │  Dashboard   │          │
│  │  (Login/     │→ │  (Map +      │→ │  (Results +  │          │
│  │  Register)   │  │  3 Modes)    │  │  Insights)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└────────────┬────────────────┬───────────────────────────────────┘
             │                │
             │ Supabase       │ HTTPS POST
             │ Auth           │ /analyze-field
             │                │
    ┌────────▼─────┐    ┌────▼─────────────────────────┐
    │   Supabase   │    │   FastAPI Backend            │
    │   Cloud      │    │   (Python + Uvicorn)         │
    │              │    │   terrascope-frontend        │
    │ • Auth       │    │   .onrender.com              │
    │ • Database   │    │                              │
    │ • RLS        │    │ • Coordinate validation      │
    └──────────────┘    │ • Earth Engine queries       │
                        │ • NDVI/NDMI calculation      │
                        │ • Visualization generation   │
                        └────────────┬─────────────────┘
                                     │
                                     │ Earth Engine
                                     │ Python API
                                     │
                        ┌────────────▼─────────────────┐
                        │   Google Earth Engine        │
                        │   (Cloud Processing)         │
                        │                              │
                        │ • ImageCollection filtering  │
                        │ • Band math (indices)        │
                        │ • Visualization              │
                        │ • Tile server URLs           │
                        └────────────┬─────────────────┘
                                     │
                                     │ Queries Sentinel-2
                                     │ Image Collection
                                     │
                        ┌────────────▼─────────────────┐
                        │   Sentinel-2 Satellites      │
                        │   (ESA Copernicus)           │
                        │                              │
                        │ • 10m spatial resolution     │
                        │ • 5-10 day revisit           │
                        │ • Multispectral bands        │
                        │ • Free & open data           │
                        └──────────────────────────────┘

Additional Services:
┌─────────────────────┐           ┌─────────────────────┐
│  Google Maps API    │           │ OpenStreetMap       │
│                     │           │ Nominatim API       │
│ • Base map tiles    │           │                     │
│ • Satellite imagery │           │ • Village geocoding │
│ • Location services │           │ • Address search    │
└─────────────────────┘           └─────────────────────┘
```

## Data Flow Sequence

### User Authentication
1. User enters credentials → App
2. App → Supabase Auth API
3. Supabase validates → Returns JWT token
4. App stores token for session

### Field Analysis Request
1. User selects field (point/boundary/satbara) → App
2. App collects GPS coordinates
3. App → POST /analyze-field → FastAPI Backend
4. Backend → Google Earth Engine API
5. Earth Engine queries Sentinel-2 collection
6. Earth Engine filters by date, location, clouds
7. Earth Engine calculates NDVI & NDMI
8. Earth Engine generates visualization
9. Earth Engine → Returns values + image URL → Backend
10. Backend → Returns JSON response → App
11. App displays dashboard with scores and imagery

### Map Display
1. User opens field selection → App
2. App → Google Maps SDK → Tile server
3. Map tiles rendered with satellite imagery
4. User location via GPS → Blue dot on map

### Village Lookup (Satbara Mode)
1. User enters village name → App
2. App → OpenStreetMap Nominatim API
3. Nominatim returns coordinates
4. App centers map on location

## Component Responsibilities

| Component | Technology | Primary Function |
|-----------|-----------|------------------|
| Mobile App | Flutter | User interface, coordinate collection, visualization |
| Supabase | PostgreSQL + Auth | User management, profile storage |
| FastAPI Backend | Python | API gateway, Earth Engine orchestration |
| Google Earth Engine | Cloud platform | Satellite image processing, index calculation |
| Sentinel-2 | ESA satellites | Raw multispectral imagery |
| Google Maps | Mapping service | Interactive maps, geocoding |
| OpenStreetMap | Community database | Village name geocoding |

## Network Protocols

- **App ↔ Supabase**: HTTPS REST + WebSocket (future)
- **App ↔ FastAPI**: HTTPS REST (JSON)
- **Backend ↔ Earth Engine**: gRPC (via Python client library)
- **App ↔ Google Maps**: HTTPS tile requests
- **App ↔ OpenStreetMap**: HTTPS REST (JSON)

## Security Layers

1. **Transport**: TLS 1.2+ for all connections
2. **Authentication**: JWT tokens with expiration
3. **Authorization**: Row Level Security (RLS) in Supabase
4. **API Keys**: 
   - Supabase anon key (client-side, RLS-protected)
   - Earth Engine credentials (backend-only)
   - Google Maps key (Android package-restricted)

## Deployment Architecture

```
User's Android Device
       ↓
    Internet
       ↓
   ┌───┴────────────────────────────┐
   │                                │
Supabase Cloud              Render.com (Backend)
(Database + Auth)           (FastAPI + Earth Engine)
   │                                │
   └────────┬───────────────────────┘
            ↓
    Google Earth Engine
  (Satellite Processing)
            ↓
     Sentinel-2 Archive
```

## Future Architecture Enhancements

### Phase 2: Government Dashboard
- Add separate web frontend (React/Vue)
- Shared backend for mobile + web
- Role-based access control (farmer vs. officer)

### Phase 3: Scalability
- Message queue (Redis/RabbitMQ) for async processing
- Multiple backend instances behind load balancer
- Database read replicas for profiles
- CDN for static assets

### Phase 4: Advanced Features
- ML models for crop classification (TensorFlow Serving)
- Time-series database for historical trends (InfluxDB)
- Real-time notifications (Firebase Cloud Messaging)
- Blockchain audit trail for subsidy claims

---

**Note**: A visual diagram (PNG/SVG format) showing this architecture with icons and color-coding will be created and added to this directory in a future update.
