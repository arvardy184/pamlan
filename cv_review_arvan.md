Tentu, ini teks yang sudah dirapikan dalam format Markdown:

```markdown
Perfect! Aku jelasin flow lengkap CV Review dari A-Z, semua file yang terlibat, dan fungsinya. Bakal panjang tapi comprehensive banget!

---

## **🔄 Flow Keseluruhan CV Review**

### **A. Upload CV Flow**

```
[User] → [CVReviewScreen] → [CVReviewViewModel] → [Repository] → [API + Room]
```

### **B. View History Flow**

```
[User] → [CVReviewListScreen] → [CVReviewListViewModel] → [Repository] → [Room/API]
```

### **C. View Detail Flow**

```
[User] → [CVReviewDetailScreen] → [CVReviewDetailViewModel] → [Repository] → [API/Room]
```

### **D. Download Flow**

```
[User] → [DownloadManager] → [Android DownloadManager] → [File System]
```

---

## **📁 File Structure & Fungsinya**

### **🏗️ 1. Data Layer (Backend/Storage)**

| File                          | Fungsi                                  | Flow                                                         |
| :---------------------------- | :-------------------------------------- | :----------------------------------------------------------- |
| **CVReviewApiService.kt** | Interface Retrofit untuk API calls      | Define endpoint upload, get list, get detail, update, delete |
| **CVReviewEntity.kt** | Room entity (table structure)           | Struktur data di database lokal                              |
| **CVReviewDao.kt** | Room DAO (database operations)          | Insert, query, update, delete di Room                        |
| **AppDatabase.kt** | Main Room database class                | Setup database & provide DAO                                 |
| **CVReviewRepository.kt** | Interface repository pattern            | Contract untuk data operations                               |
| **CVReviewRepositoryImpl.kt** | Implementasi repository                 | Handle API calls + Room sync                                 |

### **🔄 2. Data Mapping Layer**

| File                    | Fungsi                       | Flow                                      |
| :---------------------- | :--------------------------- | :---------------------------------------- |
| **CVReviewMapper.kt** | Convert data antar layer     | API Response ↔ Room Entity ↔ UI Model     |
| **CVReviewResponse.kt** | Data model untuk API response | Struktur response dari server             |

### **🧠 3. Business Logic Layer (ViewModels)**

| File                         | Fungsi                   | Flow                                        |
| :--------------------------- | :----------------------- | :------------------------------------------ |
| **CVReviewViewModel.kt** | Handle upload CV logic   | Upload file → API → save to Room → update UI |
| **CVReviewListViewModel.kt** | Handle list CV reviews   | Load data (online/offline) → display list   |
| **CVReviewDetailViewModel.kt**| Handle detail view logic | Load detail → display/edit                  |

### **🎨 4. UI Layer (Screens & Components)**

| File                        | Fungsi                           | Flow                                                  |
| :-------------------------- | :------------------------------- | :---------------------------------------------------- |
| **CVReviewScreen.kt** | Upload CV form                   | File picker → career field → upload                   |
| **CVReviewResultScreen.kt** | Show upload result               | Display scores, analysis, suggestions                 |
| **CVReviewListScreen.kt** | List all CV reviews              | History list dengan offline indicator                 |
| **CVReviewDetailScreen.kt** | Detail view + edit               | Full analysis, download, edit career field            |

### **🔧 5. UI Components**

| File                    | Fungsi                     | Flow                       |
| :---------------------- | :------------------------- | :------------------------- |
| **FileUploadButton.kt** | Button untuk pilih file    | File picker trigger        |
| **PrimaryButton.kt** | Reusable button dengan loading | UI component               |
| **FilePicker.kt** | Utility untuk file picker  | Handle file selection      |
| **CvFormatSelector.kt** | Pilih format CV            | UI for format selection    |
| **CvFormatBox.kt** | Display format options     | UI component               |

### **📥 6. Download System**

| File                  | Fungsi                     | Flow                                |
| :-------------------- | :------------------------- | :---------------------------------- |
| **DownloadManager.kt** | Handle file downloads      | Download CV files dari server       |
| **FileDownloader.kt** | Download utility wrapper   | Wrapper untuk Android DownloadManager |

### **⚙️ 7. Utilities & Config**

| File                 | Fungsi                       | Flow                     |
| :------------------- | :--------------------------- | :----------------------- |
| **NetworkUtil.kt** | Check online/offline status  | Detect connectivity      |
| **NetworkModule.kt** | Dependency injection setup   | Provide Retrofit, Room, repositories |

---

## **🎯 Detailed Flow Explanation**

### **1. 📤 Upload CV Flow**

```
CVReviewScreen.kt
├── User pilih file (FilePicker.kt)
├── User pilih career field
├── User klik upload (PrimaryButton.kt)
└── CVReviewViewModel.kt
    ├── Validate input
    ├── Call repository.uploadCV()
    └── CVReviewRepositoryImpl.kt
        ├── Convert file to multipart
        ├── API call (CVReviewApiService.kt)
        ├── Get response (CVReviewResponse.kt)
        ├── Map to entity (CVReviewMapper.kt)
        ├── Save to Room (CVReviewDao.kt)
        └── Return result
            └── CVReviewResultScreen.kt
                ├── Show scores dengan animasi
                ├── Show AI analysis
                ├── Show suggestions
                └── Download button (DownloadManager.kt)
```

### **2. 📋 View History Flow**

```
CVReviewListScreen.kt
├── CVReviewListViewModel.kt
    ├── Check connectivity (NetworkUtil.kt)
    ├── If online:
    │   ├── API call (CVReviewRepositoryImpl.kt)
    │   ├── Sync to Room (CVReviewDao.kt)
    │   └── Display data
    └── If offline:
        ├── Load from Room (CVReviewDao.kt)
        ├── Convert to UI model (CVReviewMapper.kt)
        ├── Show "Offline Mode" indicator
        └── Display cached data
```

### **3. 🔍 View Detail Flow**

```
CVReviewDetailScreen.kt
├── Get reviewId from navigation
├── CVReviewDetailViewModel.kt
    ├── Try API first (if online)
    ├── Fallback to Room (if offline/error)
    └── CVReviewRepositoryImpl.kt
        ├── API call (CVReviewApiService.kt)
        ├── Or Room query (CVReviewDao.kt)
        └── Return detail data
└── Display:
    ├── File info dengan download button
    ├── Detailed scores
    ├── AI analysis
    ├── Suggestions
    └── Edit career field option
```

### **4. 📥 Download Flow**

```
Any Screen dengan download button
├── User klik download
├── DownloadManager.kt
    ├── Get file URL from CV data
    ├── Use Android DownloadManager
    ├── Save to Downloads folder
    ├── Show notification progress
    └── Complete notification
```

---

## **🔄 Data Flow Diagram**

```
[User Input]
    ↓
[UI Components]
    ↓
[ViewModels]
    ↓
[Repository]
    ↓ ↑
[API Service] ↔ [Room Database]
    ↓           ↓
[Server]      [Local Storage]
```

---

## **📊 State Management**

### **Online State:**

-   **Data Source:** API → Room (sync) → UI
-   **Features:** Upload, delete, load more, real-time data

### **Offline State:**

-   **Data Source:** Room → UI
-   **Features:** View history, read-only mode
-   **Limitations:** No upload, no delete, no load more

---

## **🎛️ Key Features**

### **1. Smart Caching**

-   API response auto-saved to Room
-   Offline access ke semua history
-   Fallback mechanism kalau API error

### **2. File Management**

-   File picker dengan validation
-   Multipart upload ke server
-   Download dengan progress indicator

### **3. UI/UX**

-   Loading states di semua screen
-   Offline indicator
-   Error handling dengan user-friendly messages
-   Animated progress untuk scores

### **4. Data Consistency**

-   Hybrid online/offline mode
-   Auto-sync latest data
-   Delete removes from both API & Room

---

## **🔧 Dependencies**

-   **Room:** Local database
-   **Retrofit:** API calls
-   **Hilt:** Dependency injection
-   **Compose:** Modern UI
-   **Coroutines:** Async operations
-   **Flow:** Reactive data streams

---

**Itu complete flow dari CV Review feature!** Semua file udah dimap dengan jelas, dari upload sampai download, online sampai offline. Ada yang mau ditanya lebih detail?
```