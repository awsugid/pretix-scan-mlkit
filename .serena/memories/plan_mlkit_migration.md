# Migration Plan: QR Scanner to MLKit

# Goal Description
Replace the existing slow ZXing-based barcode scanner in the Android app with Google's MLKit Barcode Scanning API. This changes aims to significantly improve scanning speed and accuracy, leveraging hardware acceleration where possible. Additionally, ensure GitHub CI builds and uploads the APK.

## User Review Required
> [!IMPORTANT]
> **Dependency Change**: We will replace `com.google.zxing:core` with `com.google.mlkit:barcode-scanning`.
> **Model Selection**: We are using the **Bundled** version of MLKit. This increases app size by ~2.4MB but ensures the scanner works immediately offline without waiting for Google Play Services to download the model. This is critical for event reliability.

## Proposed Changes

### `libpretixui-android` Structure
The scanning logic is isolated in the `libpretixui-android` library module.

#### [MODIFY] [build.gradle](file:///home/avei/GithubRepo/playground/aws/pretix-scan-mlkit/pretixscan/libpretixui-repo/libpretixui-android/build.gradle)
-   Remove `implementation 'com.google.zxing:core:3.5.3'`
-   Add `implementation 'com.google.mlkit:barcode-scanning:17.3.0'`

#### [MODIFY] [ScannerView.kt](file:///home/avei/GithubRepo/playground/aws/pretix-scan-mlkit/pretixscan/libpretixui-repo/libpretixui-android/src/main/java/eu/pretix/libpretixui/android/scanning/ScannerView.kt)
-   Remove `ZXingBarcodeAnalyzer` inner class and manual YUV processing.
-   Implement a new `ImageAnalysis.Analyzer` using `BarcodeScanning.getClient()`.
-   Convert CameraX `ImageProxy` to MLKit `InputImage`.
-   Pass detected barcodes to the existing `ResultHandler`.

### GitHub CI Integration

#### [MODIFY] [ci.yml](file:///home/avei/GithubRepo/playground/aws/pretix-scan-mlkit/.github/workflows/ci.yml)
-   Add step to build Release APK: `./gradlew assemblePretixRelease`
-   Add step to upload APK artifacts using `actions/upload-artifact@v4`.

## Verification Plan

### Manual Verification
-   **Build**: Ensure the project compiles with the new dependencies.
-   **Runtime**: Warning: I cannot run the app on a real device/emulator here. Verification will rely on:
    -   Successful compilation.
    -   Code review of the CameraX -> MLKit integration (standard pattern).
    -   User to test on device:
        1.  Open the app.
        2.  Go to scan mode.
        3.  Verify camera preview is visible.
        4.  Scan a QR code.
        5.  Verify the result is detected instantly.
### CI Verification
-   Push changes to GitHub.
-   Verify "CI" workflow runs successfully.
-   Verify APK artifact is available for download.
