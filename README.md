# Hilt-new
https://dagger.dev/hilt/gradle-setup.html

•	add project level:
	classpath("com.google.dagger:hilt-android-gradle-plugin:2.38.1")

•	add to module level:
into plugins area:
    id 'kotlin-android'
    id 'kotlin-kapt'
into dependencies area:
		implementation("com.google.dagger:hilt-android:2.38.1")
		kapt("com.google.dagger:hilt-android-compiler:2.38.1")
•	extend application class with hit annotation:
@HiltAndroidApp
class GameApplication:Application() {
}


•	add to each injected class annotation:
	activities-fragments-views: @AndroidEntryPoint
    viewModels: @HiltViewModel

•	inside injected class add to injected fields @Inject annotaion

•	injectable classes should declare injected constructor
for example:
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }

•	for classes which can not declare injected instructors (like interfaces, libs etc.) should do as so:
interface AnalyticsService {
  fun analyticsMethods()
}

// Constructor-injected, because Hilt needs to know how to
// provide instances of AnalyticsServiceImpl, too.
class AnalyticsServiceImpl @Inject constructor(
  ...
) : AnalyticsService { ... }

@Module
@InstallIn(ActivityComponent::class)
abstract class AnalyticsModule {

  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}

 retrofit example:
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    // Potential dependencies of this type
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}

•	if we want to provide multiple objects with same type, we should use qualifier:

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class OtherInterceptorOkHttpClient

and then:
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

  @AuthInterceptorOkHttpClient
  @Provides
  fun provideAuthInterceptorOkHttpClient(
    authInterceptor: AuthInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(authInterceptor)
               .build()
  }

  @OtherInterceptorOkHttpClient
  @Provides
  fun provideOtherInterceptorOkHttpClient(
    otherInterceptor: OtherInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(otherInterceptor)
               .build()
  }
}


יש להגדיר את ה-scope  לש האובייקט המוזרק כך:
@ActivityScoped

לדוגמה:

@ActivityScoped
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
אובייקט שהוא singleton יהיה עם annotation של singleton

@Module
@InstallIn(SingletonComponent::class)
object AnalyticsModule {

  @Singleton
  @Provides
  fun provideAnalyticsService(): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}

![image](https://user-images.githubusercontent.com/55790024/153771693-52f00ace-2ba3-488d-8645-8b534289993a.png)
