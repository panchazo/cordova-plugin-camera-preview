# Fix Android 15 getParameters() Crash

## Problem
`java.lang.RuntimeException: getParameters failed (empty parameters)` occurring sporadically on Android 15 devices at `CameraActivity.java:654` in `onPreviewFrame` callback.

## Root Cause
Android 15 compatibility issues with deprecated Camera API where `getParameters()` returns empty parameters, causing crashes in camera operations.

## Solution Overview
Implemented comprehensive error handling and lifecycle synchronization to prevent crashes and ensure graceful degradation.

## Changes Made

### CameraActivity.java
- **Added synchronization variables**: `cameraLock` and `cameraReleasing` flag
- **Enhanced lifecycle management**: 
  - `onResume()`: Wrapped camera operations in `synchronized (cameraLock)`
  - `onPause()`: Added proper cleanup with `setPreviewCallback(null)` before release
- **Fixed takeSnapshot()**: Added try-catch with fallbacks for `getParameters()` failure
- **Added isRecording() method**: Prevents snapshots during video recording
- **Protected all getParameters() calls**: Added try-catch in `takePicture()`, `startRecord()`, `stopRecord()`, `setFocusArea()`, `switchCamera()`

### CameraPreview.java
- **Added helper method**: `getCameraParametersSafely()` centralizes error handling
- **Refactored all methods**: 37 methods now use helper method instead of direct `getParameters()` calls
- **Added try-catch blocks**: All camera parameter operations protected with Android 15 specific error messages
- **Fixed inconsistencies**: Eliminated direct `camera.getParameters()` calls in favor of helper method

### Preview.java
- **Protected setCamera()**: Added try-catch for `getParameters()` calls
- **Enhanced switchCamera()**: Added try-catch for parameter access
- **Fixed surfaceChanged()**: Added try-catch for `getParameters()` and `getSupportedPreviewSizes()`
- **Added surfaceDestroyed() cleanup**: Proper callback cleanup before camera release

## Key Features
- **Fallback mechanisms**: Default values when parameters unavailable
- **Lifecycle synchronization**: Prevents race conditions during camera operations
- **Recording state management**: Blocks conflicting operations during video recording
- **Comprehensive error handling**: All `getParameters()` calls protected with specific Android 15 error messages
- **Graceful degradation**: App continues functioning even when camera parameters fail
