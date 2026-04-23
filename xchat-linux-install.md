# Установка 0xChat (xchat) на Parrot OS / Linux

## Что это такое
0xChat — Nostr-мессенджер на Flutter. Исходники лежат в `~/xchat-app-main/`.
Flutter поддерживает сборку нативных Linux-приложений через GTK3.

---

## Шаг 1 — Системные зависимости

```bash
sudo apt install -y clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-good gstreamer1.0-plugins-bad libsecret-1-dev
```

- `clang` + `cmake` + `ninja-build` — компиляторы и система сборки C++
- `pkg-config` — помогает находить установленные библиотеки
- `libgtk-3-dev` — GTK3, через него Flutter рисует окна на Linux
- `liblzma-dev` — сжатие, нужно некоторым зависимостям
- `libgstreamer1.0-dev` + плагины — медиабиблиотека для аудио (нужна `audioplayers_linux`)
- `libsecret-1-dev` — безопасное хранение ключей (нужна `flutter_secure_storage_linux`)

---

## Шаг 2 — Установка Flutter SDK

Flutter SDK — это компилятор и инструменты для сборки. Скачиваем с GitHub:

```bash
git clone https://github.com/flutter/flutter.git ~/flutter -b stable
```

Добавляем в PATH (чтобы работал в терминале):

```bash
echo 'export PATH="$PATH:$HOME/flutter/bin"' >> ~/.bashrc
source ~/.bashrc
```

Проверяем:

```bash
flutter --version
```

---

## Шаг 3 — Включить Linux desktop сборку

Flutter по умолчанию не собирает Linux-приложения, нужно включить:

```bash
flutter config --enable-linux-desktop
```

Проверка что Linux-платформа доступна:

```bash
flutter doctor
```

Нужно чтобы было `[✓] Flutter` и `[✓] Linux toolchain`.
Предупреждения про Android/Chrome/Xcode игнорируем — нам нужен только Linux.

---

## Шаг 4 — Скачать исходники xchat

```bash
git clone https://github.com/0xchat-app/0xchat-app-main.git ~/xchat-app-main
cd ~/xchat-app-main
```

**Инициализировать git submodules** (обязательно, иначе сборка упадёт):

```bash
git submodule update --init --recursive
```

Это скачает вложенные репозитории: `0xchat-core`, `nostr-dart`, `nostr-mls-package`, `bitchat-flutter-plugin`.

---

## Шаг 5 — Патч flutter_sound (баг плагина)

Плагин `flutter_sound` называет свой CMake-таргет `taudio_plugin` вместо `flutter_sound_plugin`,
из-за чего сборка падает. Чиним вручную:

```bash
SOUND_DIR=~/.pub-cache/hosted/pub.dev/flutter_sound-9.30.0/linux
```

**5.1 — Исправить CMakeLists.txt:**

```bash
sed -i 's/set(PROJECT_NAME "taudio")/set(PROJECT_NAME "flutter_sound")/' $SOUND_DIR/CMakeLists.txt
sed -i 's/set(PLUGIN_NAME "taudio_plugin")/set(PLUGIN_NAME "flutter_sound_plugin")/' $SOUND_DIR/CMakeLists.txt
sed -i 's/set(taudio_bundled_libraries/set(flutter_sound_bundled_libraries/' $SOUND_DIR/CMakeLists.txt
```

**5.2 — Создать заголовочный файл:**

```bash
mkdir -p $SOUND_DIR/include/flutter_sound
cat > $SOUND_DIR/include/flutter_sound/flutter_sound_plugin.h << 'EOF'
#ifndef FLUTTER_PLUGIN_FLUTTER_SOUND_PLUGIN_H_
#define FLUTTER_PLUGIN_FLUTTER_SOUND_PLUGIN_H_

#include <flutter_linux/flutter_linux.h>

G_BEGIN_DECLS

#ifdef FLUTTER_PLUGIN_IMPL
#define FLUTTER_PLUGIN_EXPORT __attribute__((visibility("default")))
#else
#define FLUTTER_PLUGIN_EXPORT
#endif

FLUTTER_PLUGIN_EXPORT void flutter_sound_plugin_register_with_registrar(
    FlPluginRegistrar* registrar);

G_END_DECLS

#endif  // FLUTTER_PLUGIN_FLUTTER_SOUND_PLUGIN_H_
EOF
```

**5.3 — Добавить алиас-функцию в исходник:**

```bash
# Добавить include в начало файла
sed -i '1a #include "include/flutter_sound/flutter_sound_plugin.h"' $SOUND_DIR/taudio_plugin.cc

# Добавить функцию-алиас в конец файла
cat >> $SOUND_DIR/taudio_plugin.cc << 'EOF'

void flutter_sound_plugin_register_with_registrar(FlPluginRegistrar* registrar) {
  taudio_plugin_register_with_registrar(registrar);
}
EOF
```

---

## Шаг 6 — Сборка xchat

```bash
cd ~/xchat-app-main

# Скачать все Dart/Flutter пакеты проекта
flutter pub get

# Собрать Linux-бинарник (лог пишется и в терминал и в файл)
flutter build linux --release 2>&1 | tee ~/Desktop/xchat-build.log
```

Сборка займёт 30–60 минут (много Dart-кода, HDD медленный).

Готовый бинарник:
```
~/xchat-app-main/build/linux/x64/release/bundle/ox_chat
```

---

## Запуск

```bash
~/xchat-app-main/build/linux/x64/release/bundle/ox_chat
```

---

## Настройки после запуска

**Relay (для сообщений):**
```
wss://bendernostur.duckdns.org:8443
```

**File server / Media (для фото):**
```
https://bendernostur.duckdns.org:8444
```

Регистрация не нужна — сервер принимает загрузки от всех (NIP-96, allowPublicUploads).

---

## Если что-то пошло не так

- `flutter doctor` — покажет что не хватает
- `flutter clean && flutter pub get` — сбросить кэш и переустановить пакеты
- Логи сборки: `~/Desktop/xchat-build.log` — читать последнюю ошибку снизу
