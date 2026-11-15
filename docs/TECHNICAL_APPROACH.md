# Technical Approach

## Architecture Philosophy

TerraScope follows a **client-server architecture** with clear separation between mobile frontend, backend API, and satellite data processing. The design prioritizes:

1. **Simplicity**: Minimal dependencies, straightforward data flow
2. **Scalability**: Backend can serve many concurrent users
3. **Cost-Effectiveness**: Leveraging free/open-source tools where possible
4. **Maintainability**: Clean code structure, standard patterns

---

## System Components

### 1. Mobile Application (Flutter)

**Role**: User interface and field interaction

**Technology**: Flutter 3.0+ / Dart

**Responsibilities**:
- User authentication UI
- Interactive map interface for field selection
- GPS coordinate collection
- HTTP API communication with backend
- Results visualization and dashboard
- Local state management

**Why Flutter**:
- Single codebase for Android (and future iOS)
- Native performance with compiled Dart
- Rich ecosystem for mapping (google_maps_flutter)
- Excellent geolocation support (geolocator)
- Beautiful, customizable UI components
- Hot reload for rapid iteration

**Key Dependencies**:
```yaml
dependencies:
  flutter: sdk
  supabase_flutter: ^2.5.6      # Auth & database
  http: ^1.2.0                  # API calls
  google_maps_flutter: ^2.6.0   # Mapping
  geolocator: ^10.1.0           # GPS location
  provider: ^6.1.5              # State management
  shared_preferences: ^2.5.4    # Local storage
  google_fonts: ^8.0.1          # Typography
  flutter_animate: ^4.5.2       # Animations
  lottie: ^3.1.0                # Loading animations
```

---

### 2. Backend API (FastAPI)

**Role**: Satellite data processing orchestration

**Technology**: FastAPI (Python)

**Hosting**: Render.com (https://terrascope-frontend.onrender.com)

**Responsibilities**:
- Receive field coordinates from mobile app
- Interface with Google Earth Engine
- Query and filter Sentinel-2 imagery
- Calculate vegetation indices (NDVI, NDMI)
- Generate visualization URLs
- Return analysis results

**Why FastAPI**:
- High performance async framework
- Automatic API documentation (OpenAPI/Swagger)
- Easy integration with Earth Engine Python client
- Type safety with Pydantic
- Modern Python 3.7+ features (async/await)

**Key Endpoints**:

```python
POST /analyze-field
  Body: {
    "point": {"lat": float, "lng": float},
    "crop_type": string
  }
  Response: {
    "ndvi": {"value": float},
    "ndmi": {"value": float},
    "image": string (URL),
    "timestamp": string
  }

POST /analyze-nir-image
  Body: multipart/form-data with image file
  Response: {
    "analysis": {...},
    "error": string (if any)
  }
```

**Dependencies**:
```python
fastapi
uvicorn              # ASGI server
earthengine-api      # Google Earth Engine client
python-multipart     # File upload handling
pydantic            # Data validation
```

---

### 3. Authentication & Database (Supabase)

**Role**: User management and profile storage

**Technology**: Supabase (PostgreSQL + Auth)

**Hosting**: Supabase Cloud (https://sifzjfzzkdzjvnjgzjwi.supabase.co)

**Responsibilities**:
- User registration and login
- Password hashing and session management
- Email verification
- Profile data storage
- Row-level security (RLS) policies

**Why Supabase**:
- Open-source (self-hostable if needed)
- Built-in authentication (reduces development time)
- PostgreSQL-based (reliable, familiar)
- Row Level Security for data privacy
- Real-time capabilities for future features
- Generous free tier

**Schema**:

```sql
-- Users table (managed by Supabase Auth)
auth.users (
  id uuid PRIMARY KEY,
  email text,
  encrypted_password text,
  email_confirmed_at timestamp,
  created_at timestamp,
  user_metadata jsonb
)

-- Profiles table (application-specific)
public.profiles (
  id uuid PRIMARY KEY REFERENCES auth.users(id),
  full_name text,
  phone_number text,
  district text,
  created_at timestamp DEFAULT now()
)

-- Future tables
public.fields (...)
public.analyses (...)
```

**Security**:
- Anon key used in app has RLS restrictions
- Service role key kept on backend only
- Password hashing via bcrypt (Supabase default)
- TLS/SSL for all connections

---

### 4. Satellite Data Processing (Google Earth Engine)

**Role**: Satellite imagery analysis

**Technology**: Google Earth Engine (JavaScript/Python API)

**Access**: Earth Engine API via FastAPI backend

**Responsibilities**:
- Query Sentinel-2 image collection
- Filter by date, location, cloud cover
- Select appropriate spectral bands
- Calculate vegetation indices
- Generate color-coded visualizations
- Provide tile URLs for image display

**Why Earth Engine**:
- Planetary-scale satellite data archive
- Pre-processed Sentinel-2 imagery
- Built-in functions for index calculation
- Cloud-based processing (no local compute needed)
- Free for research/non-commercial use
- Actively maintained by Google

**Data Used**:

**Sentinel-2 Level 2A (Surface Reflectance)**
- Collection: `COPERNICUS/S2_SR_HARMONIZED`
- Bands:
  - B4 (Red): 665 nm, 10m resolution
  - B8 (NIR): 842 nm, 10m resolution
  - B11 (SWIR): 1610 nm, 20m resolution
  - QA60: Cloud mask band

**Processing Pipeline**:

```python
# Pseudocode (actual implementation on backend)
def analyze_field(lat, lng):
    # 1. Define area of interest
    point = ee.Geometry.Point([lng, lat])
    region = point.buffer(500)  # 500m radius
    
    # 2. Filter image collection
    images = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
              .filterBounds(region)
              .filterDate(ee.Date.now().advance(-60, 'day'), ee.Date.now())
              .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
              .sort('CLOUDY_PIXEL_PERCENTAGE'))
    
    # 3. Select best image
    image = images.first()
    
    # 4. Calculate indices
    ndvi = image.normalizedDifference(['B8', 'B4'])
    ndmi = image.normalizedDifference(['B8', 'B11'])
    
    # 5. Extract values at point
    ndvi_value = ndvi.reduceRegion(
        reducer=ee.Reducer.mean(),
        geometry=point,
        scale=10
    ).get('nd').getInfo()
    
    ndmi_value = ndmi.reduceRegion(
        reducer=ee.Reducer.mean(),
        geometry=point,
        scale=10
    ).get('nd').getInfo()
    
    # 6. Generate visualization
    vis_params = {
        'min': -0.2, 'max': 0.8,
        'palette': ['red', 'yellow', 'green']
    }
    map_id = ndvi.visualize(**vis_params).getMapId()
    image_url = map_id['tile_fetcher'].url_format
    
    return {
        'ndvi': ndvi_value,
        'ndmi': ndmi_value,
        'image_url': image_url
    }
```

**Limitations**:
- Depends on Sentinel-2 revisit schedule (5-10 days)
- Cloud cover prevents analysis during monsoon
- Processing time varies (typically 3-10 seconds)
- Free tier quota limits (10,000 requests/day for non-commercial)

---

### 5. Geocoding (OpenStreetMap Nominatim)

**Role**: Convert village names to coordinates

**Technology**: Nominatim API (OpenStreetMap)

**Access**: Direct HTTP calls from mobile app

**Responsibilities**:
- Convert "Village, District, State" to lat/lng
- Support for Satbara-based field lookup

**Why Nominatim**:
- Free and open-source
- No API key required (usage policy compliance)
- Good coverage of Indian villages
- Simple REST API

**Usage**:

```dart
final query = Uri.encodeComponent('$village, Maharashtra, India');
final url = Uri.parse(
  'https://nominatim.openstreetmap.org/search?q=$query&format=json&limit=1'
);
final response = await http.get(url, headers: {
  'User-Agent': 'TerrascopeApp/1.0'  // Required by OSM policy
});
final data = jsonDecode(response.body) as List;
if (data.isNotEmpty) {
  final lat = double.parse(data[0]['lat']);
  final lng = double.parse(data[0]['lon']);
  // Center map on this location
}
```

**Limitations**:
- Returns village center, not survey-specific boundaries
- Requires accurate spelling
- No validation of survey numbers
- Rate-limited (1 request/second)

---

## Data Flow

### Complete User Journey

```
1. USER: Opens app
   ↓
2. APP: Checks Supabase auth session
   ↓
3. APP: If logged in, shows main navigation; else shows login
   ↓
4. USER: Navigates to field selection
   ↓
5. APP: Loads Google Maps, requests GPS permission
   ↓
6. USER: Selects field using point/boundary/satbara method
   ↓
7. APP: Collects coordinates (LatLng object)
   ↓
8. USER: Selects crop type, taps "Analyze"
   ↓
9. APP: Shows loading page with animation
   ↓
10. APP: Sends POST request to FastAPI backend
    Payload: {"point": {"lat": X, "lng": Y}, "crop_type": "rice"}
    ↓
11. BACKEND: Receives request
    ↓
12. BACKEND: Calls Google Earth Engine API
    ↓
13. EARTH ENGINE: Queries Sentinel-2 collection
    ↓
14. EARTH ENGINE: Filters by date, location, clouds
    ↓
15. EARTH ENGINE: Calculates NDVI and NDMI
    ↓
16. EARTH ENGINE: Generates visualization map tiles
    ↓
17. EARTH ENGINE: Returns index values and image URL to backend
    ↓
18. BACKEND: Formats response JSON
    Response: {
      "ndvi": {"value": 0.65},
      "ndmi": {"value": 0.42},
      "image": "https://earthengine.googleapis.com/v1alpha/...",
      "timestamp": "2026-11-13T22:00:00Z"
    }
    ↓
19. APP: Receives response, dismisses loading page
    ↓
20. APP: Calculates health scores:
    - Crop Health = ((0.65 - 0.15) / 0.70) * 100 = 71
    - Soil Moisture = ((0.42 - 0.03) / 0.62) * 100 = 63
    ↓
21. APP: Navigates to Dashboard Page
    ↓
22. APP: Displays:
    - Overall Score: 67 (average)
    - Crop Health: 71 (green)
    - Soil Moisture: 63 (green)
    - Satellite image with NDVI overlay
    - Insights and recommendations
    ↓
23. USER: Views results, optionally shares or saves
```

### Error Handling

**Scenario 1: No Recent Cloud-Free Imagery**
```
Backend → App: {"error": "no_clear_images", "message": "Cloud cover too high"}
App → User: "Satellite analysis unavailable due to cloud cover. Please try again in a few days."
```

**Scenario 2: Network Failure**
```
App HTTP Request: Timeout after 30 seconds
App → User: "Network error. Please check your connection and try again."
```

**Scenario 3: Invalid Coordinates**
```
Backend → App: {"error": "invalid_location", "message": "Location outside coverage area"}
App → User: "Selected location is outside the coverage area."
```

---

## Security Architecture

### 1. Authentication Flow

```
User enters email/password
  ↓
App → Supabase: signInWithPassword()
  ↓
Supabase: Verifies credentials
  ↓
Supabase → App: Returns session token (JWT)
  ↓
App: Stores token in secure storage (flutter_secure_storage if used, else SharedPreferences)
  ↓
All subsequent API calls include token in Authorization header
```

### 2. Data Privacy

**Personal Data**:
- Stored in Supabase with Row Level Security (RLS)
- Each user can only access their own profile
- Admin queries require service role key (not in app)

**Location Data**:
- GPS coordinates sent to backend for analysis only
- Not stored permanently in database (future feature)
- No location tracking or history logging

**API Keys**:
- Supabase anon key: Rate-limited, RLS-protected (safe for client-side use)
- Earth Engine credentials: On backend only, not exposed
- Google Maps API key: Restricted to Android app package name

### 3. Network Security

- All API calls over HTTPS/TLS
- Certificate pinning not implemented (standard trust chain)
- No sensitive data in URL parameters (POST body only)

---

## Performance Considerations

### Mobile App Optimization

**Map Performance**:
- Tile caching handled by google_maps_flutter plugin
- Polygon simplification for large boundaries (future improvement)
- Lazy loading of map markers

**Image Loading**:
- Satellite images loaded from Earth Engine tile server
- No local caching (network-dependent)
- Loading placeholder shown during image fetch

**State Management**:
- Provider for lightweight state (language, theme)
- No complex state tree (simple app structure)
- Stateful widgets for local UI state only

### Backend Optimization

**Earth Engine Query**:
- Pre-filtered collections (date, clouds) reduce processing
- Mean reducer for point extraction (fast)
- Visualization parameters optimized for web display
- No per-pixel processing (uses built-in functions)

**API Response**:
- Minimal JSON payload (~200 bytes)
- Image URL only (no base64 embedding)
- Asynchronous request handling (FastAPI async)

**Hosting**:
- Render.com free tier (cold starts possible)
- Auto-scaling not configured (low traffic expected in MVP)

---

## Scalability Path

### Current Limits

- **Concurrent Users**: ~50-100 (Render free tier, single instance)
- **Daily Analysis Requests**: ~1,000 (Earth Engine free tier quota)
- **Supabase Storage**: Minimal (profiles only, no heavy data)

### Scaling Strategy (When Needed)

**Phase 1: Vertical Scaling**
- Upgrade Render instance to paid tier (more CPU/memory)
- Increase Earth Engine quota (paid tier: $0.20/1000 requests)
- Optimize queries (reduce Earth Engine compute units)

**Phase 2: Horizontal Scaling**
- Deploy multiple backend instances behind load balancer
- Implement Redis caching for repeated field queries
- Database read replicas for profile data

**Phase 3: Architectural Evolution**
- Message queue (RabbitMQ/SQS) for asynchronous analysis
- Separate worker processes for Earth Engine queries
- CDN for static assets and satellite image tiles
- Microservices split (auth, analysis, dashboard services)

---

## Development Workflow

### Local Development Setup

1. **Mobile App**:
   ```bash
   flutter pub get
   flutter run --debug
   # Hot reload during development
   ```

2. **Backend** (hypothetical local setup):
   ```bash
   pip install -r requirements.txt
   earthengine authenticate  # One-time setup
   uvicorn main:app --reload
   ```

### Testing Strategy

**Current Status**: No automated tests implemented

**Planned Testing**:
- **Unit Tests**: Coordinate validation, score calculation
- **Widget Tests**: UI component rendering
- **Integration Tests**: End-to-end field selection → analysis flow
- **Manual Testing**: Physical device testing with real GPS

### Deployment

**Mobile App**:
```bash
# Android Release Build
flutter build apk --release
# Output: build/app/outputs/flutter-apk/app-release.apk
```

**Backend**:
- Git push to Render-connected repository
- Automatic deployment on push to main branch
- Environment variables configured in Render dashboard

---

## Technical Debt

### Known Issues

1. **Error Handling**: Generic error messages, limited retry logic
2. **Caching**: No offline support, repeated network requests
3. **Testing**: No automated test suite
4. **Documentation**: Code comments sparse in some areas
5. **Logging**: No structured logging or monitoring
6. **Release Build**: Currently using debug signing configuration

### Planned Improvements

1. Implement comprehensive error handling with user-friendly messages
2. Add SQLite local database for field history
3. Set up automated testing pipeline (unit + integration)
4. Configure proper release signing with app signing key
5. Add analytics and crash reporting (e.g., Sentry)
6. Implement Earth Engine quota monitoring

---

## Alternative Approaches Considered

### Why Not Direct Earth Engine from App?

**Considered**: Calling Earth Engine API directly from Flutter app

**Rejected Because**:
- Earth Engine requires authentication credentials (service account JSON)
- Embedding credentials in app is security risk
- Mobile SDKs not officially supported by Google
- Backend provides abstraction layer for future flexibility

**Chosen**: FastAPI backend as intermediary

### Why Not Firebase Instead of Supabase?

**Considered**: Firebase Auth + Firestore

**Reasons for Supabase**:
- Open-source (self-hostable if needed)
- PostgreSQL (more familiar SQL vs. NoSQL)
- More generous free tier for database
- Row Level Security out-of-the-box
- Real-time without vendor lock-in

**Trade-off**: Firebase has better mobile SDK maturity, but Supabase fit the project philosophy better

### Why Not Custom Map Implementation?

**Considered**: OpenStreetMap with flutter_map plugin

**Reasons for Google Maps**:
- Better satellite imagery quality
- Familiar user interface
- More robust Flutter plugin
- Better performance on low-end devices

**Trade-off**: Google Maps API costs after free tier, but MVP usage well within free quota

---

## Future Technical Enhancements

### Short Term
1. **Caching Layer**: Redis for repeated field queries
2. **Background Jobs**: Queue-based processing for bulk analysis
3. **Error Monitoring**: Sentry integration for crash reporting

### Medium Term
1. **ML Integration**: TensorFlow Lite for on-device crop classification
2. **Time-Series Analysis**: Historical NDVI trends stored in database
3. **API Versioning**: Proper versioning for backward compatibility

### Long Term
1. **Blockchain Integration**: Immutable audit trail for subsidy claims
2. **Edge Computing**: Local Sentinel-2 processing for government offices
3. **Multi-Cloud**: Hybrid deployment across cloud providers

---

This technical approach balances simplicity, cost, and scalability for an MVP while maintaining a clear path for future enhancement as user base and requirements grow.
