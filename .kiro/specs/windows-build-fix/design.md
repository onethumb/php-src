# Design Document

## Overview

This design addresses the Windows build failures in the PHP source code by fixing configuration issues, library detection problems, and platform-specific compilation errors. The solution focuses on two main scenarios: building without `crc-fast` library support and building with `crc-fast` library support enabled.

The current Windows build system uses JavaScript-based configuration scripts (`configure.js`, `confutils.js`) and Windows-specific configuration files (`config.w32`) that differ significantly from the Unix autotools-based system. The primary issues appear to be in the `ext/hash/config.w32` file where library detection and header checking may be failing.

## Architecture

### Build System Components

1. **Windows Configuration Layer**
   - `configure.js` - Main configuration entry point
   - `win32/build/confutils.js` - Core configuration utilities
   - `ext/hash/config.w32` - Hash extension Windows configuration
   - `buildconf.bat` - Build configuration script

2. **Library Detection System**
   - `CHECK_LIB()` function for library detection
   - `CHECK_HEADER_ADD_INCLUDE()` function for header detection
   - Path resolution for Windows library locations
   - Debug/Release library variant handling

3. **CRC-Fast Integration Layer**
   - Conditional compilation based on `HAVE_CRC_FAST` define
   - Source file inclusion (`hash_crc_fast.c`, `hash_crc_common.c`)
   - Header installation (`php_hash_crc_fast.h`, `php_hash_crc_common.h`)

## Components and Interfaces

### 1. Windows Library Detection Enhancement

**Problem**: The current `CHECK_LIB("crc_fast.lib", "hash", PHP_CRC_FAST)` call may be failing due to:
- Incorrect library naming conventions on Windows
- Missing search paths for common Windows library locations
- Inadequate handling of debug vs release library variants

**Solution**: Enhance the library detection logic to:
- Support multiple library naming patterns (`crc_fast.lib`, `libcrc_fast.lib`, `crc-fast.lib`)
- Add Windows-specific search paths (vcpkg, Conan, manual installations)
- Properly handle debug library variants (`crc_fast_d.lib`, `crc_fastd.lib`)

### 2. Header Detection Robustness

**Problem**: `CHECK_HEADER_ADD_INCLUDE("libcrc_fast.h", "CFLAGS_HASH", PHP_CRC_FAST)` may fail due to:
- Non-standard header installation locations
- Missing include path configuration
- Case sensitivity issues on Windows filesystems

**Solution**: Improve header detection by:
- Checking multiple header name variants
- Adding standard Windows include paths
- Implementing case-insensitive header searching

### 3. Fallback Build Configuration

**Problem**: When `crc-fast` is not available, the build should continue gracefully without it.

**Solution**: Implement proper fallback logic that:
- Continues build process when `crc-fast` is unavailable
- Provides clear warning messages about missing optional dependencies
- Ensures all non-crc-fast functionality remains intact

### 4. Build Environment Validation

**Problem**: Windows builds may fail due to missing build tools or incorrect environment setup.

**Solution**: Add validation for:
- Required Visual Studio components
- Windows SDK availability
- Build tool versions compatibility

## Data Models

### Build Configuration State

```javascript
// Windows build configuration state
var PHP_CRC_FAST_CONFIG = {
    enabled: false,
    library_found: false,
    header_found: false,
    library_path: "",
    include_path: "",
    library_variants: ["crc_fast.lib", "libcrc_fast.lib", "crc-fast.lib"],
    header_variants: ["libcrc_fast.h", "crc_fast.h", "crc-fast.h"]
};
```

### Library Search Paths

```javascript
// Enhanced search paths for Windows
var CRC_FAST_SEARCH_PATHS = [
    // User-specified path
    PHP_CRC_FAST,
    // vcpkg integration
    VCPKG_ROOT + "\\installed\\x64-windows\\lib",
    VCPKG_ROOT + "\\installed\\x86-windows\\lib", 
    // Conan integration
    CONAN_USER_HOME + "\\.conan\\data\\crc-fast\\*\\*\\package\\*\\lib",
    // Manual installations
    "C:\\crc-fast\\lib",
    "C:\\Program Files\\crc-fast\\lib",
    "C:\\Program Files (x86)\\crc-fast\\lib",
    // Build directory relative paths
    "..\\crc-fast\\lib",
    "..\\..\\crc-fast\\lib"
];
```

## Error Handling

### 1. Library Detection Failures

- **Graceful Degradation**: When `--with-crc-fast` is specified but library is not found, provide clear error message with installation suggestions
- **Warning Mode**: When `crc-fast` is not explicitly requested, show warning but continue build
- **Diagnostic Information**: Include searched paths and attempted library names in error messages

### 2. Compilation Failures

- **Conditional Compilation**: Ensure all `#ifdef HAVE_CRC_FAST` blocks are properly structured
- **Linker Issues**: Handle missing symbols gracefully with appropriate error messages
- **Header Conflicts**: Resolve potential conflicts between system headers and crc-fast headers

### 3. Runtime Validation

- **Library Loading**: Validate that crc-fast library can be loaded at runtime
- **Function Availability**: Check that required crc-fast functions are available
- **Performance Verification**: Ensure crc-fast acceleration is actually working

## Testing Strategy

### 1. Build Configuration Testing

- **Matrix Testing**: Test all combinations of `--with-crc-fast` and `--without-crc-fast`
- **Environment Variations**: Test with different Visual Studio versions and Windows SDK versions
- **Library Scenarios**: Test with crc-fast installed via different package managers (vcpkg, Conan, manual)

### 2. Functional Testing

- **CRC Algorithm Verification**: Ensure CRC calculations produce correct results with and without crc-fast
- **Performance Testing**: Verify that crc-fast provides expected performance improvements
- **Compatibility Testing**: Test with different crc-fast library versions

### 3. Integration Testing

- **Extension Loading**: Verify hash extension loads correctly in all configurations
- **PHP Info Output**: Ensure `phpinfo()` correctly reports crc-fast status
- **Cross-Platform Consistency**: Verify Windows builds produce same results as Linux/macOS

### 4. Error Scenario Testing

- **Missing Dependencies**: Test behavior when crc-fast library is not installed
- **Partial Installation**: Test with library present but headers missing (or vice versa)
- **Version Mismatches**: Test with incompatible crc-fast library versions

## Implementation Phases

### Phase 1: Core Build System Fixes
- Fix library detection logic in `ext/hash/config.w32`
- Enhance search path resolution
- Improve error messaging

### Phase 2: Robustness Improvements  
- Add support for multiple library naming conventions
- Implement fallback mechanisms
- Add build environment validation

### Phase 3: Testing and Validation
- Implement comprehensive test suite
- Add CI/CD integration for Windows builds
- Performance benchmarking and validation

### Phase 4: Documentation and Maintenance
- Update Windows build documentation
- Create troubleshooting guides
- Establish maintenance procedures