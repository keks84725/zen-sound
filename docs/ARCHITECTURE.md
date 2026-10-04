# Архитектура Zen Sound

## 1. Общие принципы

Zen Sound строится по модульной архитектуре, разделенной на 4 ключевых слоя:

```
┌────────────────────────────────────────────────────────┐
│           UI Layer (Flutter / Retro Theme Engine)      │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│            Core State & Playback Orchestrator          │
└──────────────┬──────────────────────────┬──────────────┘
               │                          │
┌──────────────▼────────────┐ ┌───────────▼──────────────┐
│       Audio Pipeline      │ │ Isolated Storage & Cache │
│ (libmpv / hardware decode)│ │   /cache/{service_id}/   │
└───────────────────────────┘ └──────────────────────────┘
               │                          │
┌──────────────┴──────────────────────────┴──────────────┐
│               Service Providers (Plugins)              │
│   [Subsonic API]  [Yandex Music]  [Spotify]  [VK]      │
└────────────────────────────────────────────────────────┘
```

---

## 2. Модули источников (Service Providers)

Каждый сервис реализует единый интерфейс `AudioSourceProvider`:

```typescript
interface AudioSourceProvider {
  id: string;              // 'yandex' | 'spotify' | 'vk' | 'subsonic'
  displayName: string;
  badgeColor: string;
  
  // Родные разделы сервиса
  getSections(): Promise<ServiceSection[]>;
  getTracks(sectionId: string): Promise<Track[]>;
  
  // Получение прямого потока для воспроизведения или кэша
  getStreamUrl(trackId: string): Promise<string>;
}
```

Все метаданные и токены авторизации хранятся в зашифрованном локальном ключевом хранилище ОС (Keyring / EncryptedSharedPreferences).

---

## 3. Изолированный кэш (Isolated Cache Vault)

Кэш каждого сервиса изолирован на уровне каталогов файловой системы:
* `%APPDATA%/zen-sound/cache/yandex/`
* `%APPDATA%/zen-sound/cache/spotify/`
* `%APPDATA%/zen-sound/cache/vk/`
* `%APPDATA%/zen-sound/cache/subsonic/`

Каждый трек сохраняется вместе с локальным метафайлом тегов. При офлайн-воспроизведении плеер в первую очередь проверяет локальный каталог активного сервиса. Очистка кэша выполняется через простое удаление подкаталога конкретного сервиса, исключая случайное удаление файлов из других источников.

---

## 4. Спецификация скинов (Theme Schema)

Скин представляет собой JSON-документ:

```json
{
  "name": "80s Braun",
  "author": "Zen Sound Community",
  "version": "1.0.0",
  "colors": {
    "chassis": "#EA580C",
    "faceplate": "#FBF7EE",
    "border": "#18181B",
    "accent": "#22C55E",
    "danger": "#EF4444",
    "text": "#18181B",
    "mutedText": "#71717A",
    "albumPlaceholder": "#BFDBFE"
  },
  "metrics": {
    "borderWidth": "3px",
    "shadowOffset": "5px",
    "borderRadius": "28px"
  }
}
```
