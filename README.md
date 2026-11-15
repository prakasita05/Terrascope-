# TerraScope Android

**Connecting Farmers and Authorities Through Satellite Intelligence**

TerraScope is an agricultural monitoring platform that leverages satellite imagery and geospatial analysis to support farmers and government authorities in crop monitoring, field assessment, and agricultural service delivery.

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![Platform](https://img.shields.io/badge/platform-Android-green)
![Flutter](https://img.shields.io/badge/Flutter-3.0+-02569B?logo=flutter)
![License](https://img.shields.io/badge/license-Not%20Yet%20Specified-lightgrey)

---

## 📋 Overview

TerraScope provides farmers with tools to register and monitor their agricultural fields using satellite-based vegetation and moisture indices. The platform bridges the gap between farmers and government authorities by enabling remote field assessment, reducing the need for physical surveys while improving accuracy and transparency in agricultural services.

### Problem Statement

Traditional agricultural monitoring and government service delivery face several challenges:

- **Manual field surveys** are time-consuming, expensive, and limited in scale
- **Subsidy and insurance applications** require extensive documentation and field verification
- **Crop damage assessment** often relies on delayed physical inspections
- **Lack of real-time monitoring** makes it difficult to detect issues early
- **Information asymmetry** between farmers and authorities leads to delays and disputes

### The TerraScope Solution

TerraScope addresses these challenges by:

1. **Satellite-Based Monitoring**: Leveraging Sentinel-2 satellite imagery and Google Earth Engine for real-time crop health assessment
2. **Geo-Tagged Field Registration**: Enabling farmers to precisely register their fields using GPS coordinates or traditional survey numbers
3. **Remote Assessment**: Allowing authorities to review field conditions without physical visits
4. **Data-Driven Insights**: Providing vegetation indices (NDVI) and moisture indices (NDMI) for objective crop health evaluation
5. **Streamlined Services**: Facilitating faster processing of subsidies, insurance, and damage assessment claims

---

## ✨ Key Features

### 🌾 Field Registration & Selection

TerraScope offers **three flexible methods** for field registration:

1. **Point Selection**
   - Single tap on map to mark field location
   - Suitable for small plots or quick registration
   - GPS-based location capture

2. **Boundary Drawing**
   - Draw precise field boundaries on the map
   - Calculate accurate field area
   - Polygon-based field representation

3. **Satbara-Based Lookup**
   - Enter village name and survey number
   - Automatic field location using land records
   - Integration with traditional land documentation

### 🛰️ Satellite-Based Crop Monitoring

- **NDVI (Normalized Difference Vegetation Index)**
  - Measures crop health and vegetation density
  - Range: -1 to +1 (higher values indicate healthier crops)
  - Generated from Sentinel-2 satellite imagery

- **NDMI (Normalized Difference Moisture Index)**
  - Assesses soil and plant moisture content
  - Helps identify water stress in crops
  - Critical for irrigation planning

### 📊 Visual Field Reports

- **Health Score Dashboard**: Color-coded health metrics for crop and soil
- **Satellite Imagery**: Visual representation of field conditions using NDVI overlays
- **Trend Analysis**: Historical data comparison (planned feature)
- **Actionable Insights**: Recommendations based on vegetation indices

### 🗺️ Interactive Mapping

- **Google Maps Integration**: Familiar interface for field navigation
- **Real-Time Location**: GPS-based current position tracking
- **OpenStreetMap Geocoding**: Village and location search
- **Multi-Layer Display**: Satellite imagery overlays on standard maps

### 👤 User Management

- **Secure Authentication**: Supabase-based login and registration
- **Profile Management**: Store farmer details, district, and contact information
- **Multi-Language Support**: Interface localization for regional languages
- **Session Management**: Secure token-based authentication

---

## 🏛️ Farmer Dashboard

The farmer-focused interface provides:

### Registration & Profile
- Create and manage farmer profiles
- Store contact information and district details
- Link multiple fields to a single farmer account

### Field Management
- Register new fields using any of the three methods
- View all registered fields on a map
- Access field history and previous analyses

### Satellite Analysis
- Request satellite analysis for registered fields
- Select crop type for context-specific insights
- View NDVI and NDMI health scores
- Access visual satellite imagery with vegetation overlays

### Agricultural Services
- Access information about subsidies and schemes
- Submit applications for crop insurance
- Request damage assessment for registered fields

---

## 🏢 Government Dashboard

**Current Status**: The Android application is primarily farmer-focused. The government dashboard functionality is planned or may exist as a separate web-based platform.

### Planned Government Features

When implemented, the government dashboard will enable:

- **Centralized Application Review**: View and process farmer applications
- **Remote Field Verification**: Assess fields using satellite data without physical visits
- **Damage Assessment Support**: Compare pre-event and post-event satellite imagery
- **Subsidy Eligibility Review**: Verify field details and crop information
- **Data Analytics**: District-wide crop health statistics and trends
- **Field Survey Planning**: Identify fields requiring physical inspection

---

## 🛠️ Technology Stack

### Mobile Application
- **Framework**: Flutter 3.0+
- **Language**: Dart
- **Platform**: Android (iOS support possible)
- **State Management**: Provider
- **UI Components**: Material Design with custom theming

### Backend & APIs
- **API Framework**: FastAPI (Python)
- **Hosting**: Render (terrascope-frontend.onrender.com)
- **Authentication**: Supabase Auth
- **Database**: Supabase PostgreSQL

### Geospatial & Mapping
- **Mapping**: Google Maps Flutter Plugin
- **Location Services**: Geolocator
- **Geocoding**: OpenStreetMap Nominatim API
- **Satellite Analysis**: Google Earth Engine

### Satellite Data
- **Imagery Source**: Sentinel-2 (ESA)
- **Processing Platform**: Google Earth Engine
- **Indices Calculated**: NDVI, NDMI
- **Temporal Resolution**: Revisit time based on satellite availability

### Supporting Libraries
- **HTTP Client**: http package
- **Localization**: flutter_localizations, intl
- **Animations**: flutter_animate, Lottie
- **Typography**: Google Fonts
- **Storage**: Shared Preferences

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    TerraScope Android App                   │
│                      (Flutter + Dart)                       │
└─────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
         ┌──────────▼────────┐  ┌──────▼──────────┐
         │   Supabase Auth   │  │  FastAPI Backend│
         │   & Database      │  │  (Python)       │
         └───────────────────┘  └─────────────────┘
                                          │
                                ┌─────────┴──────────┐
                                │                    │
                    ┌───────────▼──────────┐  ┌──────▼─────────┐
                    │ Google Earth Engine  │  │  Google Maps   │
                    │  (Satellite Analysis)│  │  (Geocoding)   │
                    └──────────────────────┘  └────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │  Sentinel-2 Satellite │
                    │  (ESA - Free & Open)  │
                    └───────────────────────┘
```

### Data Flow

1. **User Authentication**: Farmer logs in via Supabase authentication
2. **Field Selection**: User selects field using one of three methods (point/boundary/satbara)
3. **Coordinates Sent**: GPS coordinates sent to FastAPI backend
4. **Google Earth Engine Processing**:
   - Backend queries recent Sentinel-2 imagery for the location
   - Calculates NDVI and NDMI for the field
   - Generates color-coded visualization overlay
5. **Results Returned**: Backend sends health scores and satellite image URL to app
6. **Display Dashboard**: App shows visual report with scores and recommendations

---

## 📡 How Satellite Monitoring Works

### 1. Satellite Data Collection
- **Sentinel-2** satellites (operated by ESA) pass over each location every 5-10 days
- Captures multispectral imagery including visible and near-infrared bands
- Free and publicly available through Google Earth Engine

### 2. Vegetation Index Calculation

**NDVI Formula:**
```
NDVI = (NIR - Red) / (NIR + Red)
```
- **NIR (Near-Infrared)**: Band 8 from Sentinel-2
- **Red**: Band 4 from Sentinel-2
- Healthy vegetation reflects more NIR and absorbs more red light

**NDMI Formula:**
```
NDMI = (NIR - SWIR) / (NIR + SWIR)
```
- **SWIR (Short-Wave Infrared)**: Band 11 from Sentinel-2
- Sensitive to moisture content in vegetation and soil

### 3. Health Score Generation

TerraScope converts raw index values into intuitive 0-100 health scores:

```
Crop Health Score = ((NDVI - 0.15) / 0.70) × 100
Soil Moisture Score = ((NDMI - 0.03) / 0.62) × 100
```

- **0-30**: Poor condition (red indicator)
- **31-60**: Moderate condition (yellow indicator)
- **61-100**: Good condition (green indicator)

### 4. Visual Representation
- Color-coded field overlays on satellite imagery
- Interactive maps showing field boundaries
- Comparative analysis over time (planned)

---

## 📸 Screenshots

> **Note**: Screenshots will be added in future updates. The following features are available for capture:
> - Field selection interface (map with three modes)
> - Satellite analysis dashboard with NDVI/NDMI scores
> - Profile and authentication screens
> - Interactive map with field overlays

---

## 🚀 Future Scope

### Planned Features

1. **Historical Trend Analysis**
   - Track NDVI/NDMI changes over time
   - Seasonal comparison and anomaly detection
   - Predictive analytics for yield estimation

2. **Government Web Dashboard**
   - Dedicated interface for agricultural officers
   - Bulk application processing
   - District-level analytics and reporting

3. **Automated Notifications**
   - Alert farmers about significant health score changes
   - Remind users to monitor fields periodically
   - Weather integration for context-aware alerts

4. **Crop-Specific Recommendations**
   - Tailored advice based on crop type and growth stage
   - Fertilizer and irrigation recommendations
   - Pest and disease risk assessment

5. **Offline Mode**
   - Cache field data for offline access
   - Queue analysis requests when connectivity is restored
   - Downloadable field reports

6. **Multi-Field Comparison**
   - Side-by-side analysis of multiple fields
   - Benchmarking against regional averages
   - Best practice identification

7. **Enhanced Geocoding**
   - Integration with official land records APIs
   - Automated boundary detection from cadastral maps
   - Survey number validation

8. **Insurance Integration**
   - Direct API integration with insurance providers
   - Automated claim submission with satellite evidence
   - Damage quantification using satellite data

---

## ⚠️ Current Limitations

### Technical Limitations

1. **Satellite Availability**
   - Dependent on Sentinel-2 revisit schedule (5-10 days)
   - Cloud cover can affect image quality
   - Analysis may be delayed during extended cloud coverage

2. **Accuracy Constraints**
   - NDVI/NDMI are indicators, not definitive diagnoses
   - Small field sizes may be affected by boundary pixels
   - Requires human interpretation for final decisions

3. **Network Dependency**
   - Requires internet connection for satellite analysis
   - Large satellite images may consume significant data
   - API response time depends on backend processing

4. **Debug Build Status**
   - Current APK uses debug signing
   - Release version with production signing pending
   - Performance optimizations ongoing

### Functional Limitations

1. **Single-Dashboard Implementation**
   - Currently farmer-focused only
   - Government dashboard not yet implemented in mobile app
   - Web-based government portal may exist separately

2. **Manual Field Registration**
   - No automated cadastral map integration yet
   - Satbara lookup requires accurate spelling
   - Boundary drawing can be imprecise for large fields

3. **Limited Historical Data**
   - No long-term trend analysis yet
   - Single-point-in-time assessment only
   - Comparison features planned for future releases

4. **Regional Coverage**
   - Currently configured for Maharashtra, India
   - Satbara integration specific to Indian land records
   - International deployment requires localization

---

## 📥 Download & Installation

### Download the APK

> **Status**: No production-ready APK is currently available for distribution.
> 
> The application is configured with debug signing only. A signed release build suitable for public distribution will be published here once ready.

**When available, download the latest version from:**

[📱 Download TerraScope Android APK](https://github.com/Metzo64/Terrascope-Android-/releases/latest)

### Installation Instructions

1. Download the APK file from the link above
2. Enable "Install from Unknown Sources" in Android Settings
3. Open the downloaded APK file
4. Follow the installation prompts
5. Grant necessary permissions (Location, Storage)

### System Requirements

- **Android Version**: 5.0 (Lollipop) or higher
- **Storage**: Minimum 50 MB free space
- **RAM**: 2 GB or more recommended
- **Internet**: Required for satellite analysis and map loading
- **GPS**: Required for location-based field selection

---

## 🔐 Privacy & Security

### Data Handling

- **User Authentication**: Handled securely via Supabase with industry-standard encryption
- **Location Data**: GPS coordinates sent only to backend for analysis, not stored permanently
- **Personal Information**: Profile data stored securely in Supabase PostgreSQL
- **API Keys**: Supabase anon key is rate-limited and restricted by Row Level Security (RLS)

### Permissions Required

- **Location Access**: To capture field coordinates and show current position on map
- **Internet**: For API communication and satellite data retrieval
- **Storage**: For caching map tiles and temporary image storage

---

## 🤝 Contributing

This repository contains project documentation and release distribution only. The source code is maintained privately.

If you encounter issues or have suggestions:
1. Open an issue in this repository
2. Provide detailed description with screenshots if applicable
3. Include device model and Android version for bug reports

---

## 📄 License

License information will be added in future updates. Currently, all rights reserved.

---

## 📞 Contact & Acknowledgments

### Project Information

- **Repository**: https://github.com/Metzo64/Terrascope-Android-
- **Issues**: https://github.com/Metzo64/Terrascope-Android-/issues

### Acknowledgments

- **Sentinel-2 Imagery**: Provided by the European Space Agency (ESA) through the Copernicus Programme
- **Google Earth Engine**: Satellite data processing platform
- **OpenStreetMap**: Geocoding and mapping data
- **Supabase**: Backend infrastructure and authentication
- **Flutter Community**: Open-source framework and packages

---

## 📊 Project Status

**Current Version**: 1.0.0 (Development)

- ✅ Field selection and registration
- ✅ Satellite-based NDVI/NDMI analysis
- ✅ Interactive mapping and geocoding
- ✅ User authentication and profiles
- ✅ Multi-language support framework
- ⏳ Government dashboard (planned)
- ⏳ Historical trend analysis (planned)
- ⏳ Production release build (pending)

---

*TerraScope - Empowering agriculture through satellite intelligence*
