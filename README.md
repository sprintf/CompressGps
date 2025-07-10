# CompressGps

A high-performance GPS telemetry compression library designed for racing applications, featuring adaptive sampling, lap-based chunking, and optimized binary encoding.

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/paulnormington/CompressGps)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Kotlin](https://img.shields.io/badge/kotlin-1.9.10-purple.svg)](https://kotlinlang.org/)
[![Java](https://img.shields.io/badge/java-17+-orange.svg)](https://openjdk.org/)

## Overview

CompressGps is specifically designed for **endurance racing telemetry** where multiple amateur drivers share vehicles and benefit from comparative data analysis. The library achieves **93%+ compression ratios** while maintaining racing-precision accuracy and corruption resilience.

### Key Features

- 🏁 **Racing-Optimized**: Tuned for low-performance cars (200hp, 3000lb, street tires)
- 📊 **Adaptive Sampling**: High-frequency capture in braking/acceleration zones, reduced sampling in straights
- 🛡️ **Corruption Resilient**: Lap-based chunking ensures corrupted data doesn't affect entire sessions
- ⚡ **Real-Time Friendly**: Minimal memory footprint (12MB for 3-hour sessions)
- 🌐 **Upload Optimized**: Small file sizes perfect for pit stop cellular/WiFi uploads
- 🎯 **Coaching Ready**: Zone classification for racing line analysis and driver coaching

## Performance

| Metric | Result |
|--------|--------|
| **Compression Ratio** | 14-21:1 typical |
| **Space Savings** | 93%+ on real racing data |
| **Memory Usage** | 12MB for 10Hz, 3-hour sessions |
| **Upload Size** | 4.5KB for 30-minute sessions |
| **GPS Precision** | ~3.6 feet (1e-5 degrees) |
| **Heading Precision** | 0.1 degrees |

## Quick Start

### Installation

Add to your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.normtronix:compress-gps:1.0.0")
}
```

### Basic Usage

```kotlin
import com.normtronix.*

// Create compression stream
val stream = GpsCompressedStream()

// Add GPS points
stream.addPoint(GpsData(
    latitude = 40.23456,
    longitude = -74.54321,
    speedMph = 85,
    heading = 45.5f,
    timestamp = System.currentTimeMillis(),
    lapNumber = 3
))

// Compress and upload during pit stops
val compressedData = stream.toBinary()
println("Compressed ${stream.getStats().first} points to ${compressedData.size} bytes")

// Upload via gRPC/HTTP
uploadToCloud(compressedData)

// Clean up memory for next session
stream.clearData()
```

### Advanced: Racing Analytics

```kotlin
// Analyze racing dynamics
val zone = stream.classifyDynamicZone(currentPoint, previousPoint)
when (zone) {
    RacingZone.BRAKING -> enableHighFrequencySampling()
    RacingZone.CORNER -> captureRacingLine()
    RacingZone.ACCELERATION -> analyzeThrottleApplication()
    RacingZone.STRAIGHT -> reduceSamplingForBandwidth()
}

// Get compression metrics
val metrics = stream.getCompressionMetrics()
println("Compression ratio: ${metrics.compressionRatio}:1")
println("Space saved: ${metrics.spaceSavedPercent}%")
```

## Racing Applications

### Endurance Racing Workflow

1. **Real-time Collection**: Minimal memory GPS data gathering during racing
2. **Pit Stop Processing**: Fast compression during natural race breaks  
3. **Cloud Upload**: Small file sizes for quick cellular/WiFi transmission
4. **Driver Coaching**: Immediate analysis for next driver stint

### Multi-Driver Coaching

```kotlin
// Compare different drivers on same track
val driver1Data = loadDriverSession("driver1-session.gpsc")
val driver2Data = loadDriverSession("driver2-session.gpsc")

// Analyze specific corners
val turn5Analysis = analyzeTurnPerformance(driver1Data, turn = 5)
// Output: "Driver 1: carry more speed through turn 5 entry"
```

## Architecture

### Compression Techniques

- **Adaptive Frequency Sampling**: Automatically detects GPS rates (0.8-10Hz) and optimizes sampling
- **Delta Encoding**: Stores differences between points rather than absolute values
- **Variable-Length Encoding**: Efficient binary representation with ZigZag encoding for signed values
- **Lap-Based Chunking**: Independent compression per lap for corruption resilience
- **Distance Filtering**: 6-foot minimum threshold to eliminate GPS noise

### Binary Format (GPSC)

```
Header: Magic(4) + Version(4) + ChunkCount(4)
Chunks: Size(4) + LapData + Deltas[]
Encoding: VarInt + ZigZag for signed, Unsigned VarInt for time
```

### Racing Zone Classification

The library automatically classifies GPS points into racing zones:

- **STRAIGHT**: Low dynamic activity, reduced sampling
- **BRAKING**: High deceleration, maximum sampling frequency  
- **CORNER**: Turning zones, medium-high sampling
- **ACCELERATION**: High acceleration, maximum sampling frequency

## Testing

The library includes comprehensive testing with real racing data:

```bash
# Run full test suite
./gradlew test

# Test compression performance
./gradlew test --tests "*testCompressionPerformance*"

# Analyze racing dynamics
./gradlew test --tests "*testRacingDynamicsAnalysis*"

# Memory usage analysis
./gradlew test --tests "*testMemoryUsageCalculation*"
```

### Real-World Test Results

Based on actual racing telemetry:

- **49,625 GPS points** → **253KB** (93.0% compression)
- **Racing speeds**: 27-90 mph analyzed
- **G-forces measured**: 0.47-0.77g acceleration, 0.46-1.11g braking
- **Lap analysis**: Skips caution laps, focuses on race pace (laps 3-10)

## API Reference

### Core Classes

#### `GpsData`
```kotlin
data class GpsData(
    val latitude: Double,      // GPS latitude in degrees
    val longitude: Double,     // GPS longitude in degrees  
    val speedMph: Int,        // Speed in miles per hour
    val heading: Float,       // Heading in degrees (0-360)
    val timestamp: Long,      // Unix timestamp in milliseconds
    val lapNumber: Int = 999  // Lap number (999 = unknown)
)
```

#### `GpsCompressedStream`
```kotlin
class GpsCompressedStream {
    fun addPoint(point: GpsData)
    fun toBinary(): ByteArray
    fun getStats(): Pair<Int, Int>
    fun getCompressionMetrics(): CompressionMetrics
    fun clearData()
    
    companion object {
        fun fromBinary(data: ByteArray): List<GpsData>
    }
}
```

### Constants for Racing Applications

```kotlin
object GpsConstants {
    const val DISTANCE_THRESHOLD_FEET = 6.0
    const val MAX_BRAKING_DECEL_G = 0.8    // Street tire limit
    const val MAX_ACCELERATION_G = 0.5     // 200hp/3000lb limit
    const val CORNER_HEADING_CHANGE_THRESHOLD = 5.0
}
```

## Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md).

### Development Setup

```bash
git clone https://github.com/paulnormington/CompressGps.git
cd CompressGps
./gradlew build
./gradlew test
```

### Areas for Contribution

- Track-specific optimization profiles
- Additional racing analytics
- gRPC integration examples
- Performance optimizations
- Additional telemetry formats

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Designed for amateur endurance racing community
- Optimized for low-performance racing cars (200hp, 3000lb, street tires)
- Tested with real racing telemetry data from multiple tracks
- Built for driver coaching and performance improvement

## Support

- 📖 [Documentation](docs/)
- 🐛 [Issue Tracker](https://github.com/paulnormington/CompressGps/issues)
- 💬 [Discussions](https://github.com/paulnormington/CompressGps/discussions)
- 📧 Email: [your-email@example.com]

---

**Built for racers, by racers** 🏁