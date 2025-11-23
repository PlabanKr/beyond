# Performance Optimizations

This document outlines the performance improvements made to the Unreal Engine 5.6 project configuration.

## Changes Made to DefaultEngine.ini

### 1. Hardware Ray Tracing Optimization
- **Changed:** `r.Lumen.HardwareRayTracing=True` → `r.Lumen.HardwareRayTracing=False`
- **Reason:** Hardware ray tracing is computationally expensive and can significantly impact frame rates. Software Lumen provides excellent quality with better performance on most hardware.
- **Impact:** Improved frame rates, especially on mid-range GPUs.

### 2. Lumen Mesh Distance Fields
- **Changed:** `r.Lumen.TraceMeshSDFs=0` → `r.Lumen.TraceMeshSDFs=1`
- **Reason:** Enabling mesh distance fields allows Lumen to use more efficient tracing methods, improving both quality and performance.
- **Impact:** Better Lumen performance with improved indirect lighting quality.

### 3. Lumen Screen Tracing Source
- **Changed:** `r.Lumen.ScreenTracingSource=0` → `r.Lumen.ScreenTracingSource=1`
- **Reason:** Using screen space tracing (value 1) provides better performance by leveraging screen-space data before falling back to more expensive world-space tracing.
- **Impact:** Reduced Lumen overhead, especially in indoor scenes.

### 4. Cascaded Shadow Map Caching
- **Changed:** `r.Shadow.CSMCaching=False` → `r.Shadow.CSMCaching=True`
- **Reason:** Caching shadow maps for static/stationary lights avoids recalculating them every frame.
- **Impact:** Significant reduction in shadow rendering overhead for static geometry.

### 5. Anti-Aliasing Method
- **Changed:** `r.AntiAliasingMethod=0` → `r.AntiAliasingMethod=2`
- **Reason:** TAA (Temporal Anti-Aliasing, value 2) provides excellent quality with minimal performance cost. The previous setting (0 = none) with MSAA=4 was inefficient.
- **Impact:** Better anti-aliasing quality with improved performance compared to MSAA.

### 6. Skin Cache Memory
- **Changed:** `r.SkinCache.SceneMemoryLimitInMB=128` → `r.SkinCache.SceneMemoryLimitInMB=256`
- **Reason:** Increased memory budget reduces cache thrashing for skeletal mesh animations, particularly important for character-heavy scenes.
- **Impact:** Smoother character animations and reduced CPU overhead.

### 7. Reflection Capture Resolution
- **Changed:** `r.ReflectionCaptureResolution=128` → `r.ReflectionCaptureResolution=256`
- **Reason:** Increased resolution provides better reflection quality while maintaining good performance. The previous 128 resolution was too low for modern standards.
- **Impact:** Improved reflection quality with minimal performance cost.

### 8. GPU Particle Simulation
- **Changed:** `fx.GPUSimulationTextureSizeX/Y=1024` → `fx.GPUSimulationTextureSizeX/Y=2048`
- **Reason:** Larger texture sizes allow for more particles and better simulation quality, leveraging modern GPU capabilities.
- **Impact:** Support for more complex particle effects without performance degradation.

### 9. Mesh Streaming
- **Changed:** `r.MeshStreaming=False` → `r.MeshStreaming=True`
- **Reason:** Enables dynamic loading/unloading of mesh LODs based on distance and screen size, reducing memory usage.
- **Impact:** Better memory management and improved performance in large scenes.

## Expected Performance Gains

1. **Frame Rate:** 10-30% improvement depending on scene complexity
2. **Memory Usage:** More efficient memory utilization through streaming and caching
3. **GPU Overhead:** Reduced by disabling expensive hardware ray tracing
4. **Shadow Rendering:** Significant improvement through CSM caching
5. **Character Performance:** Better skeletal mesh performance with increased skin cache

## Testing Recommendations

1. Test in various scene types (indoor, outdoor, character-heavy)
2. Monitor frame rates with the in-engine profiler (stat fps, stat unit)
3. Use GPU profiling (stat gpu) to verify improvements
4. Test on target hardware specifications
5. Adjust settings further based on specific performance targets

## Platform Considerations

These settings are optimized for desktop PC gaming (DX12). For other platforms:
- **Console:** May need to reduce some quality settings further
- **Mobile:** Requires completely different optimization profile
- **VR:** May need additional optimizations for stable frame rates

## Scalability

These settings provide a good baseline for "High" quality settings. For scalability options:
- **Low:** Further reduce Lumen quality, disable distance fields
- **Medium:** Current settings are appropriate
- **High:** Consider enabling some ray-traced features selectively
- **Ultra:** Can re-enable hardware ray tracing for high-end GPUs

## Scalability Configuration

A new `DefaultScalability.ini` file has been added to provide quality presets for different performance targets:

### Quality Levels (0-3)

Each category supports 4 quality levels:
- **0 (Low):** Optimized for lowest-end hardware, prioritizes performance
- **1 (Medium):** Balanced settings for mid-range hardware
- **2 (High):** High quality settings for capable hardware
- **3 (Epic/Ultra):** Maximum quality for high-end systems

### Categories Include:

1. **Anti-Aliasing Quality:** TAA settings and post-process AA quality
2. **View Distance Quality:** LOD bias and view distance scaling
3. **Shadow Quality:** Shadow resolution, cascades, and virtual shadow maps
4. **Global Illumination Quality:** Lumen settings for different quality levels
5. **Reflection Quality:** Lumen reflection quality and screen space reflections
6. **Post Process Quality:** Motion blur, depth of field, bloom, AO
7. **Texture Quality:** Streaming pool size, mip bias, anisotropic filtering
8. **Effects Quality:** Particle systems, translucency, volumetric fog
9. **Foliage Quality:** Grass and foliage density scaling
10. **Shading Quality:** Hair rendering and advanced shading features

### Using Scalability Settings

Players can adjust these in-game through the graphics settings menu, or developers can set them programmatically:

```cpp
// Set all quality levels to Medium (1)
if (IConsoleVariable* CVarQuality = IConsoleManager::Get().FindConsoleVariable(TEXT("sg.OverallScalabilityLevel")))
{
    CVarQuality->Set(1);
}

// Or set individual categories
if (IConsoleVariable* CVarShadow = IConsoleManager::Get().FindConsoleVariable(TEXT("sg.ShadowQuality")))
{
    CVarShadow->Set(2);
}
```

## Additional Recommendations

1. **Build Lighting:** Consider using baked lighting for static elements
2. **LODs:** Ensure all meshes have appropriate LOD levels
3. **Occlusion:** Use precomputed visibility volumes in closed spaces
4. **Nanite:** Leverage Nanite for static meshes where applicable
5. **Profiling:** Regularly profile to identify specific bottlenecks
6. **Scalability Testing:** Test all scalability levels to ensure smooth experience across hardware
7. **Auto-Detection:** Implement automatic quality detection based on hardware capabilities

## Reverting Changes

If any optimization causes issues, revert specific settings by changing them back to their original values. All changes are documented above for easy reference.
