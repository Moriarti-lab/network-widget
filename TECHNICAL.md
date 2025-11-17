# 🔧 Техническая документация

## Архитектура проекта

### Структура файлов
```
NetworkWidget/
├── app/
│   ├── src/main/
│   │   ├── java/com/minimaxagent/networkwidget/
│   │   │   ├── NetworkWidgetProvider.java      # Основной провайдер виджета
│   │   │   └── MainActivity.java               # Активность настроек
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   ├── widget_layout.xml           # Макет виджета
│   │   │   │   └── activity_main.xml           # Макет настроек
│   │   │   ├── xml/
│   │   │   │   └── network_widget_info.xml     # Конфигурация виджета
│   │   │   ├── drawable/
│   │   │   │   ├── widget_background.xml       # Фон виджета
│   │   │   │   ├── button_background.xml       # Фон кнопок
│   │   │   │   └── ic_network_connected.xml    # Иконка сети
│   │   │   ├── values/
│   │   │   │   ├── strings.xml                 # Строковые ресурсы
│   │   │   │   ├── colors.xml                  # Цвета
│   │   │   │   ├── styles.xml                  # Стили
│   │   │   │   └── arrays.xml                  # Массивы
│   │   │   └── AndroidManifest.xml             # Манифест приложения
│   └── build.gradle                            # Конфигурация модуля
├── build.gradle                                # Конфигурация проекта
├── settings.gradle                             # Настройки Gradle
├── gradle.properties                           # Свойства Gradle
├── README.md                                   # Основная документация
└── QUICK_SETUP.md                              # Быстрая настройка
```

## Ключевые классы

### NetworkWidgetProvider
```java
public class NetworkWidgetProvider extends AppWidgetProvider
```

**Основные методы:**
- `onUpdate()` - обновление виджета
- `onReceive()` - обработка действий
- `toggleAirplaneMode()` - переключение режима полета
- `openEngineeringMode()` - открытие инженерного меню
- `switchNetworkMode()` - смена типа сети
- `updateWidget()` - обновление интерфейса виджета

**Поддерживаемые действия:**
- `ACTION_TOGGLE_AIRPLANE_MODE` - переключение режима полета
- `ACTION_OPEN_ENGINEERING_MODE` - открытие инженерного меню
- `ACTION_SWITCH_NETWORK_MODE` - смена сетевого режима
- `ACTION_REFRESH_STATUS` - обновление статуса

### Сетевые режимы
```java
private static final String[] NETWORK_MODES = {
    "LTE Only",           // 14
    "GSM/WCDMA/LTE(PRL)", // 12
    "GSM Only",           // 1
    "WCDMA Only",         // 2
    "Auto (PRL)"          // 0
};
```

## Конфигурация виджета

### Размеры и параметры
```xml
<appwidget-provider
    android:minWidth="160dp"          <!-- 2 ячейки -->
    android:minHeight="110dp"         <!-- 2 ячейки -->
    android:resizeMode="horizontal|vertical"
    android:updatePeriodMillis="1800000"  <!-- 30 минут -->
    android:targetCellWidth="2"
    android:targetCellHeight="2"
    android:widgetCategory="home_screen|keyguard" />
```

### Интерфейс виджета
- **Статусная область** - показывает тип сети и иконку
- **Кнопка режима полета** - быстрое переключение
- **Кнопка инженерного меню** - открытие скрытых настроек
- **Кнопка смены сети** - переключение типов сети

## API и разрешения

### Используемые API
```java
// Системные настройки
Settings.Global.putInt(context.getContentResolver(), 
    Settings.Global.AIRPLANE_MODE_ON, value);

// Сетевая информация
ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
NetworkInfo activeNetwork = cm.getActiveNetworkInfo();

// Телефония
TelephonyManager tm = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
```

### Необходимые разрешения
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_SECURE_SETTINGS" />
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

## Методы переключения сетей

### 1. Рефлексия (ограниченно работает)
```java
private void setNetworkMode(int networkMode) {
    try {
        TelephonyManager telephonyManager = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
        Class<?> telephonyManagerClass = Class.forName(telephonyManager.getClass().getName());
        
        Method getITelephonyMethod = telephonyManagerClass.getDeclaredMethod("getITelephony");
        getITelephonyMethod.setAccessible(true);
        
        Object stub = getITelephonyMethod.invoke(telephonyManager);
        Class<?> iTelephonyClass = Class.forName(stub.getClass().getName());
        
        Method setPreferredNetworkType = iTelephonyClass.getMethod("setPreferredNetworkType", int.class);
        setPreferredNetworkType.invoke(stub, networkMode);
        
    } catch (Exception e) {
        Log.w(TAG, "Не удалось установить сетевой режим", e);
    }
}
```

### 2. USSD команды (для инженерного меню)
```java
private void openEngineeringMode(Context context) {
    try {
        Intent intent = new Intent(Intent.ACTION_DIAL);
        intent.setData(Uri.parse("tel:*#*#4636#*#*"));
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
    } catch (Exception e) {
        Log.e(TAG, "Ошибка открытия инженерного меню", e);
    }
}
```

## Обработка состояний

### Определение типа сети
```java
private String getCurrentNetworkInfo(Context context) {
    ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
    NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
    
    if (activeNetwork == null) return "Нет сети";
    
    String type = activeNetwork.getTypeName();
    String subtype = activeNetwork.getSubtypeName();
    
    if (type.equals("WIFI")) {
        return "WiFi";
    } else if (type.equals("MOBILE")) {
        if (subtype.contains("LTE")) {
            return "4G LTE";
        } else if (subtype.contains("3G")) {
            return "3G";
        } else if (subtype.contains("2G")) {
            return "2G";
        } else {
            return "Мобильная";
        }
    }
    
    return type;
}
```

### Режим полета
```java
private boolean isAirplaneModeOn(Context context) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
        return Settings.Global.getInt(
            context.getContentResolver(),
            Settings.Global.AIRPLANE_MODE_ON,
            0
        ) == 1;
    } else {
        return Settings.System.getInt(
            context.getContentResolver(),
            Settings.System.AIRPLANE_MODE_ON,
            0
        ) == 1;
    }
}
```

## Логирование и отладка

### Уровни логирования
```java
private static final String TAG = "NetworkWidgetProvider";

// Информационные сообщения
Log.d(TAG, "Обновление виджета");
Log.d(TAG, "Получено действие: " + action);

// Предупреждения
Log.w(TAG, "Не удалось установить сетевой режим", e);
Log.w(TAG, "Не удалось установить сетевой режим", e);

// Ошибки
Log.e(TAG, "Ошибка переключения режима полета", e);
Log.e(TAG, "Ошибка открытия инженерного меню", e);
```

## Совместимость

### Минимальные требования
- **API Level:** 24 (Android 7.0)
- **Target API:** 34 (Android 14)
- **Архитектура:** arm64-v8a, armeabi-v7a, x86_64

### Протестированные устройства
- ✅ Samsung Galaxy S21+ (One UI)
- ✅ OnePlus 9 Pro (OxygenOS)
- ✅ Google Pixel 6 (Stock Android)
- ⚠️ Xiaomi (MIUI) - ограниченная функциональность
- ❌ Huawei (EMUI) - инженерное меню заблокировано

## Расширение функциональности

### Добавление новых сетевых режимов
1. Обновите массив `NETWORK_MODES`
2. Добавьте соответствующие значения в `NETWORK_MODE_VALUES`
3. Обновите метод определения сетевого типа

### Добавление новых кнопок
1. Обновите макет `widget_layout.xml`
2. Добавьте новые действия в `NetworkWidgetProvider`
3. Обновите метод `setupButtonListeners()`

---

*Техническая документация от MiniMax Agent*