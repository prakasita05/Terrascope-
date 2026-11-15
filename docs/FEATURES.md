# TerraScope Features Documentation

## Feature Overview

TerraScope provides a comprehensive set of features designed for agricultural field monitoring and management. This document details each feature's functionality, user workflow, and technical implementation.

---

## 1. User Authentication

### Description
Secure user registration and login system powered by Supabase Auth.

### Features
- Email and password-based authentication
- Secure session management
- Password reset capability (via Supabase)
- Persistent login sessions

### User Workflow
1. New users register with email, password, and basic profile information
2. Email verification (configured in Supabase)
3. Login with credentials
4. Automatic session restoration on app restart

### Technical Implementation
- **Provider**: Supabase Auth
- **Storage**: Session tokens in secure storage
- **Security**: TLS encryption, bcrypt password hashing (Supabase)
- **State Management**: StreamBuilder listening to auth state changes

### Profile Fields
- Full name
- Email address
- Phone number
- District
- User ID (auto-generated)

---

## 2. Field Selection & Registration

TerraScope's most distinctive feature: **three flexible methods** for field registration.

### 2.1 Point Selection Method

#### Description
Quick field registration by tapping a single point on the map.

#### Use Cases
- Small plots where precise boundary is unnecessary
- Quick reconnaissance monitoring
- Fields with uniform shape

#### User Workflow
1. Open field selection page
2. Ensure "Point" tab is selected
3. Tap anywhere on the map to mark field location
4. Selected point shows as green marker
5. Select crop type from dropdown
6. Tap "Analyze" to submit

#### Technical Details
- **Input**: Single LatLng coordinate
- **Map Marker**: Green pin at selected location
- **Backend Payload**: `{"point": {"lat": X, "lng": Y}, "crop_type": "..."}`

### 2.2 Boundary Drawing Method

#### Description
Draw precise field boundaries by tapping multiple points to create a polygon.

#### Use Cases
- Irregular field shapes
- Accurate area calculation needed
- Fields with distinct boundaries visible on satellite imagery

#### User Workflow
1. Select "Boundary" tab
2. Tap multiple points on map to trace field perimeter
3. Polygon automatically closes and fills with semi-transparent overlay
4. Tap "Clear Boundary" to restart if needed
5. Select crop type
6. Tap "Analyze" when boundary is complete

#### Technical Details
- **Input**: List of LatLng coordinates forming polygon
- **Visualization**: Polygon overlay with stroke and fill
- **Area Calculation**: Client-side using coordinate geometry
- **Backend Processing**: Centroid calculated from polygon for satellite query
- **Minimum Points**: 3 (enforced in UI)

### 2.3 Satbara-Based Method

#### Description
Field location using traditional Indian land records (Survey Number system).

#### Use Cases
- Farmers who know their survey number but not exact GPS location
- Official documentation alignment
- Large field identification

#### User Workflow
1. Select "Satbara" tab
2. Enter village name in text field
3. Enter survey number (e.g., "123", "45/2", "67A")
4. Tap "Locate Field" button
5. App geocodes village using OpenStreetMap Nominatim API
6. Map centers on village location with marker
7. User can then tap to mark exact field or proceed with village center
8. Select crop type
9. Tap "Analyze"

#### Technical Details
- **Geocoding API**: OpenStreetMap Nominatim
- **Query Format**: `"<Village>, Maharashtra, India"`
- **Fallback**: If geocoding fails, shows error and allows manual map selection
- **Future Enhancement**: Integration with official land records API for automatic boundary retrieval

#### Limitations
- Requires accurate village name spelling
- Survey number not validated (no official API integration yet)
- Returns village center, not survey-specific boundary
- Manual refinement may be needed

---

## 3. Crop Type Selection

### Description
User selects crop type to provide context for satellite analysis.

### Supported Crop Types
- **General / Mixed**: Default for mixed cropping or unspecified
- **Rice (Paddy)**: Water-intensive crop, high moisture expected
- **Wheat**: Rabi crop, moderate water requirements
- **Cotton**: Kharif crop, drought-sensitive

### Purpose
While current implementation doesn't apply crop-specific thresholds, this data:
- Enables future crop-specific health scoring
- Provides context for backend analysis improvements
- Supports aggregated analytics (e.g., district-wise rice health)

### Technical Implementation
- **Data Type**: Dropdown selection
- **Default**: "general"
- **Backend Field**: `crop_type` in analysis request payload

---

## 4. Satellite Analysis

### Description
Core feature that processes field coordinates through Google Earth Engine to calculate vegetation and moisture indices.

### Analysis Process

#### Step 1: Request Submission
- User selects field and taps "Analyze"
- Coordinates sent to FastAPI backend at `/analyze-field` endpoint
- Loading screen with animation displayed

#### Step 2: Backend Processing
1. Backend receives coordinates and crop type
2. Queries Google Earth Engine for recent Sentinel-2 imagery
3. Filters images by:
   - Date range (recent 30-60 days)
   - Cloud cover percentage (<20%)
   - Field location bounds
4. Selects best available image (least clouds, most recent)
5. Extracts relevant spectral bands (Red, NIR, SWIR)
6. Calculates NDVI: `(NIR - Red) / (NIR + Red)`
7. Calculates NDMI: `(NIR - SWIR) / (NIR + SWIR)`
8. Generates color-coded visualization using Earth Engine's `visualize()` function
9. Returns JSON response with index values and image URL

#### Step 3: Results Display
- App receives analysis results
- Navigates to dashboard page
- Displays health scores, colors, and satellite imagery

### Satellite Data Specifications

**Source**: Sentinel-2 Level 2A (Surface Reflectance)

**Bands Used**:
- Band 4 (Red): 665 nm wavelength
- Band 8 (NIR): 842 nm wavelength
- Band 11 (SWIR): 1610 nm wavelength

**Spatial Resolution**: 10m (B4, B8), 20m (B11)

**Temporal Resolution**: 5-10 day revisit time

**Coverage**: Global (TerraScope uses imagery over Indian subcontinent)

### Analysis Limitations

- **Cloud Cover**: Cloudy weather prevents analysis (requires clear imagery)
- **Temporal Lag**: Analysis reflects conditions from last clear satellite pass (may be 5-10 days old)
- **Small Fields**: Fields smaller than ~100m² may have accuracy issues due to pixel resolution
- **Edge Effects**: Field boundaries may include neighboring pixels

---

## 5. Field Health Dashboard

### Description
Visual presentation of satellite analysis results with health scores, imagery, and insights.

### Dashboard Components

#### 5.1 Overall Health Score
- **Calculation**: Average of crop health and soil moisture scores
- **Range**: 0-100
- **Color Coding**:
  - 🔴 Red (0-30): Poor condition, immediate attention needed
  - 🟡 Yellow (31-60): Moderate condition, monitoring required
  - 🟢 Green (61-100): Good condition, healthy field

#### 5.2 Crop Health Score (NDVI-based)
- **Source**: NDVI from satellite analysis
- **Conversion Formula**: `((NDVI - 0.15) / 0.70) × 100`, clamped to 0-100
- **Interpretation**:
  - High score: Dense, healthy vegetation
  - Low score: Sparse vegetation, stress, or bare soil
  - Negative: Water bodies, non-vegetated areas

#### 5.3 Soil Moisture Score (NDMI-based)
- **Source**: NDMI from satellite analysis
- **Conversion Formula**: `((NDMI - 0.03) / 0.62) × 100`, clamped to 0-100
- **Interpretation**:
  - High score: Adequate moisture, well-irrigated
  - Low score: Water stress, irrigation needed
  - Very low: Drought conditions

#### 5.4 Satellite Imagery Display
- **Type**: Color-coded NDVI visualization overlay
- **Colors**: 
  - Red/Brown: Low vegetation
  - Yellow: Moderate vegetation
  - Green: Dense healthy vegetation
- **Source**: Google Earth Engine-generated visualization URL
- **Interactivity**: Tap to view full-screen (planned)

#### 5.5 Insights Section

Provides interpretation and recommendations based on scores:

**Good Health (61-100)**:
- ✅ Vegetation density is healthy
- ✅ Moisture levels adequate
- 💡 Continue current practices

**Moderate Health (31-60)**:
- ⚠️ Some areas may need attention
- 💡 Consider irrigation if soil moisture is low
- 💡 Monitor for pest/disease signs

**Poor Health (0-30)**:
- 🚨 Field requires immediate attention
- 💡 Check for water stress, pest infestation, or disease
- 💡 Consider consulting agricultural extension officer

### Technical Implementation

**Score Normalization**: Raw NDVI/NDMI values mapped to 0-100 scale matching web dashboard formula

**Color Calculation**: Dynamic color interpolation based on score value

**Background**: Custom background image with semi-transparent overlay for visual appeal

**Animations**: Flutter Animate for smooth transitions

---

## 6. Interactive Mapping

### Description
Google Maps-based interface for field visualization and selection.

### Map Features

#### 6.1 Base Map
- **Provider**: Google Maps via `google_maps_flutter` plugin
- **Type**: Satellite hybrid (satellite imagery with street labels)
- **Controls**: Zoom, pan, rotate, tilt
- **Initial Center**: Pune, Maharashtra (18.5204°N, 73.8567°E)

#### 6.2 Location Services
- **Current Location**: Blue dot showing user's GPS position
- **Auto-center**: Map centers on user location on initial load
- **Permissions**: Runtime location permission request
- **Fallback**: Default to Pune if location permission denied

#### 6.3 Field Overlays
- **Point Selection**: Green marker pin
- **Boundary Drawing**: Polygon with green stroke and semi-transparent green fill
- **Satbara Location**: Orange marker at geocoded village

#### 6.4 Map Interactions
- **Tap**: Places marker or adds boundary point
- **Long Press**: (Reserved for future features)
- **Drag**: Pan map to explore
- **Pinch**: Zoom in/out

---

## 7. User Profile Management

### Description
View and manage user account information.

### Profile Features

#### 7.1 Profile Display
- Full name
- Email address
- Phone number
- District
- Account creation date (from Supabase metadata)

#### 7.2 Profile Data Source
1. **Primary**: Supabase `profiles` table (if exists)
2. **Fallback**: User metadata from authentication object
3. **Update**: Edit profile (planned feature)

#### 7.3 Sign Out
- Confirmation dialog before logout
- Session cleared from secure storage
- Redirect to login page
- All cached data retained for next login

---

## 8. Multi-Language Support

### Description
Localization framework for regional language support.

### Current Implementation
- **Framework**: Flutter's `flutter_localizations` and `intl` packages
- **Supported Locales**: English (en), Marathi (mr) - structure in place
- **Language Switcher**: Settings or profile page (to be activated)
- **Scope**: UI labels, error messages, dashboard text

### Localized Components
- Welcome screen text
- Field selection instructions
- Dashboard labels ("Crop Health", "Soil Moisture", etc.)
- Error messages
- Button labels

### Technical Details
- **ARB Files**: `lib/l10n/app_*.arb` files for each language
- **Code Generation**: `flutter gen-l10n` generates type-safe localization classes
- **Provider**: `LanguageProvider` manages current locale
- **Persistence**: Selected language saved in SharedPreferences

---

## 9. NIR Camera Upload (Experimental)

### Description
Upload near-infrared (NIR) camera images for analysis.

### Status
⚠️ **Experimental Feature**: This was an earlier development experiment exploring handheld NIR camera integration (Raspberry Pi NoIR camera). It is not a primary production feature.

### Functionality
- User uploads NIR image from device gallery
- Backend performs image validation
- If valid NIR specimen detected, analysis proceeds
- If invalid, error message shown

### Use Case
This feature was designed to explore low-cost field monitoring using smartphone-attached NIR cameras or Raspberry Pi-based field devices. It is separate from the core satellite-based workflow.

---

## 10. Loading & Animations

### Description
User experience enhancements during satellite analysis processing.

### Loading Page
- **Trigger**: Shown while backend processes satellite analysis request
- **Animation**: Lottie animation (analyzing.json)
- **Duration**: Typically 5-15 seconds depending on Earth Engine query time
- **Transparency**: No progress indication (backend processing time variable)

### UI Animations
- **Page Transitions**: Smooth slide animations between screens
- **Card Entrance**: Fade and slide up for dashboard cards
- **Button Feedback**: Scale animation on tap
- **Framework**: `flutter_animate` package

---

## Feature Matrix

| Feature | Status | Platform | Dependencies |
|---------|--------|----------|--------------|
| Email/Password Auth | ✅ Production | Android | Supabase |
| Point Field Selection | ✅ Production | Android | Google Maps |
| Boundary Drawing | ✅ Production | Android | Google Maps |
| Satbara Lookup | ✅ Production | Android | OpenStreetMap |
| NDVI Analysis | ✅ Production | Android | FastAPI + Earth Engine |
| NDMI Analysis | ✅ Production | Android | FastAPI + Earth Engine |
| Health Dashboard | ✅ Production | Android | - |
| Profile Management | ✅ Production | Android | Supabase |
| Multi-Language | 🟡 Framework Ready | Android | - |
| NIR Upload | 🟡 Experimental | Android | FastAPI |
| Historical Trends | ❌ Planned | - | - |
| Government Dashboard | ❌ Planned | Web | - |
| Offline Mode | ❌ Planned | Android | - |

**Legend**:
- ✅ Production: Fully implemented and tested
- 🟡 Partial: Framework exists but not fully activated/tested
- ❌ Planned: Designed but not yet implemented

---

## Planned Features (Roadmap)

### Short Term (Next 3-6 months)
1. **Historical Trend Analysis**: Track NDVI/NDMI changes over time
2. **Field Management**: Save and manage multiple registered fields
3. **Downloadable Reports**: PDF generation of field health reports
4. **Push Notifications**: Alerts for significant health changes

### Medium Term (6-12 months)
1. **Government Web Dashboard**: Dedicated portal for agricultural officers
2. **Bulk Field Processing**: Analyze multiple fields simultaneously
3. **Advanced Insights**: Crop-specific thresholds and recommendations
4. **Weather Integration**: Rainfall and temperature data overlay

### Long Term (12+ months)
1. **Yield Prediction**: Machine learning models for harvest estimation
2. **Insurance Integration**: Direct API connections with insurance providers
3. **Marketplace Linkage**: Connect farmers with buyers based on crop health
4. **Community Features**: Farmer groups, knowledge sharing

---

## Technical Debt & Known Issues

### Current Limitations

1. **Error Handling**: Limited user-facing error messages for satellite analysis failures
2. **Offline Support**: No offline capability for field viewing or cached results
3. **Loading Time**: No progress indication during backend processing
4. **Field Storage**: Selected fields not saved; must re-select on each session
5. **Image Caching**: Satellite images not cached, require network on each view

### Planned Improvements

1. Better error messages explaining cloud cover issues
2. Offline field database with SQLite
3. Retry mechanism for failed analysis requests
4. Field history and comparison features
5. Performance optimization for large boundary polygons

---

This documentation represents the current state of TerraScope features as implemented in version 1.0.0. Features and specifications may change as the platform evolves.
