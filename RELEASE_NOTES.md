# TerraScope Android Release Notes

## Version 1.0.0 - Initial Release (Pending)

**Release Date**: To be announced

**Status**: 🚧 Development Build - Production release pending

---

### Overview

This is the initial MVP (Minimum Viable Product) release of TerraScope Android, providing farmers with satellite-based crop monitoring using Sentinel-2 imagery and Google Earth Engine.

### ✨ Features Included

#### Field Registration & Selection
- ✅ **Point Selection**: Tap-to-select single-point field registration
- ✅ **Boundary Drawing**: Multi-point polygon field boundary creation
- ✅ **Satbara Lookup**: Village name and survey number-based field location
- ✅ **Interactive Google Maps**: Satellite view with GPS location tracking
- ✅ **Crop Type Selection**: Support for Rice, Wheat, Cotton, and General crops

#### Satellite Analysis
- ✅ **NDVI Calculation**: Normalized Difference Vegetation Index for crop health
- ✅ **NDMI Calculation**: Normalized Difference Moisture Index for soil moisture
- ✅ **Health Scoring**: 0-100 scale with color-coded indicators (Red/Yellow/Green)
- ✅ **Satellite Imagery**: Visual NDVI overlay from Sentinel-2
- ✅ **Real-time Processing**: Analysis results typically within 10 seconds

#### Dashboard & Visualization
- ✅ **Overall Health Score**: Combined crop and soil health metric
- ✅ **Detailed Metrics**: Separate crop health and soil moisture scores
- ✅ **Visual Reports**: Satellite imagery with vegetation index overlays
- ✅ **Actionable Insights**: Contextual recommendations based on field condition

#### User Management
- ✅ **Secure Authentication**: Email and password-based login via Supabase
- ✅ **User Profiles**: Store farmer information (name, phone, district)
- ✅ **Session Management**: Persistent login across app restarts

#### User Experience
- ✅ **Intuitive Interface**: Clean, modern Material Design UI
- ✅ **Loading Animations**: Visual feedback during analysis processing
- ✅ **Smooth Transitions**: Animated page navigation
- ✅ **Localization Framework**: Support for multi-language (English default)

---

### 🛠️ Technical Specifications

**Platform**: Android 5.0 (Lollipop, API 21) and above

**Architecture**:
- Frontend: Flutter 3.0+
- Backend: FastAPI (Python)
- Satellite: Google Earth Engine + Sentinel-2
- Database: Supabase (PostgreSQL)
- Maps: Google Maps Flutter Plugin

**Permissions Required**:
- 📍 Location Access (GPS for field selection)
- 🌐 Internet (API communication and map loading)
- 📁 Storage (image caching, future offline support)

**Network Requirements**:
- Internet connection required for all features
- Recommended: Wi-Fi or 4G for faster satellite image loading
- Data usage: ~1-3 MB per field analysis (including imagery)

---

### ⚠️ Known Limitations (Version 1.0.0)

#### Satellite Analysis Constraints
- **Cloud Cover Dependency**: Analysis unavailable during extended cloudy periods (especially monsoon)
- **Temporal Lag**: Results reflect last clear satellite pass (5-10 days old, sometimes more)
- **Small Field Accuracy**: Fields smaller than ~0.25 acres may have reduced accuracy
- **Indicator Nature**: NDVI/NDMI are indicators, not definitive diagnoses—require expert interpretation

#### Functional Limitations
- **No Field History**: Cannot save fields or view previous analysis results
- **No Offline Mode**: All features require active internet connection
- **Government Dashboard Missing**: Mobile app is farmer-focused only (government portal TBD)
- **No Data Export**: Cannot generate PDF reports or export data
- **Single-Point Analysis**: Entire field represented by single value (no variability mapping)

#### Technical Limitations
- **Debug Build Only**: Current APK uses debug signing (not optimized for production)
- **Network-Only**: No local caching of field data or satellite images
- **Generic Errors**: Limited user-friendly error messages for analysis failures
- **API Response Time**: 5-15 seconds processing time with no progress indicator

#### Regional Scope
- **Maharashtra-Centric**: Satbara lookup and geocoding optimized for Maharashtra
- **English Only**: Multi-language framework exists but not activated
- **Indian Context**: Designed for Indian agricultural practices and land records

---

### 📋 System Requirements

**Minimum Requirements**:
- Android 5.0 (API 21) or higher
- 2 GB RAM
- 50 MB free storage
- GPS capability
- Active internet connection

**Recommended**:
- Android 8.0 or higher
- 4 GB RAM
- 100 MB free storage (for future updates)
- 4G/Wi-Fi connectivity

---

### 🐛 Known Issues

1. **Loading Screen Edge Case**: If backend request fails, loading page may not auto-dismiss (workaround: restart app)
2. **Map Marker Persistence**: Previous markers occasionally remain when switching selection tabs
3. **No Input Validation**: Can submit analysis without valid field selection (shows generic error)
4. **Cold Start Delays**: First request after server idle may take 30-60 seconds (backend hosting limitation)
5. **Image Display**: On slow networks, dashboard may load without satellite image initially

**None of these issues are critical** and most have simple workarounds. Fixes are planned for version 1.1.0.

---

### 🔐 Security & Privacy

- **Authentication**: Secure password hashing via Supabase (bcrypt)
- **Data Transmission**: All API calls over HTTPS/TLS
- **Location Privacy**: GPS coordinates sent only for analysis, not tracked or stored long-term
- **Profile Data**: Protected by Row Level Security (RLS) in Supabase database
- **No Third-Party Tracking**: No analytics or ad frameworks included

**Note**: No formal Privacy Policy published yet—required before Google Play Store distribution.

---

### 📦 Installation

**Current Status**: No production APK available for public download yet.

**Reason**: Application requires release signing configuration and testing before public distribution.

**When Available**:
1. Download APK from [GitHub Releases](https://github.com/Metzo64/Terrascope-Android-/releases)
2. Enable "Install from Unknown Sources" in Android Settings
3. Open the downloaded APK file
4. Grant requested permissions (Location, Internet)
5. Log in or create account

---

### 🚀 What's Next (Planned for v1.1.0)

#### High Priority
- [ ] Configure production release signing
- [ ] Implement field history database
- [ ] Improve error messages with specific causes and actions
- [ ] Display satellite image acquisition date in dashboard
- [ ] Add input validation before analysis submission

#### Medium Priority
- [ ] Offline field viewing (cached results)
- [ ] PDF report generation
- [ ] Historical trend charts (NDVI over time)
- [ ] Push notifications for analysis completion
- [ ] Multi-language activation (Marathi)

#### Long Term
- [ ] Government web dashboard
- [ ] Official land records API integration
- [ ] Weather data integration
- [ ] Crop-specific recommendations
- [ ] Insurance provider API partnerships

---

### 🙏 Acknowledgments

- **Sentinel-2 Imagery**: European Space Agency (ESA) Copernicus Programme
- **Google Earth Engine**: Satellite data processing platform
- **OpenStreetMap**: Geocoding and village location services
- **Supabase**: Authentication and database infrastructure
- **Flutter Community**: Open-source framework and ecosystem

---

### 📞 Support & Feedback

**Issues**: Report bugs and request features at [GitHub Issues](https://github.com/Metzo64/Terrascope-Android-/issues)

**Questions**: For technical support and general inquiries, open a discussion in the repository

---

### 📊 Version History

| Version | Release Date | Status | Highlights |
|---------|--------------|--------|------------|
| 1.0.0 | TBD | Development | Initial MVP with satellite analysis |

---

### ⚖️ License

License information to be added. Currently all rights reserved.

---

**Important**: This is a development release. The application is functional but requires additional configuration (release signing, privacy policy) before official distribution channels (Google Play Store) can be used. Direct APK distribution via GitHub Releases is the current distribution method.

For the most up-to-date information, refer to the [README](README.md) and [project documentation](docs/).
