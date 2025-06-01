# y-cruncher Installation and Testing Guide

## 📋 Table of Contents
- [Overview](#overview)
- [Download and Installation](#download-and-installation)
- [System Requirements](#system-requirements)
- [Running Tests](#running-tests)
- [Reading Results](#reading-results)
- [Troubleshooting](#troubleshooting)
- [Performance Examples](#performance-examples)

## 🎯 Overview

y-cruncher is a multi-threaded program for computing mathematical constants (Pi, e, Golden Ratio, etc.) to trillions of digits. It's widely used for:
- **System Benchmarking**: CPU and memory performance testing
- **Stability Testing**: Stress testing for overclocked systems
- **Educational Purpose**: Learning about high-precision mathematics

## 📥 Download and Installation

### Step 1: Download

Visit the official y-cruncher website: https://www.numberworld.org/y-cruncher/

**Choose the appropriate version:**
- **Linux (Static)** - Recommended for most users (no dependencies)
- **Linux (Dynamic)** - Better performance but requires libraries

```bash
# Download static version (recommended)
wget https://cdn.numberworld.org/y-cruncher-downloads/y-cruncher%20v0.8.6.9545-static.tar.xz
```

### Step 2: Extract Files

```bash
# Extract the archive
tar -xJf "y-cruncher v0.8.6.9545-static.tar.xz"

# Navigate to directory
cd "y-cruncher v0.8.6.9545-static"

# Make executable (if needed)
chmod +x y-cruncher
```

### Step 3: Verify Installation

```bash
# Run y-cruncher
./y-cruncher

# You should see the main menu
```

## 💻 System Requirements

### Minimum Requirements
- **OS**: 64-bit Linux
- **CPU**: Any x86/x64 processor
- **RAM**: 512 MB (for small computations)
- **Disk**: 100 MB free space

### Recommended for Serious Testing
- **CPU**: Multi-core processor (8+ cores)
- **RAM**: 8+ GB
- **Disk**: 10+ GB free space
- **Architecture**: AVX2 or AVX512 support for best performance

## 🚀 Running Tests

### Test 1: Benchmark Pi (Quick Performance Test)

This is the standard benchmark for comparing system performance.

```bash
./y-cruncher
# Select: 0 (Benchmark Pi)
# Select: 1 (Multi-Threaded all cores)
# Choose size based on your RAM:
#   - 4-8 GB RAM: Option 4 (250M digits)
#   - 8-16 GB RAM: Option 5 (500M digits)
#   - 16+ GB RAM: Option 6 (1B digits)
```

**Example Output:**
```
Total Computation Time: 80.340 seconds (1.339 minutes)
CPU Utilization: 1476.75% + 2.08% kernel overhead
Multi-core Efficiency: 92.30% + 0.13% kernel overhead
```

### Test 2: Custom Pi Computation (Long Duration Test)

For stress testing or specific time duration requirements.

```bash
./y-cruncher
# Select: 4 (Custom Compute a Constant)
# Pi is selected by default
# Select: 3 (Decimal Digits)
# Enter digits based on target duration:
#   - 5 minutes: 2000000000 (2B digits)
#   - 7-8 minutes: 3000000000 (3B digits)
#   - 10+ minutes: 4000000000 (4B digits)
# Select: 0 (Start Computation!)
```

### Test 3: Component Stress Test (System Stability)

Tests individual system components for stability.

```bash
./y-cruncher
# Select: 2 (Component Stress Tester)
# Select: 4 (Time Limit per test)
# Enter: 300 (for 5 minutes per component)
# Select: 0 (Start Stress-Testing!)
```

## 📊 Reading Results

### Key Performance Metrics

#### 1. **CPU Utilization**
```
CPU Utilization: 1534.20% + 0.34% kernel overhead
Multi-core Efficiency: 95.89% + 0.02% kernel overhead
```
- **Target**: >95% efficiency indicates good multi-threading
- **Formula**: CPU Utilization = (Actual cores used) × 100%

#### 2. **Timing Breakdown**
```
Series CommonP2B3: 370.252 seconds (85.0%)
Large Division: 18.094 seconds (4.1%)
Base Converting: 27.893 seconds (6.4%)
```
- **Series Computation**: Usually 80-90% of total time
- **Division/Multiplication**: CPU-intensive operations
- **Base Converting**: Memory bandwidth sensitive

#### 3. **Memory Usage**
```
Working Memory: 13.2 GiB
Total Memory: 13.3 GiB
```
- Monitor that memory usage doesn't exceed 80-90% of available RAM
- If insufficient memory: reduce digit count or use swap mode

#### 4. **Validation**
```
Spot Check: Good through 2,500,000,000
```
- **"Good"**: Computation completed successfully
- **"Failed"**: Indicates hardware instability or errors

### Performance Comparison Example

| System | Cores | RAM | 1B Digits Time | Notes |
|--------|-------|-----|----------------|--------|
| VM SSE3 | 8C | 4GB | ~350 seconds | Basic performance |
| VM SSE3 | 16C | 16GB | ~145 seconds | 2× cores = 2.4× faster |
| Native AVX512 | 16C | 16GB | ~27 seconds | 5× faster than SSE3 |

## 🔧 Troubleshooting

### Common Issues

#### 1. **Out of Memory (OOM Kill)**
```
Out of memory: Killed process 1796 (05-A64 ~ Kasumi)
```
**Solution:**
- Reduce digit count
- Close other applications
- Add swap space
- Use disk mode for large computations

#### 2. **Insufficient Memory Warning**
```
Warning: There is insufficient physical memory.
```
**Solutions:**
```bash
# Check available memory
free -h

# Add swap file (if needed)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

#### 3. **Dynamic Version Library Error**
```
error while loading shared libraries: libatomic.so.1
```
**Solution:**
```bash
# Install missing libraries
sudo apt update
sudo apt install libatomic1 libtbb-dev

# Or use static version instead
cd ~/y-cruncher*static*/
./y-cruncher
```

#### 4. **Permission Warnings**
```
Insufficient permissions to set thread priority.
```
**Note**: This is normal and doesn't affect functionality.

### Memory Requirements Guide

| Digit Count | RAM Needed | Recommended For |
|-------------|------------|-----------------|
| 25M | ~200 MB | Quick test |
| 100M | ~500 MB | Basic benchmark |
| 500M | ~2.2 GB | Standard test |
| 1B | ~4.4 GB | Performance test |
| 2.5B | ~11.5 GB | Stress test |
| 5B+ | 20+ GB | Enterprise/Record |

## 📈 Performance Examples

### Example 1: 8-Core VM Performance
```
Processor: Common KVM processor (8 cores)
Memory: 3.91 GiB
Instruction Set: x64 SSE3

500M digits: 111.928 seconds
CPU Utilization: 766.93% (95.87% efficiency)
```

### Example 2: 16-Core VM Performance
```
Processor: Common KVM processor (16 cores)  
Memory: 15.6 GiB
Instruction Set: x64 SSE3

3B digits: 435.779 seconds (7.26 minutes)
CPU Utilization: 1534.20% (95.89% efficiency)
Memory Usage: 13.2 GiB (84% of available)
```

### Example 3: Native AMD Zen 4
```
Processor: AMD EPYC 9334 (16 cores)
Memory: 15.6 GiB  
Instruction Set: x64 AVX512-GFNI

2.5B digits: 80.340 seconds (1.34 minutes)
CPU Utilization: 1476.75% (92.30% efficiency)
```

## 🎯 Quick Start Commands

### 5-Minute CPU Burn Test
```bash
./y-cruncher
# Press: 4 → 3 → type "2000000000" → 0
```

### Standard Benchmark
```bash
./y-cruncher  
# Press: 0 → 1 → 5 (for 500M digits)
```

### Memory Stress Test
```bash
./y-cruncher
# Press: 4 → 3 → type "1500000000" → 0
```

## 📝 Tips for Best Results

1. **Close other applications** before testing
2. **Monitor temperatures** during long tests
3. **Use UPS** for critical/long computations
4. **Start small** and increase digit count gradually
5. **Save validation files** for verification
6. **Check system stability** with component tests first

## 🔗 Additional Resources

- **Official Website**: https://www.numberworld.org/y-cruncher/
- **Documentation**: https://www.numberworld.org/y-cruncher/guides/
- **Community Forum**: https://mersenneforum.org/forumdisplay.php?f=136
- **GitHub Issues**: https://github.com/Mysticial/y-cruncher/issues

---

**Note**: y-cruncher is developed by Alexander J. Yee and is free for non-commercial use. Please respect the license terms and contribute to the community by sharing your benchmark results!