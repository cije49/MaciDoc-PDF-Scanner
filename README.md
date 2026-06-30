# MaciDoc – PDF Scanner (Flutter)

A simple mobile document scanner. Take a photo of a document, save it as a PDF
on the device, and add more pages to it later.

Store title: **MaciDoc – PDF Scanner** · Launcher name: **MaciDoc** ·
Application ID: `com.nexforgelabs.pdfscanner` · Dart package: `macidoc`.
(The on-disk project folder is still named "PDF Scanner" — that's just the
mounted workspace directory and doesn't affect the app; rename it in your file
explorer if you like. The Android code namespace `com.example.pdf_scanner` is an
internal build identifier and is never shown to users.)

## What's included

**Scanning**

- Two capture modes (Settings → Camera):
  - **Auto-detect edges** — native scanner with edge detection, perspective crop,
    and multi-page capture (`cunning_document_scanner`).
  - **Full photo (no crop)** — a plain camera photo, for damaged/folded/irregular
    documents where auto-crop misfires (`image_picker`).
- Optional **B&W scan filter** (grayscale + contrast), chosen in
  **Settings → Scan colour** (defaults to colour).
- **Review & Save screen** after each scan: it opens straight into processing
  (no flash), then lets you **add more pages**, delete pages, and **Save**.
  Backing out asks before discarding so you don't lose scanned pages.

**PDF size**

- **Quality presets** (Settings → PDF quality): High / Balanced (recommended) /
  Small. Page images are downscaled and re-encoded as JPEG to shrink files while
  staying readable.

**Documents**

- Home screen listing saved PDFs (newest first) with size and date
- **Search** documents by name
- **Scan** → name it → capture one or more pages → saved as a PDF
- **Saved to Downloads:** on Android each PDF is also written to
  `Download/MaciDoc/` (via MediaStore) so it shows up in the Files app, not
  just inside the app. Use **Save to Downloads** to re-export any document.
- Open, **share/export**, **rename**, and delete documents

**Edit a document** (tap a document, or its **Edit** menu)

- Thumbnails of every page
- **Reorder** pages (long-press and drag)
- **Clean up (enhance)** a page — grayscale + contrast for a crisp scan look
- **Retake** or **delete** an individual page
- **Add pages** to an existing document

State is managed with plain `setState`.

## Project layout

```
lib/
  main.dart                      App entry point + theme
  models/
    pdf_document.dart            A saved PDF (file, name, date, size)
  services/
    scanner_service.dart         Native scan + B&W enhancement
    pdf_service.dart             Build/append/reorder/delete pages → PDF
    storage_service.dart         Where PDFs live; list/rename/delete
  screens/
    home_screen.dart             Document list, search, scan, share
    document_detail_screen.dart  Page thumbnails: reorder/retake/delete/add
```

### How appending works

The `pdf` package writes PDFs but can't re-open its own output to edit it.
So each PDF keeps a hidden sidecar folder of its source page images
(`.pages/<DocName>/`). Adding a page copies the new photo into that folder and
rebuilds the PDF from all pages. Deleting a document removes both the PDF and
its sidecar folder.

## First-time setup

This repo contains the Dart source and `pubspec.yaml`. You need to generate the
platform folders (android/ios) once with Flutter installed:

```bash
cd "PDF Scanner"

# Generate android/ios/etc WITHOUT overwriting the existing lib/ and pubspec
flutter create --project-name pdf_scanner .

flutter pub get
```

> `flutter create .` will not overwrite files that already exist, so your
> `lib/` code and `pubspec.yaml` are preserved. If it ever reports a conflict
> on `pubspec.yaml` or `main.dart`, keep the versions in this repo.

### Required permissions

**Android**

The native scanner uses Google ML Kit and needs **minSdkVersion 21+**. Set this
in `android/app/build.gradle`:

```gradle
defaultConfig {
    minSdkVersion 21
}
```

Add the camera permission inside `android/app/src/main/AndroidManifest.xml`,
above the `<application>` tag. The two storage lines are for **saving to
Downloads** on older Android (API ≤ 29); on Android 10+ MediaStore needs none:

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="29" />
```

For Android 10 (API 29) Downloads writes, also add this attribute to the
`<application>` tag:

```xml
<application
    android:requestLegacyExternalStorage="true"
    ... >
```

Saving to Downloads uses a small built-in native MethodChannel
(`android/.../MainActivity.kt`) backed by Android's MediaStore — no third-party
plugin and no GSON/ProGuard rules needed.

> `cunning_document_scanner` bundles the ML Kit document-scanner UI, so it needs
> no extra permissions on modern Android.

**iOS** — minimum deployment target **13.0** (set in `ios/Podfile`:
`platform :ios, '13.0'`). Add to `ios/Runner/Info.plist`:

```xml
<key>NSCameraUsageDescription</key>
<string>This app uses the camera to scan documents.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>This app needs photo access to import scanned pages.</string>
```

## Run it on your phone

1. Enable developer mode / USB debugging (Android) or trust your dev cert (iOS).
2. Plug in your phone and confirm it's detected:
   ```bash
   flutter devices
   ```
3. Run:
   ```bash
   flutter run
   ```

## Dependencies

| Package                   | Purpose                                     |
| ------------------------- | ------------------------------------------- |
| cunning_document_scanner  | Native edge detection, crop, multi-capture  |
| image_picker              | Full-photo (no-crop) capture mode           |
| image                     | B&W filter + PDF compression (resize/encode)|
| pdf                       | Generate PDF documents                      |
| path_provider             | Find the app's local storage dir            |
| path                      | Join file paths safely                      |
| intl                      | Format dates in the list                    |
| open_filex                | Open a saved PDF in a viewer                |
| share_plus                | Share / export PDFs to other apps           |

## Ideas for next

- OCR to make PDFs text-searchable
- Cloud backup / sync
- Folders or tags to organize documents
- Password-protect / encrypt a PDF
