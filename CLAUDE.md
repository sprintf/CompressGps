# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

compress-gps is a sophisticated GPS telemetry compression system designed for endurance racing applications. It features advanced compression algorithms optimized for real-time data collection, lap-based chunking for corruption resilience, and adaptive sampling tuned for racing dynamics.

**Primary Use Case**: Endurance racing telemetry where multiple amateur drivers share cars and can learn from each other's data through coaching analysis.

## Build System

This project uses Gradle with Kotlin DSL for build configuration.

### Common Commands

```bash
# Build the project
./gradlew build

# Run tests (comprehensive test suite)
./gradlew test

# Run specific test categories
./gradlew test --tests "*testCompressionPerformance*"
./gradlew test --tests "*testAdaptiveSampling*"
./gradlew test --tests "*testRacingDynamicsAnalysis*"

# Clean build artifacts
./gradlew clean
```

## Architecture Overview

### Core Data Structures

**GPS Data Model:**
- `GpsData`: Core GPS point (lat, lon, speed, heading, timestamp, lapNumber)
- `GpsDelta`: Compressed delta representation between GPS points
- `LapChunk`: Independent lap compression unit for corruption resilience

**Racing Analytics:**
- `RacingZone` enum: STRAIGHT, BRAKING, CORNER, ACCELERATION
- `FrequencyAnalysis`: Automatic GPS frequency detection
- `CompressionMetrics`: Upload efficiency analysis

### Advanced Features

1. **Adaptive Frequency Sampling**
   - Automatically detects GPS data rates (0.8-10Hz)
   - Dynamic zone classification for racing dynamics
   - High-frequency sampling in braking/acceleration zones
   - Reduced sampling in straights for bandwidth efficiency

2. **Lap-Based Chunked Compression**
   - Independent compression per lap (corruption resilient)
   - Sequential chunk format with size prefixes
   - Corrupted laps don't affect other laps

3. **Racing-Specific Optimizations**
   - Tuned for low-performance cars (200hp, 3000lb, street tires)
   - Acceleration limits: 0.5g max
   - Braking limits: 0.8g max (street tire constraints)
   - Speed range: 30-100 mph racing speeds

4. **Compression Techniques**
   - Variable-length integer encoding (VarInt)
   - ZigZag encoding for signed values
   - Unsigned encoding for time deltas (always positive)
   - Distance threshold filtering (6-foot minimum)
   - Heading quantization to 0.1 degrees

## Performance Characteristics

### Compression Results (Real Data)
- **File 1**: 49,625 points → 253KB (93.0% space savings)
- **File 2**: 17,598 points → 95KB (92.7% space savings)
- **Compression ratio**: 14-21:1 typical

### Memory Usage (10Hz, 3 hours)
- **Raw memory**: 12 MB total (manageable for real-time collection)
- **Per session**: ~4MB typical
- **Compressed upload**: 500KB-4KB typical

### Racing Data Analysis
Real data shows:
- **GPS frequency**: 2.0Hz actual (varies from assumed 10Hz)
- **Speed analysis**: 27-90 mph race pace (laps 3-9)
- **G-forces**: 0.47-0.77g acceleration, 0.46-1.11g braking
- **Zone distribution**: Mixed straight/corner/braking/acceleration

## Pit Stop Workflow

**Designed for pit stop compression & upload:**
1. Real-time collection during racing (minimal memory)
2. Compression during pit stops (fast processing)
3. Cloud upload via gRPC (small file sizes)
4. Memory cleanup for next session
5. Immediate coaching insights available

Typical 30-minute session:
- Collection: 1,050 data points
- Compression: 4.5KB file
- Upload time: <1 second
- Memory freed: 61.5KB

## File Structure

```
src/
├── main/java/org/example/
│   └── GpsData.kt              # Core implementation (~470 lines)
│       ├── GpsConstants        # Racing dynamics constants
│       ├── GpsData            # GPS point data class
│       ├── GpsDelta           # Delta compression
│       ├── RacingZone         # Zone classification
│       ├── LapChunk           # Lap-based compression
│       ├── GpsCompressedStream # Main compression class
│       └── Extension functions # Binary serialization
└── test/java/org/example/
    ├── GpsCompressionTest.kt   # Comprehensive test suite
    └── resources/              # Real GPS data files
        ├── gps-2022-03-12.csv # Racing data (49K points)
        └── gps-2023-05-27.csv # Racing data (17K points)
```

## Binary Format Specification

**File Header:**
- Magic: "GPSC" (4 bytes)
- Version: 1 (4 bytes)
- Chunk count (4 bytes)

**Chunk Format:**
- Chunk size (4 bytes)
- Lap number (4 bytes)
- Start point: lat(8) + lon(8) + speed(4) + heading(4) + timestamp(8)
- Delta count (4 bytes)
- Variable-length deltas: lat, lon, speed, heading, time

**Encoding:**
- Signed deltas: ZigZag + VarInt
- Time deltas: Unsigned VarInt (optimization)
- Coordinates: 1e-5 degree precision (~3.6 feet)
- Heading: 0.1 degree precision

## Test Coverage

**8 comprehensive tests:**
1. `testDistanceThresholds` - Distance filtering validation
2. `testSerializationRoundTrip` - Binary format integrity
3. `testAdaptiveSampling` - Zone classification testing
4. `testRacingDynamicsAnalysis` - Real racing data analysis
5. `testCompressionPerformance` - Compression efficiency
6. `testMemoryUsageCalculation` - Memory footprint analysis
7. `testPitStopWorkflow` - End-to-end pit stop simulation
8. `testUnsignedTimeOptimization` - Time delta optimization

## Development Guidelines

### Racing Context
- Skip first 2 laps (caution laps) in analysis
- Focus on laps 3-10 for race pace analysis
- Multiple drivers per car (amateur endurance racing)
- Coaching applications: "carry more speed through turn 8"

### Code Quality
- DRY principle applied (e.g., readVariableLength reuses readUnsignedVariableLength)
- Comprehensive error handling with chunk-level corruption recovery
- Real-world testing with actual racing telemetry data

### Technical Constraints
- JVM heap: Recommend 2-4GB for large sessions
- Upload bandwidth: Optimized for cellular/WiFi pit connections
- Battery efficiency: Adaptive sampling reduces power consumption
- Memory efficiency: Automatic cleanup after upload

## Future Considerations

- gRPC integration for cloud upload
- Multi-driver comparison analytics
- Real-time coaching feedback systems
- Track-specific optimization profiles