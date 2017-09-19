# JNI Hook
```c++
#include <include/AndHook.h>
#include <android/log.h>

//-------------------------------------------------------------------------

static intptr_t sys_getString;

//-------------------------------------------------------------------------

static jstring JNICALL my_getString(JNIEnv *env, jclass obj, jobject resolver, jstring name)
{
	const char *cname = env->GetStringUTFChars(name, NULL);
	if (strcmp(cname, "android_id") == 0) {
		return env->NewStringUTF("fakeandroidid");
	} //if

	jstring js = reinterpret_cast<jstring>(env->CallStaticObjectMethod(obj, 
																	   GetMethodID(env, sys_getString, alloca(JNI_METHOD_SIZE)),
																	   resolver, name));
	if (js != NULL) {
		__android_log_print(ANDROID_LOG_INFO, __FUNCTION__, 
							"js = %s", env->GetStringUTFChars(js, NULL));
	} //if
	return js;
}

//-------------------------------------------------------------------------

void hook_test(JNIEnv *env)
{
	sys_getString = JAVAHookFunction(env, env->FindClass("android/provider/Settings$Secure"), "getString", 
									 "(Landroid/content/ContentResolver;Ljava/lang/String;)Ljava/lang/String;",
									 reinterpret_cast<void *>(my_getString));
}
```

# Native Hook
```c++
#include <include/AndHook.h>
#include <android/log.h>
#define MSHook(X) MSHookFunction(reinterpret_cast<void *>(X), reinterpret_cast<void *>(my_##X), reinterpret_cast<void **>(&sys_##X));

//-------------------------------------------------------------------------

static int(*sys_access)(const char *pathname, int mode);
static int my_access(const char *pathname, int mode)
{
	if (strstr(pathname, "/system/bin/su") != NULL ||
		strstr(pathname, "/system/xbin/su") != NULL) {
		return -1;
	} //if

	__android_log_print(ANDROID_LOG_INFO, __FUNCTION__, "access %s, %d", pathname, mode);
	return sys_access(pathname, mode);
}

//-------------------------------------------------------------------------

void hook_test(JNIEnv *env)
{
	MSHook(access);
}
```
