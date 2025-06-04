

# 📚 **ARCHITECTURE GUIDE - ELEVATE PROJECT**

## 🏗️ **1. OVERALL ARCHITECTURE (MVVM + Clean Architecture)**

```
📱 UI Layer (Compose)
    ↕️
🧠 ViewModel Layer  
    ↕️
🗃️ Repository Layer (Interface + Implementation)
    ↕️
🌐 Data Source (API Service + Local Storage)
```

---

## 🎯 **2. KENAPA PAKAI PATTERN INI?**

### **✅ Separation of Concerns**
- **UI**: Cuma handle tampilan
- **ViewModel**: Handle business logic & state management
- **Repository**: Handle data (dari API atau database)
- **API Service**: Handle network calls

### **✅ Testable**
- Setiap layer bisa di-test secara terpisah
- Mock repository untuk testing ViewModel
- Mock API service untuk testing Repository

### **✅ Scalable**
- Gampang tambah fitur baru
- Gampang ganti sumber data (API → Database)
- Code reusable

---

# 📋 **3. BREAKDOWN PER COMPONENT**

## 🌐 **API SERVICE**
**File**: `CounselingApiService.kt`, `AuthApiService.kt`

```kotlin
interface CounselingApiService {
  @GET("counseling/counselors")
  suspend fun getCounselors(): ConsultantResponse
  
  @GET("counseling/counselors/{id}")
  suspend fun getCounselorDetail(@Path("id") id: Int): ConsultantDetailResponse
}
```

**Tugasnya**:
- ✅ Define endpoint API yang bisa dipanggil
- ✅ Convert HTTP response jadi Kotlin object
- ✅ Handle HTTP methods (GET, POST, PUT, DELETE)

---

## 📊 **MODEL/DATA CLASS**
**File**: `Consultant.kt`, `User.kt`, `CVReviewResponse.kt`

```kotlin
data class Consultant(
  val id: Int,
  val userId: Int,
  val specialization: String,
  val bio: String,
  val verified: Boolean,
  val users: CounselorUser
)
```

**Tugasnya**:
- ✅ Represent data structure dari API
- ✅ Type-safe data handling
- ✅ Auto-convert JSON ↔ Kotlin object (dengan Gson)

---

## 🗃️ **REPOSITORY (Interface + Implementation)**

### **Interface** (`CounselingRepository.kt`)
```kotlin
interface CounselingRepository {
  suspend fun getCounselors(): Result<ConsultantResponse>
  suspend fun getCounselorDetail(id: Int): Result<ConsultantDetailResponse>
}
```

### **Implementation** (`CounselingRepositoryImpl.kt`)
```kotlin
class CounselingRepositoryImpl(
  private val apiService: CounselingApiService
) : CounselingRepository {
  
  override suspend fun getCounselors(): Result<ConsultantResponse> {
    return try {
      val response = apiService.getCounselors()
      Result.success(response)
    } catch (e: Exception) {
      Result.failure(e)
    }
  }
}
```

**Kenapa pisah Interface & Implementation?**
- ✅ **Dependency Inversion**: ViewModel cuma tahu interface, ga tahu implementasinya
- ✅ **Easy Testing**: Bisa mock interface untuk unit test
- ✅ **Flexibility**: Bisa ganti implementasi (API → Database) tanpa ubah ViewModel

---

## 🧠 **VIEWMODEL**
**File**: `CounselingViewModel.kt`

```kotlin
@HiltViewModel
class CounselingViewModel @Inject constructor(
  private val repository: CounselingRepository
) : ViewModel() {
  
  private val _uiState = MutableStateFlow(CounselingUiState())
  val uiState: StateFlow<CounselingUiState> = _uiState.asStateFlow()
  
  fun loadCounselors() {
    viewModelScope.launch {
      _uiState.value = _uiState.value.copy(isLoading = true)
      
      repository.getCounselors().fold(
        onSuccess = { response ->
          _uiState.value = _uiState.value.copy(
            consultants = response.data,
            isLoading = false
          )
        },
        onFailure = { error ->
          _uiState.value = _uiState.value.copy(
            error = error.message,
            isLoading = false
          )
        }
      )
    }
  }
}
```

**Tugasnya**:
- ✅ Handle business logic
- ✅ Manage UI state (loading, error, success)
- ✅ Call repository untuk ambil data
- ✅ Survive configuration changes (rotate screen)

---

## 🎨 **UI STATE**
**File**: `CounselingUiState.kt`

```kotlin
data class CounselingUiState(
  val consultants: List<Consultant> = emptyList(),
  val isLoading: Boolean = false,
  val error: String? = null,
  val selectedCategory: CounselingCategory? = null
)
```

**Tugasnya**:
- ✅ Represent semua kemungkinan state di UI
- ✅ Single source of truth untuk UI
- ✅ Immutable (ga bisa diubah langsung)

---

## 📱 **UI/SCREEN (Compose)**
**File**: `CounselingScreen.kt`, `CounselingDetailScreen.kt`

```kotlin
@Composable
fun CounselingScreen(
  navController: NavController,
  viewModel: CounselingViewModel
) {
  val uiState by viewModel.uiState.collectAsState()
  
  when {
    uiState.isLoading -> CircularProgressIndicator()
    uiState.error != null -> ErrorMessage(uiState.error)
    else -> CounselorList(uiState.consultants)
  }
}
```

**Tugasnya**:
- ✅ Render UI berdasarkan state
- ✅ Handle user interaction
- ✅ Navigate antar screen

---

## 💉 **DEPENDENCY INJECTION (Hilt)**

### **NetworkModule.kt**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
  
  @Provides
  @Singleton
  fun provideCounselingApiService(retrofit: Retrofit): CounselingApiService =
    retrofit.create(CounselingApiService::class.java)
    
  @Provides
  @Singleton
  fun provideCounselingRepository(apiService: CounselingApiService): CounselingRepository =
    CounselingRepositoryImpl(apiService)
}
```

**Tugasnya**:
- ✅ Create & provide dependencies
- ✅ Singleton pattern (1 instance aja)
- ✅ Auto-inject ke ViewModel

**Kenapa butuh DI?**
- ✅ **No Manual Creation**: Ga perlu `new CounselingRepository()` di setiap tempat
- ✅ **Automatic Injection**: Hilt otomatis inject dependencies
- ✅ **Easy Testing**: Bisa inject mock dependencies untuk testing

---

# 🚀 **4. FLOW PER FITUR**

## 👥 **COUNSELING (Lita)**

### **Current Status**: ✅ **COMPLETE**

### **Flow**:
```
1. User buka CounselingScreen
2. ViewModel.loadCounselors() dipanggil
3. Repository ambil data dari API
4. UI update berdasarkan state (loading → success/error)
5. User pilih counselor → navigate ke detail
6. CounselingDetailScreen load detail counselor
7. User klik "Continue" → navigate ke payment
```

### **Files**:
- ✅ `CounselingApiService.kt` - API endpoints
- ✅ `CounselingRepository.kt` - Data layer
- ✅ `CounselingViewModel.kt` - Business logic
- ✅ `CounselingScreen.kt` - List counselors
- ✅ `CounselingDetailScreen.kt` - Detail + payment
- ✅ `Consultant.kt` - Data model

---

## 📄 **CV REVIEW (Arvan)**

### **Current Status**: ✅ **COMPLETE**

### **Flow**:
```
1. User upload CV di CVReviewScreen
2. ViewModel kirim ke API untuk di-analyze
3. Navigate ke CVReviewResultScreen
4. Tampil analysis result + download option
5. User bisa lihat history di CVReviewListScreen
6. Detail review di CVReviewDetailScreen
```

### **Files**:
- ✅ `CVReviewApiService.kt`
- ✅ `CVReviewRepository.kt`
- ✅ `CVReviewViewModel.kt`
- ✅ `CVReviewScreen.kt`
- ✅ `CVReviewResultScreen.kt`
- ✅ `CVReviewListScreen.kt`
- ✅ `CVReviewDetailScreen.kt`

---

## 👤 **PROFILE (Aldi)**

### **Current Status**: ✅ **PARTIAL**

### **Flow**:
```
1. User buka ProfileScreen
2. Load user data dari local storage/API
3. User bisa edit profile di EditProfileScreen
4. Update data ke API
5. Sync dengan local storage
```

### **Files**:
- ✅ `ProfileRepository.kt`
- ✅ `ProfileViewModel.kt`
- ✅ `ProfileScreen.kt`
- ✅ `EditProfileScreen.kt`
- ⚠️ **Missing**: API integration untuk update profile

---

## 🏠 **HOME SCREEN (Azizah)**

### **Current Status**: ✅ **PARTIAL**

### **Flow**:
```
1. Load user info from storage
2. Tampil course recommendations
3. Show recent activities
4. Navigation ke fitur lain
```

### **Files**:
- ✅ `HomeScreen.kt`
- ✅ `HomeViewModel.kt`
- ⚠️ **Missing**: API untuk recommendations

---

## 📝 **ASSESSMENT (Keisya)**

### **Current Status**: ✅ **BASIC**

### **Flow**:
```
1. User mulai assessment
2. Jawab pertanyaan step by step
3. Submit hasil ke API
4. Tampil results & recommendations
```

### **Files**:
- ✅ `AssessmentScreen.kt`
- ✅ `AssessmentViewModel.kt`
- ⚠️ **Missing**: Complete API integration

---

## 📚 **COURSE (Afnan)**

### **Current Status**: ❌ **NOT STARTED**

### **What's Needed**:
- Course API endpoints
- Course repository & ViewModel
- Course list & detail screens
- Enrollment system

---

## 💼 **JOB & SKILL MATCHING (Rifti)**

### **Current Status**: ❌ **NOT STARTED**

### **What's Needed**:
- Job matching API
- Skill assessment integration
- Job recommendation algorithm
- Job list & detail screens

---

# 🎯 **5. INTEGRATION CHECKLIST**

## ✅ **SUDAH TERINTEGRASI**
- ✅ Counseling (Complete dengan API)
- ✅ CV Review (Complete dengan API)
- ✅ Authentication (Login/Register)

## ⚠️ **PARTIALLY INTEGRATED**
- ⚠️ Profile (UI ready, API integration partial)
- ⚠️ Home Screen (UI ready, recommendations missing)
- ⚠️ Assessment (Basic flow, need complete API)

## ❌ **BELUM TERINTEGRASI**
- ❌ Course (Need API & UI)
- ❌ Job & Skill Matching (Need API & UI)

---

# 📋 **6. NEXT STEPS PER ANGGOTA TIM**

## **Lita (Counseling)**: ✅ DONE
- Bisa help tim lain dengan pattern yang sama

## **Arvan (CV Review)**: ✅ DONE
- Fix pagination conflict issue
- Help tim lain dengan API integration

## **Aldi (Profile)**:
1. Complete API integration untuk update profile
2. Add upload photo functionality
3. Sync profile data dengan fitur lain

## **Azizah (Home Screen)**:
1. Integrate dengan Course API untuk recommendations
2. Add recent activities from user data
3. Connect dengan Assessment results

## **Keisya (Assessment)**:
1. Complete API integration
2. Add result persistence
3. Integrate dengan Profile & Home Screen

## **Afnan (Course)**:
1. Create Course API service & repository
2. Build Course screens (list, detail, enrollment)
3. Integrate dengan Assessment recommendations

## **Rifti (Job & Skill Matching)**:
1. Design job matching API
2. Build job recommendation system
3. Create job list & detail screens
4. Integrate dengan CV Review results

---

# 🔧 **7. TEMPLATE UNTUK FITUR BARU**

Setiap fitur baru ikutin pattern ini:

```
📁 data/api/
  └── FeatureApiService.kt

📁 data/repository/
  ├── FeatureRepository.kt (interface)
  └── FeatureRepositoryImpl.kt

📁 model/
  └── FeatureModels.kt

📁 ui/feature/
  ├── FeatureScreen.kt
  ├── FeatureDetailScreen.kt
  └── FeatureUiState.kt

📁 viewmodel/feature/
  └── FeatureViewModel.kt

📁 di/
  └── Update NetworkModule.kt
```

---

# 🎯 **SUMMARY**

Project kalian udah punya **foundation yang solid** dengan **Clean Architecture + MVVM**. Yang perlu dilakukan:

1. **Follow existing pattern** untuk fitur baru
2. **Complete API integration** untuk fitur yang belum
3. **Test integration** antar fitur
4. **Consistent error handling** di semua fitur

**Counseling & CV Review** bisa jadi **reference** buat anggota tim lain karena udah complete! 🚀

Ada pertanyaan spesifik tentang bagian mana yang belum clear? 😊

```plaintext
📱 UI Layer (Compose)
    ↕️
🧠 ViewModel Layer  
    ↕️
🗃️ Repository Layer (Interface + Implementation)
    ↕️
🌐 Data Source (API Service + Local Storage)
```

```kotlin
interface CounselingApiService {
  @GET("counseling/counselors")
  suspend fun getCounselors(): ConsultantResponse
  
  @GET("counseling/counselors/{id}")
  suspend fun getCounselorDetail(@Path("id") id: Int): ConsultantDetailResponse
}
```

```kotlin
data class Consultant(
  val id: Int,
  val userId: Int,
  val specialization: String,
  val bio: String,
  val verified: Boolean,
  val users: CounselorUser
)
```

```kotlin
interface CounselingRepository {
  suspend fun getCounselors(): Result<ConsultantResponse>
  suspend fun getCounselorDetail(id: Int): Result<ConsultantDetailResponse>
}
```

```kotlin
class CounselingRepositoryImpl(
  private val apiService: CounselingApiService
) : CounselingRepository {
  
  override suspend fun getCounselors(): Result<ConsultantResponse> {
    return try {
      val response = apiService.getCounselors()
      Result.success(response)
    } catch (e: Exception) {
      Result.failure(e)
    }
  }
}
```

```kotlin
@HiltViewModel
class CounselingViewModel @Inject constructor(
  private val repository: CounselingRepository
) : ViewModel() {
  
  private val _uiState = MutableStateFlow(CounselingUiState())
  val uiState: StateFlow<CounselingUiState> = _uiState.asStateFlow()
  
  fun loadCounselors() {
    viewModelScope.launch {
      _uiState.value = _uiState.value.copy(isLoading = true)
      
      repository.getCounselors().fold(
        onSuccess = { response ->
          _uiState.value = _uiState.value.copy(
            consultants = response.data,
            isLoading = false
          )
        },
        onFailure = { error ->
          _uiState.value = _uiState.value.copy(
            error = error.message,
            isLoading = false
          )
        }
      )
    }
  }
}
```

```kotlin
data class CounselingUiState(
  val consultants: List<Consultant> = emptyList(),
  val isLoading: Boolean = false,
  val error: String? = null,
  val selectedCategory: CounselingCategory? = null
)
```

```kotlin
@Composable
fun CounselingScreen(
  navController: NavController,
  viewModel: CounselingViewModel
) {
  val uiState by viewModel.uiState.collectAsState()
  
  when {
    uiState.isLoading -> CircularProgressIndicator()
    uiState.error != null -> ErrorMessage(uiState.error)
    else -> CounselorList(uiState.consultants)
  }
}
```

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
  
  @Provides
  @Singleton
  fun provideCounselingApiService(retrofit: Retrofit): CounselingApiService =
    retrofit.create(CounselingApiService::class.java)
    
  @Provides
  @Singleton
  fun provideCounselingRepository(apiService: CounselingApiService): CounselingRepository =
    CounselingRepositoryImpl(apiService)
}
```

```plaintext
1. User buka CounselingScreen
2. ViewModel.loadCounselors() dipanggil
3. Repository ambil data dari API
4. UI update berdasarkan state (loading → success/error)
5. User pilih counselor → navigate ke detail
6. CounselingDetailScreen load detail counselor
7. User klik "Continue" → navigate ke payment
```

```plaintext
1. User upload CV di CVReviewScreen
2. ViewModel kirim ke API untuk di-analyze
3. Navigate ke CVReviewResultScreen
4. Tampil analysis result + download option
5. User bisa lihat history di CVReviewListScreen
6. Detail review di CVReviewDetailScreen
```

```plaintext
1. User buka ProfileScreen
2. Load user data dari local storage/API
3. User bisa edit profile di EditProfileScreen
4. Update data ke API
5. Sync dengan local storage
```

```plaintext
1. Load user info from storage
2. Tampil course recommendations
3. Show recent activities
4. Navigation ke fitur lain
```

```plaintext
1. User mulai assessment
2. Jawab pertanyaan step by step
3. Submit hasil ke API
4. Tampil results & recommendations
```

```plaintext
📁 data/api/
  └── FeatureApiService.kt

📁 data/repository/
  ├── FeatureRepository.kt (interface)
  └── FeatureRepositoryImpl.kt

📁 model/
  └── FeatureModels.kt

📁 ui/feature/
  ├── FeatureScreen.kt
  ├── FeatureDetailScreen.kt
  └── FeatureUiState.kt

📁 viewmodel/feature/
  └── FeatureViewModel.kt

📁 di/
  └── Update NetworkModule.kt
```

