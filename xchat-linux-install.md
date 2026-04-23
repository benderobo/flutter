# Установка 0xChat (xchat) на Parrot OS / Linux

## Что это такое
0xChat — Nostr-мессенджер на Flutter. Исходники лежат в `~/xchat-app-main/`.
Flutter поддерживает сборку нативных Linux-приложений через GTK3.

---

## Шаг 1 — Системные зависимости

```bash
sudo apt install -y clang cmake ninja-build pkg-config libgtk-3-dev liblzma-dev libstdc++-12-dev
```

- `clang` + `cmake` + `ninja-build` — компиляторы и система сборки C++
- `pkg-config` — помогает находить установленные библиотеки
- `libgtk-3-dev` — GTK3, через него Flutter рисует окна на Linux
- `liblzma-dev` — сжатие, нужно некоторым зависимостям

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

Предупреждения про Android/Chrome/Xcode игнорируем — нам нужен только раздел Linux.

---

## Шаг 4 — Сборка xchat

```bash
cd ~/xchat-app-main

# Скачать все Dart/Flutter пакеты проекта
flutter pub get

# Собрать Linux-бинарник (Release = оптимизированная сборка)
flutter build linux --release
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

Или создать .desktop файл для запуска из меню.

---

## Настройки после запуска

**Relay (для сообщений):**
```
wss://name.domen.com:port
```

**File server / Media (для фото):**
```
https://name.domen.com:port
```

Регистрация не нужна — сервер принимает загрузки от всех (NIP-96, allowPublicUploads).

---

## Если что-то пошло не так

- `flutter doctor` — покажет что не хватает
- `flutter clean && flutter pub get` — сбросить кэш и переустановить пакеты
- Логи сборки: читать внимательно последнюю ошибку (обычно внизу)
