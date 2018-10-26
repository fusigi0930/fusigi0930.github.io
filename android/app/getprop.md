# Get property in Application (from android studio)
android SDK dose not have methods to get/set property, we can use the following code to get property

```java
  static public String getprop(String key, String defaultValue) {
  String value = defaultValue;
  try {
    Class<?> c = Class.forName("android.os.SystemProperties");
    Method get = c.getMethod("get", String.class, String.class );
    value = (String)(   get.invoke(c, key, "unknown" ));
  } catch (Exception e) {
    Log.d(TAG, "get property error, " + e.getMessage());
  }
  Log.d(TAG, "get property, " + key + " = " + value);
  return value;
}
```
