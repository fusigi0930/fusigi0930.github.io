# Get property in Application (from android studio)
android SDK dose not have methods to get/set property, we can use the following code to get property

```java
static public String getprop(String key, String defValue) {
    String value = defValue;
    try {
        Class<?> c = Class.forName("android.os.SystemProperties");
        Method get = c.getMethod("get", String.class, String.class);
        value = (String) get.invoke(c, key, "null");
    }
    catch (Exception e) {
        Log.e(TAG, "getprop exception: " + e.toString());
    }
    return value;
}
```
