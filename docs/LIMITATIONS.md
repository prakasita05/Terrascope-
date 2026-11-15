# Current Limitations and Known Issues

This document provides an honest assessment of TerraScope's current limitations, technical constraints, and known issues. Understanding these limitations is crucial for setting appropriate expectations and planning future improvements.

---

## Satellite Analysis Limitations

### 1. Cloud Cover Dependency

**Issue**: Satellite analysis requires cloud-free imagery

**Impact**:
- During monsoon season (June-September), analysis may be unavailable for extended periods
- Even partial cloud cover can affect accuracy
- Users may experience "No clear images available" errors

**Current Behavior**:
- Backend filters images with >20% cloud cover
- If no suitable images found in last 60 days, analysis fails
- No automatic retry or notification when imagery becomes available

**Mitigation**:
- Users can try again after a few days
- System currently does not predict when next clear image will be available

**Future Enhancement**:
- Implement notification system for when analysis becomes possible
- Increase date range during monsoon (trade-off: less recent data)
- Explore SAR (Synthetic Aperture Radar) data that penetrates clouds

### 2. Temporal Lag

**Issue**: Analysis reflects conditions from the last clear satellite pass, not real-time

**Impact**:
- Results may be 5-10 days old (Sentinel-2 revisit time)
- During cloudy periods, could be weeks old
- Rapid changes (sudden pest outbreak, irrigation) may not be captured

**Current Behavior**:
- No timestamp displayed to user indicating when satellite image was captured
- Users may assume analysis reflects current conditions

**Mitigation**:
- Documentation explains this limitation
- Backend includes image acquisition date in response (not currently displayed in app)

**Future Enhancement**:
- Display image date prominently in dashboard
- Show "freshness" indicator (e.g., "Data from 3 days ago")
- Alert users when data is older than threshold (e.g., >14 days)

### 3. Spatial Resolution Constraints

**Issue**: Sentinel-2 pixels are 10m × 10m (100 m²)

**Impact**:
- Very small fields (<0.25 acres / ~1000 m²) may have accuracy issues
- Field edges include surrounding pixels (mixed pixel problem)
- Narrow strip fields may not be accurately analyzed

**Current Behavior**:
- No minimum field size validation
- System attempts analysis regardless of field size
- Small fields may show anomalous results

**Mitigation**:
- Backend uses mean reducer over 500m buffer (helps average out noise)
- Polygon-based selection allows users to define precise boundaries

**Future Enhancement**:
- Warn users when selected field is very small
- Use Planet/Maxar imagery for small fields (paid service)
- Implement buffer adjustment based on field size

### 4. Index Interpretation

**Issue**: NDVI and NDMI are indicators, not definitive diagnoses

**Impact**:
- Low NDVI could indicate multiple causes: drought, disease, pests, poor soil, or natural crop stage
- System cannot distinguish between causes
- Requires human expertise for final interpretation

**Current Behavior**:
- Simple color-coded scores (red/yellow/green)
- Generic recommendations ("may need attention")
- No disease-specific or pest-specific diagnosis

**Mitigation**:
- Documentation clearly states indices are indicators only
- Recommendations suggest consulting agricultural experts

**Future Enhancement**:
- Integrate weather data for context (drought vs. disease)
- Compare with neighboring fields for anomaly detection
- Crop-specific thresholds and growth stage modeling

---

## Technical & Performance Limitations

### 1. Network Dependency

**Issue**: All features require active internet connection

**Impact**:
- Cannot use app in areas with poor connectivity
- Cannot view previously analyzed fields offline
- Large satellite images consume mobile data

**Current Behavior**:
- No offline mode
- No local caching of field data or results
- Each dashboard view reloads satellite image from network

**Mitigation**:
- Use Wi-Fi for analysis requests when possible
- Satellite images are optimized for web display (compressed)

**Future Enhancement**:
- Implement SQLite database for field history
- Cache satellite images locally for offline viewing
- Download button for PDF reports

### 2. API Response Time

**Issue**: Analysis can take 5-15 seconds, with no progress indication

**Impact**:
- Users uncertain if system is processing or frozen
- No feedback on what stage processing is at
- Cold starts on free hosting tier can cause 30+ second delays

**Current Behavior**:
- Loading animation shown without progress updates
- User cannot cancel or queue requests
- Failed requests only show generic error

**Mitigation**:
- Animation provides visual feedback that app is working
- Most requests complete within 10 seconds

**Future Enhancement**:
- Progress indicator (e.g., "Querying satellites...", "Calculating indices...")
- Timeout handling with user-friendly retry
- Upgrade to paid hosting for faster cold starts

### 3. Release Build Status

**Issue**: Current APK uses debug signing configuration

**Impact**:
- Not suitable for Google Play Store distribution
- Debug signing is not secure for production
- Performance optimizations not enabled
- APK size larger than optimized release build

**Current Behavior**:
- `build.gradle.kts` configured with `signingConfig = signingConfigs.getByName("debug")`
- No release keystore configured
- No ProGuard/R8 obfuscation

**Mitigation**:
- Direct APK distribution (GitHub Releases) still possible
- Functionality not affected, only security and optimization

**Required Actions**:
- Generate release signing key
- Configure release build in Gradle
- Test signed release build
- Update version code/name for production

### 4. Error Handling

**Issue**: Limited user-facing error messages

**Impact**:
- Generic "Analysis failed" messages don't explain root cause
- Users unsure if problem is temporary or permanent
- Difficult to troubleshoot without backend logs

**Current Behavior**:
- HTTP errors show generic message
- No distinction between client errors (bad input) and server errors (Earth Engine failure)
- No suggested actions for users

**Mitigation**:
- Backend returns error codes (e.g., "no_clear_images")
- Some specific errors have custom messages

**Future Enhancement**:
- Comprehensive error dictionary with user-friendly explanations
- Suggested actions for each error type
- Automatic retry for transient failures

---

## Functional Limitations

### 1. Government Dashboard Missing

**Issue**: Mobile app is farmer-focused only; no government officer interface

**Impact**:
- Cannot achieve goal of connecting farmers and authorities
- Government officers cannot review field data
- No application processing workflow
- No bulk field analysis

**Current Status**:
- Government dashboard may exist as separate web platform (not confirmed)
- No mobile app for officers
- No role-based access control in current implementation

**Planned Solution**:
- Dedicated web dashboard for government users
- Application queue and review workflow
- District-level analytics and reporting

### 2. No Field History

**Issue**: App does not save registered fields or analysis history

**Impact**:
- Users must re-select field each time they want analysis
- Cannot track changes over time
- Cannot compare current vs. previous results
- No proof of historical field condition for insurance claims

**Current Behavior**:
- Each analysis is one-time, ephemeral
- No database storage of field registrations
- No user association with fields

**Mitigation**:
- Backend logs analysis requests (not exposed to users)

**Future Enhancement**:
- Field management screen with saved fields
- Historical trend charts (NDVI over time)
- Downloadable field history reports

### 3. Satbara Integration Incomplete

**Issue**: Survey number lookup does not validate against official land records

**Impact**:
- No verification that survey number is valid
- Returns village center, not actual survey boundary
- Cannot automatically retrieve field boundaries from cadastral maps
- Relies on accurate village name spelling

**Current Behavior**:
- Geocodes village using OpenStreetMap
- Survey number stored but not validated
- User manually marks field location on map after village is located

**Mitigation**:
- User can refine location manually after geocoding
- Still faster than searching map without any reference

**Future Enhancement**:
- Integrate with Maharashtra Bhulekh API for official validation
- Automatic boundary retrieval from cadastral data
- Survey number validation with suggestions for typos

### 4. Single-Point Analysis Only

**Issue**: Analysis based on field centroid, not full field coverage

**Impact**:
- Doesn't account for within-field variability
- Large fields may have different conditions in different areas
- Cannot identify specific problem zones within field

**Current Behavior**:
- Even for boundary-drawn polygons, analysis uses centroid coordinates
- Single NDVI/NDMI value represents entire field
- No spatial variability information

**Mitigation**:
- 500m buffer in backend reduces single-pixel noise
- Mean reducer averages over small area

**Future Enhancement**:
- Zone-based analysis for large fields
- Heatmap showing variability within field
- Identification of problem zones for targeted intervention

---

## Data & Privacy Limitations

### 1. No Data Export

**Issue**: Users cannot export their field data or analysis results

**Impact**:
- Cannot share reports with banks, insurance companies
- No offline record for personal archives
- Difficult to demonstrate field history

**Current Behavior**:
- Results viewable only within app
- No PDF generation
- Screenshot is only sharing option

**Future Enhancement**:
- "Download Report" button generating PDF
- Email/WhatsApp sharing options
- CSV export of historical data

### 2. Minimal Profile Data

**Issue**: Profile captures only basic information

**Impact**:
- No farm size, ownership details, crop history
- Cannot tailor recommendations to user's farming system
- Limited utility for government verification

**Current Behavior**:
- Stores name, email, phone, district only
- No verification of farmer identity or land ownership

**Future Enhancement**:
- Extended profile with farm details
- Aadhaar/Ration Card integration for identity verification
- Link to official land records for ownership proof

### 3. Credentials in Code

**Issue**: Supabase anon key is hardcoded in `lib/main.dart`

**Impact**:
- Key visible in decompiled APK
- Security risk if key permissions are misconfigured

**Mitigation**:
- Anon key is designed for client-side use
- Supabase Row Level Security restricts access
- Key is rate-limited by Supabase

**Best Practice**:
- Current approach is standard for Supabase Flutter apps
- RLS policies are critical security layer
- Service role key (with elevated permissions) is NOT in app

**Improvement**:
- Use environment variables in build process
- Implement additional backend validation layer

---

## Regional & Localization Limitations

### 1. Maharashtra-Centric

**Issue**: Features assume Maharashtra state context

**Impact**:
- Satbara terminology specific to Maharashtra
- District list likely Maharashtra-only
- Geocoding assumes "Maharashtra, India"

**Current Behavior**:
- Hardcoded "Maharashtra, India" in Nominatim queries
- May work for other states but not optimized

**Future Enhancement**:
- State selection in profile
- State-specific land record terminology
- Multi-state geocoding

### 2. Limited Language Support

**Issue**: Multi-language framework exists but not fully activated

**Impact**:
- English-only interface currently
- Marathi localization incomplete
- No support for Hindi or other regional languages

**Current Behavior**:
- `flutter_localizations` framework implemented
- ARB files may exist but incomplete
- Language switcher not active in UI

**Future Enhancement**:
- Complete Marathi translation
- Add Hindi, Kannada, Telugu, Gujarati
- Voice input for low-literacy users

---

## Regulatory & Compliance Limitations

### 1. No Privacy Policy

**Issue**: App collects personal data without published privacy policy

**Impact**:
- May not comply with IT Act requirements
- Users unaware of data usage
- Cannot distribute on Google Play without policy

**Required Actions**:
- Draft comprehensive privacy policy
- Host on accessible URL
- Link from app and Play Store listing

### 2. No Terms of Service

**Issue**: No legal terms governing app usage

**Impact**:
- Unclear liability for incorrect analysis
- No disclaimer about satellite limitations
- Potential misuse without recourse

**Required Actions**:
- Draft terms of service
- Include disclaimers about index interpretation
- Specify acceptable use and limitations

### 3. No License Specified

**Issue**: Repository and app have no license information

**Impact**:
- Unclear rights for code/documentation use
- Cannot legally distribute on some platforms

**Future Action**:
- Select appropriate license (proprietary vs. open-source)
- Add LICENSE file to repository

---

## Known Bugs & Issues

### 1. Loading Page Edge Cases

**Symptom**: If backend request fails, loading page may not dismiss

**Impact**: User stuck on loading screen

**Workaround**: Close and reopen app

**Fix Priority**: High (affects user experience)

### 2. Map Marker Persistence

**Symptom**: Previous markers sometimes remain when switching tabs

**Impact**: Visual clutter, confusion about selected field

**Workaround**: Clear markers manually or restart selection

**Fix Priority**: Medium

### 3. No Input Validation

**Symptom**: Can submit analysis with invalid/empty coordinates

**Impact**: Backend error with unhelpful message

**Workaround**: Ensure field is selected before tapping "Analyze"

**Fix Priority**: High (should validate before API call)

### 4. Image Display on Slow Networks

**Symptom**: Dashboard may show without satellite image if network slow

**Impact**: Scores visible but no visual context

**Workaround**: Wait for image to load, or reopen dashboard

**Fix Priority**: Low (rare, cosmetic)

---

## Performance Bottlenecks

### 1. Earth Engine Query Time

**Constraint**: Earth Engine processing is external, uncontrollable

**Typical Time**: 3-10 seconds

**Worst Case**: 30+ seconds during high load

**Cannot Improve**: Dependent on Google infrastructure

**Can Improve**: Better error handling, caching for repeated queries

### 2. Cold Start Latency

**Issue**: Render.com free tier has cold starts (instance sleeps after inactivity)

**Impact**: First request after idle period takes 30-60 seconds

**Mitigation**: Paid tier has always-on instances

**Alternative**: Keep-alive ping service (artificial traffic)

### 3. Large Satellite Image Loading

**Issue**: Earth Engine tile URLs require multiple requests for full image

**Impact**: Image loads progressively, may appear incomplete initially

**Mitigation**: Use lower resolution visualization (trade-off: less detail)

---

## Comparison with Alternatives

### TerraScope vs. Drone Surveys

**TerraScope Advantages**:
- Free (no drone purchase required)
- Scales to many fields easily
- Historical data available from satellites

**TerraScope Disadvantages**:
- Lower resolution (10m vs. cm-level for drones)
- Weather-dependent (cloud cover)
- Less timely (5-10 day revisit vs. on-demand)

### TerraScope vs. Manual Field Inspection

**TerraScope Advantages**:
- Remote assessment (no travel required)
- Objective, quantifiable metrics
- Larger area coverage

**TerraScope Disadvantages**:
- Cannot identify specific diseases/pests
- Misses ground-level issues (soil crusting, lodging)
- Requires interpretation skills

### TerraScope vs. IoT Sensors

**TerraScope Advantages**:
- No hardware installation cost
- Works for any field (no setup needed)
- Spatial coverage (whole field, not point measurements)

**TerraScope Disadvantages**:
- Less precise (10m pixels vs. point measurements)
- No real-time soil moisture (only satellite-inferred)
- No microclimate data (temperature, humidity)

---

## Risk Assessment

### High-Risk Limitations

Issues that could prevent successful deployment:

1. **Cloud cover during monsoon**: Multi-month unusability in key growing season
2. **Release signing missing**: Cannot distribute via official channels
3. **No government dashboard**: Core value proposition incomplete
4. **Error handling gaps**: User frustration from unclear failures

### Medium-Risk Limitations

Issues that degrade experience but don't block usage:

1. **No field history**: Reduced utility, but core analysis works
2. **Network dependency**: Common limitation for mobile apps
3. **Satbara incomplete integration**: Manual workaround available

### Low-Risk Limitations

Issues that are acceptable for MVP:

1. **Single language**: English suffices for initial rollout
2. **No data export**: Screenshots provide workaround
3. **Debug build**: Direct distribution still possible

---

## Roadmap for Addressing Limitations

### Phase 1 (Next 3 months)
- ✅ Configure release signing
- ✅ Implement field history database
- ✅ Improve error messages and handling
- ✅ Add image acquisition date display

### Phase 2 (3-6 months)
- Government web dashboard MVP
- Offline field viewing
- Historical trend analysis
- Push notifications

### Phase 3 (6-12 months)
- Official land records API integration
- Multi-language activation
- Advanced analytics (crop-specific models)
- Insurance API partnerships

---

## Conclusion

TerraScope's limitations fall into three categories:

1. **Inherent to satellite technology**: Cloud dependency, spatial resolution, temporal lag — cannot be eliminated, only mitigated
2. **MVP shortcuts**: Missing features, incomplete integration — will be addressed in planned roadmap
3. **Technical debt**: Debug build, error handling — fixable in short term

Understanding these limitations is crucial for setting appropriate user expectations and prioritizing development efforts. The platform provides genuine value within its constraints, but users must understand what satellite analysis can and cannot tell them about their fields.

Transparency about limitations builds trust and guides users toward correct interpretation of results.
