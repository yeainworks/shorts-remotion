---
name: shorts-remotion
description: >
  Делает готовый вертикальный шортс (YouTube Shorts / Reels / TikTok, 40–65 с) по идее пользователя
  ТОЛЬКО на Remotion (без Blender) с ЛЮБЫМ персонажем: свой маскот/стикер/рисунок пользователя (PNG,
  одна картинка или набор эмоций) оживает в 3D-локациях three.js — мягкий свет, размытие фона, движущаяся
  камера, эмоции, походка, прыжки, значки-эффекты; можно и без персонажа. Голос (ElevenLabs или бесплатный
  Piper), субтитры как у Ricks Clips, фоновая музыка. Используй, когда просят «сделай шортс/рилс/ролик про…»,
  «объясни (тему) в видео», «ролик с моим персонажем/маскотом», «ролик как у Ricks Clips».
---

# Shorts (Remotion) — шортс с любым персонажем, от идеи до mp4

Всё нужное — в этом файле: правила стиля, порядок работы и **весь код** проекта (его надо записать
на диск как есть). Работай строго по шагам. Пользователь может ничего не знать про видео и 3D —
объясняй простыми словами. `<P>` — папка ролика (латиница, без пробелов, напр. `short-pizza`).

**Персонаж не зашит в код.** Герой ролика — картинки пользователя в `public/chars/<id>/`. Код сам даёт им
жизнь: дыхание, моргание, «бонк» на смене эмоции, рот/качание на репликах, походку, прыжки, наклоны,
разворот и значки (пот, «!», «?», злость, блёстки, «…»). Персонажей может быть несколько или ни одного.

---

## Шаг 0. Проверка окружения
Нужны **Node.js 18+** и **ffmpeg** (больше ничего — Blender не нужен).
- `node -v` → если нет: https://nodejs.org (кнопка LTS), установить, ПЕРЕЗАПУСТИТЬ Claude/терминал.
- `ffmpeg -version` → если нет: Windows `winget install Gyan.FFmpeg`; Mac — https://brew.sh, затем `brew install ffmpeg`.
- Для голоса ElevenLabs — расширение «Claude» для Chrome (Chrome Web Store) и аккаунт elevenlabs.io
  (вход делает сам пользователь). Без них — бесплатный голос Piper.
- Интернет нужен при установке и при рендере (шрифт субтитров грузится из Google Fonts).

Частые проблемы: `node`/`npx` не найдены после установки → перезапустить терминал; рендер падает на WebGL →
в `remotion.config.ts` поменять `"angle"` на `"swiftshader"` (медленнее, но работает везде);
whisper на Windows ругается на `Expand-Archive` → уже обходится в `tools/whisper.mjs`.

## Шаг 1. Идея, язык и персонаж
Ролик — на языке пользователя. Если язык неясен — спроси. Если идея размыта — предложи 3 темы-хука по
одной строке и попроси выбрать.

**Спроси про персонажа** (если пользователь уже дал картинку — не спрашивай):
- **Свой персонаж** — PNG с прозрачным фоном (стикер, маскот канала, рисунок). Минимум одна картинка
  (`neutral`). Лучше набор эмоций — те же размер холста и положение персонажа, меняется только мимика/поза.
  Белый фон без прозрачности — уберётся флагом `--key-white`.
- **Несколько героев** — у каждого своя папка-id (`hero`, `boss`, `cat`…).
- **Без персонажа** — ролик на предметах, вывесках и камере (Actor просто не добавляются).

Если персонаж нарисован кодом (HTML/SVG/скрипт сборки в проекте пользователя) — недостающие эмоции сделай
тем же способом, сохранив стиль: глаза, брови, рот, поза. Сам персонажа не выдумывай и не генерируй нейросетью.

**Имена эмоций** (файл = `<эмоция>.png`, любое своё имя тоже работает): `neutral` (обязательна), `happy`,
`smug`, `proud`, `laugh`, `nervous`, `shock`, `realize`, `confused`, `angry`, `sad`, `blank`.
Служебные (необязательно): `talk` — рот открыт (чередуется с `neutral` на репликах), `blink` — глаза
закрыты, `<эмоция>_talk`, `<эмоция>_blink`. Нет картинки эмоции → берётся `neutral`, а эмоцию передают
значки-эффекты и тряска (см. Шаг 5).

## Шаг 2. Сценарий (ОБЯЗАТЕЛЬНО прочитай «Приложение А: эталонные сценарии» в конце файла)
Пиши в той же манере и на том же уровне, **копировать нельзя** — повторяй структуру и подачу.

**Формат:** 110–160 слов (40–60 с), **6–9 абзацев = шотов** (каждый 4–8 с). Фразы 5–18 слов; после длинной —
короткая (2–5 слов). Новая картинка каждые 1–2 фразы.

**Структура:**
1. **Хук (первая фраза, ≤2.5 с):** парадокс / контринтуитивное утверждение / прямой вызов зрителю. Сразу даёт
   картинку. Без «Привет», без «сегодня расскажу».
2. **«Представь…» / «Ты…»** — зритель становится героем; дальше во втором лице.
3. **Смешные конкретные детали** (числа, предметы, повторы-ритм) вместо общих слов.
4. **«Ружьё»** — деталь, которая вернётся в финале.
5. **Вопрос вслух** ~на 40%: «Как такое возможно?» — ответ не сразу.
6. **Бытовая аналогия** из жизни каждого.
7. **Механизм простыми словами + один реальный факт/число** (правда! не уверен — смягчи формулировку).
8. **Финал-«кнопка»**: закольцовка с «ружьём» или короткое масштабное последствие, ≤10 слов. Без морали и CTA.

Короткие реплики героев («Опять 40 секунд!») — в кавычках внутри текста диктора: они дают поводы для эмоций.
Если у персонажа есть характер (ленивый, ворчливый, наивный…) — пусть эмоции и реплики ему соответствуют.

**Покажи пользователю** текст + раскадровку таблицей (абзац → что в кадре → кто что делает/эмоция) и спроси
«Сценарий ок?». **Дальше — только после явного одобрения.** Используй 2–4 локации на ролик.
В раскадровке используй только эмоции, для которых есть картинки, либо явно помечай «эффект ×, без картинки».

## Шаг 3. Создать проект
1. Создай `<P>` и запиши в него **все файлы из «Приложения Б»** ровно как есть (пути — как в заголовках).
2. `cd <P>` → `npm install` (1–3 мин).
3. **Персонажи:** для каждой картинки
   `node tools/char.mjs add <id> <путь/к/картинке.png> [эмоция] [--h 1.3] [--bust] [--key-white]`
   - `<id>` — латиница (`hero`, `frog`…); эмоция по умолчанию `neutral`;
   - `--h` — рост в метрах на сцене (дверь 2.1 м, стол 0.75 м; человек ~1.6, маскот 0.9–1.4, кот 0.4);
   - `--bust` — картинка обрезана снизу (только голова/плечи): нет тени у ног, ставь героя за стол, стойку,
     руль или нижний край кадра, поднимая `pos[1]`;
   - `--key-white` — сделать белый фон прозрачным.
   Положил/удалил файлы в `public/chars/` руками → `node tools/char.mjs scan`. Список — в `src/chars.json`.
   Нет картинки для id из `Video.tsx` → рисуется серая заглушка-силуэт (так и задумано для примера).
4. Запиши текст для озвучки в `script/voice.txt` (абзацы через пустую строку, можно теги эмоций
   ElevenLabs вида `[shouting]`) и тот же текст для субтитров в `script/voice_display.txt` (без тегов,
   числа цифрами: «40 секунд», «85%»).

## Шаг 4. Голос
**Правила:** без согласия пользователя нигде не регистрироваться, ничего не покупать, не вводить карту/пароли.
Перед генерацией в ElevenLabs назови расход кредитов (≈ символы × 0.5 для v4 Turbo; генерируется 2 варианта)
и спроси разрешения. **Для монетизации бесплатный тариф ElevenLabs не подходит** (нет коммерческой лицензии) —
предупреди.

**A. ElevenLabs через браузер (лучше):** новая вкладка `https://elevenlabs.io/app/speech-synthesis/text-to-speech` →
Voice: русский мужской «Stanislav – Deep, Empathetic and Warm» (англ. — «Adam»/«Brian»; нет в списке —
Voice Library → Add; у канала есть свой голос — бери его) → Model: **Eleven v4 Turbo** (нет — Multilingual v2) →
Stability 0.45, Similarity 0.8, Style 0.25, Speed 1.0 → вставить текст → Generate → выбрать живой вариант → ↓ скачать.
Если в Chrome включено «Спрашивать, куда сохранять», файл висит как `.tmp` в папке загрузок — возьми самый
свежий по размеру. Ускорь на 10% без смены тона: `ffmpeg -y -i скачанный.mp3 -af atempo=1.1 public/voice.mp3`.

**B. Бесплатно, без регистрации — Piper** (честно: роботизированнее, без эмоций; для черновиков и роликов без
монетизации): `node tools/tts_piper.mjs script/voice.txt public/voice.wav ru_RU-dmitri-medium 1.08`
(голоса: `ru_RU-irina-medium`, `en_US-ryan-high`, `de_DE-thorsten-medium`, `es_ES-davefx-medium`…).
Тогда в `script/edit.json`: `{"voice": "voice.wav"}`.

**Тайминги слов (обязательно):** `node tools/whisper.mjs public/voice.mp3 ru` (язык ролика) → `audio/words.json`
и список «секунда слово». По нему: **начало каждого шота = время первого слова абзаца**; время ударных слов =
моменты эмоций, прыжков, эффектов и «бонков». Затем: `node tools/plan.mjs` (субтитры) и
`node tools/music.mjs <длина_сек+3> playful` (музыка без лицензий; настроения `playful|tense|calm`).

## Шаг 5. Шоты — правишь ТОЛЬКО `src/Video.tsx`
Файл уже содержит рабочий пример из 2 шотов (холл с лифтом) — **прочитай его целиком** и замени на свои шоты.
Всё время в секундах ГОЛОСА. Шот = `{ t0, t1, el: <Scene3D …>…</Scene3D> }`.

**Набор (`src/kit.tsx`):**
- Локация: `<Room wall floor wain ceil window sky />` (комната 6×4×3 м, задняя стена z=−2, окно слева),
  `<Box p s c />` (любой предмет-коробка с обводкой рёбер), `<Desk/> <Chair/> <Shelf/> <Plant/> <Bench/>
  <Monitor/> <Mirror/> <Door/> <Lamp/> <Clock hh mm/> <Sign text bg fg/>` (таблички, вывески, постеры).
  Для улицы/транспорта/кафе/болота/космоса — собирай из `Box` + `Sign` по тем же правилам.
- Персонаж: объект `Actor` → `<Character a={…} />`. Поля:
  - `char` — id из `src/chars.json`; `pos` [x, y, z] (y — высота опоры: 0 пол, 0.75 «за столом» для `--bust`);
    `seed` — число (разная фаза дыхания/моргания); `h` — рост на этот ролик (иначе из `chars.json`);
  - `expr` [[сек, эмоция], …] — смена картинки + «бонк» на каждой смене;
  - `talk` [[с, по], …] — реплики: рот (`talk`-картинки) или ритмичное качание, если их нет;
  - `walk` [[сек, x, z], …] — перемещение вразвалку; `hop` [сек, …] — прыжки (присед → прыжок → приземление);
  - `lean` [[сек, угол], …] — наклон в радианах (+ влево, 0.15 — заметно); `face` [[сек, 1|−1], …] —
    развернуть зеркально (смотреть на собеседника); `fx` [[сек, эффект, длит.], …] — значок вручную;
  - `hidden` [[с, по]] — скрыть (ракурс «от лица» героя).
  Эффекты: `sweat`, `bang` («!»), `question` («?»), `anger`, `sparkle`, `dots` («…»). Сами появляются на
  эмоциях: nervous→пот, shock→пот+«!», realize→«!», confused→«?», angry/stern→злость, proud/smug→блёстки,
  blank→«…»; laugh/shock/angry ещё и трясутся.
- Шот: `<Scene3D t0 cam={[[сек,[x,y,z],[куда смотрит],fov], …]} focus={[x,y,z]} light="day|night|studio">`.
- Предмет «бонкает» на ударном слове: `scale={bonkAt(useSec(), секунда)}` (внутри компонента сцены).

**Правила постановки (каждое — исправленная ошибка):**
- **Эмоции меняются часто** — у каждого героя смена каждые 1–2 с по словам; «бонк» срабатывает сам на
  смене. Нельзя случайных периодических «бонков» (герои «дёргаются»).
- Нет картинки нужной эмоции → не подменяй её чужой: `neutral` + эффект + действие (прыжок, наклон, тряска).
- **Персонаж плоский — ставь его лицом к камере**: камера в секторе ±40° от фронта героя, иначе он «картонный».
  Разговор двоих — `face`, чтобы смотрели друг на друга.
- **Жест и его цель — в одном кадре.** «Указать» = герой рядом с предметом + `bonkAt` предмета на слове.
  Никаких наездов-зумов на предмет.
- **Больше общих и средних планов**: камера в 1.5–2.5 м от героев, высота ≈ 0.8×рост героя (но 1.0–1.35 для
  людей), fov 70–78; камера всегда медленно дрейфует (0.2–0.4 м за шот).
- **Фокус (`focus`) — на говорящем/главном герое** (точка на уровне его лица): фон и передний план размываются сами.
- Расставляй героев **под каждый шот** там, где действие (у зеркала, у двери); все участники — в кадре;
  масштаб героя к предметам — по `h` (маленький маскот не выше двери).
- **8–15 предметов в кадре** общего плана, надписи на предметах — на языке ролика. Пустая комната = «нейрослоп».
- **Свет мягкий, без полос жалюзи** (пользователь их забраковал). Не ставь ярких ламп близко к камере — пересвет.
- Зеркала/стекло — стилизованные (`Mirror`), не «настоящие» отражения.

## Шаг 6. Проверка кадров и рендер
- **Кадры (обязательно после каждого шота):** `npx remotion still src/index.ts Video checks/s1.png --frame=<кадр>`
  (кадр = секунда × 30). Открой PNG и пройди «Чек-лист» ниже. Исправь → снова кадр.
- **Превью (по умолчанию):** `npx remotion render src/index.ts Video out/preview.mp4 --scale=0.5 --concurrency=4`
  (540×960, ~1–2 мин на ролик). Скажи пользователю, что это превью.
- **Финал (полное качество 1080×1920) — по команде пользователя «финальный рендер»:**
  `npx remotion render src/index.ts Video out/final.mp4 --concurrency=4` (≈ 3–4 с рендера на секунду видео).
  Ничего не урезать ради скорости.
- Скопируй результат в корень `<P>` под понятным именем и сообщи **полный путь** к файлу.

## Чек-лист перед отдачей
- [ ] Кадр 1: уже действие/герой, хук звучит с 0.0–0.2 с, первый субтитр в первые 0.3 с.
- [ ] Герои — картинки пользователя без фона-«квадрата» и без серых заглушек; масштаб к предметам верный;
      стоят на полу (или за предметом, если `--bust`); лицом к камере.
- [ ] Герои живые: эмоции меняются по словам, дышат, на репликах говорят/качаются; без случайных дёрганий.
- [ ] Действия понятны, действие и цель в одном кадре.
- [ ] Свет мягкий, без пересвета; фон размыт, герой резкий; 8–15 предметов, есть цветовой акцент.
- [ ] Каждый шот начинается на первом слове абзаца (±0.2 с); эмоции/прыжки/эффекты — на ударных словах.
- [ ] Субтитры: Montserrat Black, белые с чёрной обводкой, 1–3 слова, центр-низ, текст = сценарий.
- [ ] Только голос и музыка (без звуковых эффектов), музыка тише голоса (0.11, макс 0.18).
- [ ] 40–65 с, 1080×1920 (финал), файл открывается целиком.

## Правки пользователя
Меняй только нужное: эмоция/действие/камера — массивы этого шота в `Video.tsx`; новая эмоция-картинка →
`tools/char.mjs add`; текст — новый голос → `whisper` → сдвинуть `t0/t1` и ключи шотов после правки →
`plan.mjs`; субтитры/музыка — `edit.json`/`music.mjs`.
Перерендер быстрый, поэтому рендерь ролик целиком (превью), показывай кадры «до/после».

## Нельзя (анти-«нейрослоп»)
Выдуманные или сгенерированные нейросетью персонажи вместо картинок пользователя; чужая эмоция вместо нужной;
пустые локации; статичная камера и наезды на каждый объект; одна эмоция на весь шот; случайные «дёрганья»;
плашки-титры кроме субтитров; звуковые эффекты; музыка громче голоса.

---

# Приложение Б: файлы проекта (записать как есть)

### `package.json`
```json
{
  "name": "shorts-remotion",
  "private": true,
  "type": "module",
  "dependencies": {
    "@react-three/fiber": "8.17.10",
    "@react-three/postprocessing": "2.16.3",
    "@remotion/cli": "4.0.250",
    "@remotion/google-fonts": "4.0.250",
    "@remotion/install-whisper-cpp": "4.0.250",
    "@remotion/three": "4.0.250",
    "postprocessing": "6.36.4",
    "react": "18.3.1",
    "react-dom": "18.3.1",
    "remotion": "4.0.250",
    "three": "0.169.0"
  },
  "optionalDependencies": { "sherpa-onnx-node": "1.12.15" },
  "devDependencies": { "@types/three": "0.169.0", "@types/react": "18.3.12" }
}
```

### `tsconfig.json`
```json
{ "compilerOptions": { "target": "ES2022", "module": "ES2022", "jsx": "react-jsx", "strict": false, "moduleResolution": "bundler",
  "esModuleInterop": true, "skipLibCheck": true, "resolveJsonModule": true, "lib": ["dom", "ES2022"] } }
```

### `remotion.config.ts`
```ts
import { Config } from "@remotion/cli/config";
Config.setChromiumOpenGlRenderer("angle");
```

### `src/index.ts`
```ts
import { registerRoot } from "remotion";
import { Root } from "./Root";
registerRoot(Root);
```

### `src/Root.tsx`
```tsx
import React from "react";
import { Composition } from "remotion";
import { Video, TOTAL } from "./Video";
export const Root: React.FC = () => <Composition id="Video" component={Video} durationInFrames={TOTAL} fps={30} width={1080} height={1920} />;
```

### `src/chars.json`
```json
{}
```
(Его перезаписывает `tools/char.mjs` — руками не править, кроме `h` и `bust`.)

### `src/char.ts`
```ts
// Универсальный персонаж: ЛЮБАЯ картинка PNG с прозрачным фоном (маскот, стикер, рисунок пользователя).
// Картинки: public/chars/<id>/<эмоция>.png, список — src/chars.json (пишет tools/char.mjs).
// Здесь только загрузка и рисование кадра на Canvas 2D; анимацию считает Character в kit.tsx.
import { useEffect, useState } from "react";
import { continueRender, delayRender, staticFile } from "remotion";
import CHARS from "./chars.json";

export type CharDef = { h: number; bust?: boolean; files: Record<string, string> };
type Box4 = [number, number, number, number];                       // x0, y0, x1, y1 в пикселях исходника
export type CharSet = { def: CharDef; imgs: Record<string, HTMLImageElement>; box: Box4 };
export type CharPose = {
  expr: string; blink: boolean; talk: number; squash: number; tilt: number; flip: number;
  shake: [number, number]; fx: [string, number][]; seed: number;   // fx: [эффект, сколько секунд уже показан]
};

const INK = "#2a1d1a";
// Значки, которые сами появляются на эмоции; и эмоции с тряской.
export const FX_OF: Record<string, string[]> = {
  nervous: ["sweat"], shock: ["sweat", "bang"], realize: ["bang"], confused: ["question"],
  angry: ["anger"], stern: ["anger"], proud: ["sparkle"], smug: ["sparkle"], blank: ["dots"],
};
export const SHAKY = ["laugh", "shock", "angry", "holdlaugh"];

// ------------------------------------------------------------------ загрузка (один раз на картинку)
const cache = new Map<string, Promise<{ img: HTMLImageElement; box: Box4 } | null>>();
const opaqueBox = (im: HTMLImageElement): Box4 => {
  const s = Math.min(1, 256 / Math.max(im.width, im.height));
  const w = Math.max(1, Math.round(im.width * s)), h = Math.max(1, Math.round(im.height * s));
  const cv = document.createElement("canvas"); cv.width = w; cv.height = h;
  const g = cv.getContext("2d")!; g.drawImage(im, 0, 0, w, h);
  const a = g.getImageData(0, 0, w, h).data;
  let x0 = w, y0 = h, x1 = -1, y1 = -1;
  for (let y = 0; y < h; y++) for (let x = 0; x < w; x++) if (a[(y * w + x) * 4 + 3] > 20) {
    if (x < x0) x0 = x; if (x > x1) x1 = x; if (y < y0) y0 = y; if (y > y1) y1 = y;
  }
  return x1 < 0 ? [0, 0, im.width, im.height] : [x0 / s, y0 / s, (x1 + 1) / s, (y1 + 1) / s];
};
const load = (src: string) => {
  if (!cache.has(src)) cache.set(src, new Promise((res) => {
    const im = new Image();
    im.onload = () => res({ img: im, box: opaqueBox(im) });
    im.onerror = () => res(null);
    im.src = src;
  }));
  return cache.get(src)!;
};

// null — персонажа нет в chars.json (рисуется заглушка), "loading" — ещё грузится.
export const useChar = (id: string): CharSet | null | "loading" => {
  const def = (CHARS as Record<string, CharDef>)[id];
  const [st, setSt] = useState<CharSet | null | "loading">(def ? "loading" : null);
  const [handle] = useState(() => (def ? delayRender("персонаж " + id) : null));
  useEffect(() => {
    if (handle === null) return;
    const names = Object.keys(def.files);
    Promise.all(names.map((n) => load(staticFile(`chars/${id}/${def.files[n]}`)))).then((rs) => {
      const imgs: Record<string, HTMLImageElement> = {};
      const box: Box4 = [Infinity, Infinity, -Infinity, -Infinity];   // общая рамка всех эмоций — размер не скачет
      rs.forEach((r, i) => {
        if (!r) return;
        imgs[names[i]] = r.img;
        box[0] = Math.min(box[0], r.box[0]); box[1] = Math.min(box[1], r.box[1]);
        box[2] = Math.max(box[2], r.box[2]); box[3] = Math.max(box[3], r.box[3]);
      });
      setSt(Object.keys(imgs).length ? { def, imgs, box } : null);
      continueRender(handle);
    });
  }, [handle]);
  return st;
};

// Какую картинку показать: эмоция → её talk/blink-вариант → общий talk/blink (только для neutral) → neutral.
export const pickImage = (imgs: Record<string, HTMLImageElement>, p: CharPose): HTMLImageElement | undefined => {
  const base = imgs[p.expr] ? p.expr : "neutral";
  const alt = p.talk > 0.5 ? "talk" : p.blink ? "blink" : null;
  if (alt && imgs[`${base}_${alt}`]) return imgs[`${base}_${alt}`];
  if (alt && base === "neutral" && imgs[alt]) return imgs[alt];
  return imgs[base] ?? Object.values(imgs)[0];
};

// ------------------------------------------------------------------ рисование кадра
// Холст: персонаж W×H метров стоит по центру у нижнего края (отступ FOOT), K — пикселей на метр.
export const FOOT = 0.03;
export function drawChar(ctx: CanvasRenderingContext2D, set: CharSet | null, p: CharPose, W: number, H: number, K: number) {
  const cw = ctx.canvas.width, ch = ctx.canvas.height, cx = cw / 2, fy = ch - FOOT * K;
  ctx.clearRect(0, 0, cw, ch);
  ctx.lineCap = "round"; ctx.lineJoin = "round";
  if (!set?.def.bust) {                                                  // тень у ног
    ctx.fillStyle = "rgba(0,0,0,0.22)"; ctx.beginPath();
    ctx.ellipse(cx, fy, W * 0.42 * K, 0.035 * K, 0, 0, Math.PI * 2); ctx.fill();
  }
  ctx.save();
  ctx.translate(cx + p.shake[0] * K, fy + p.shake[1] * K);
  ctx.rotate(-p.tilt);
  ctx.scale(p.flip / Math.sqrt(p.squash), p.squash);                    // объём сохраняется: ниже → шире
  const img = set ? pickImage(set.imgs, p) : undefined;
  if (img && set) {
    const [x0, y0, x1, y1] = set.box;
    ctx.drawImage(img, x0, y0, x1 - x0, y1 - y0, (-W / 2) * K, -H * K, W * K, H * K);
  } else placeholder(ctx, W * K, H * K, p);
  ctx.restore();
  const top = fy - H * K * p.squash;
  for (const [fx, age] of p.fx) drawFx(ctx, fx, age, cx + p.flip * W * 0.42 * K, top, H * K);
}

// Заглушка, пока нет картинки: серый силуэт с глазами (видно, что персонажа надо добавить).
const placeholder = (ctx: CanvasRenderingContext2D, w: number, h: number, p: CharPose) => {
  ctx.fillStyle = `hsl(${(p.seed * 67) % 360},12%,72%)`; ctx.strokeStyle = INK; ctx.lineWidth = w * 0.04;
  ctx.beginPath(); ctx.roundRect(-w / 2, -h, w, h, w * 0.45); ctx.fill(); ctx.stroke();
  ctx.fillStyle = INK;
  for (const s of [-1, 1]) {
    ctx.beginPath(); ctx.ellipse(s * w * 0.17, -h * 0.72, w * 0.06, p.blink ? w * 0.012 : w * 0.08, 0, 0, Math.PI * 2); ctx.fill();
  }
  ctx.lineWidth = w * 0.03; ctx.beginPath();
  if (p.talk > 0.5) ctx.ellipse(0, -h * 0.55, w * 0.08, w * 0.06, 0, 0, Math.PI * 2); else { ctx.moveTo(-w * 0.12, -h * 0.55); ctx.lineTo(w * 0.12, -h * 0.55); }
  ctx.stroke();
};

// Значки у головы: (x, top) — правый верх персонажа, u — его рост в пикселях. Появляются с «выстрелом».
const drawFx = (ctx: CanvasRenderingContext2D, fx: string, age: number, x: number, top: number, u: number) => {
  const pop = age < 0.12 ? (age / 0.12) * 1.25 : 1 + 0.25 * Math.exp(-(age - 0.12) * 12);
  const s = u * 0.16 * pop;
  ctx.save(); ctx.translate(x, top + u * 0.08); ctx.scale(s / 100, s / 100);
  ctx.strokeStyle = INK; ctx.lineWidth = 7;
  const glyph = (txt: string, col: string) => {
    ctx.font = "900 110px Arial Black, Arial, sans-serif"; ctx.textAlign = "center"; ctx.textBaseline = "middle";
    ctx.lineWidth = 18; ctx.strokeStyle = "#fff"; ctx.strokeText(txt, 0, 0); ctx.fillStyle = col; ctx.fillText(txt, 0, 0);
  };
  if (fx === "sweat") {
    ctx.translate(0, 40 + Math.min(age, 1.5) * 25);                    // капля сползает
    ctx.fillStyle = "#c9e6ff"; ctx.strokeStyle = "#3d7fd0"; ctx.lineWidth = 6;
    ctx.beginPath(); ctx.moveTo(0, -45); ctx.quadraticCurveTo(32, 5, 0, 22); ctx.quadraticCurveTo(-32, 5, 0, -45); ctx.fill(); ctx.stroke();
  } else if (fx === "bang") glyph("!", "#ff3b2f");
  else if (fx === "question") glyph("?", "#2f6fd6");
  else if (fx === "dots") { ctx.translate(0, -20); glyph("…", INK); }
  else if (fx === "anger") {                                            // «венка» злости: 4 дуги
    ctx.strokeStyle = "#e0262b"; ctx.lineWidth = 14; ctx.translate(0, -30);
    for (let i = 0; i < 4; i++) { ctx.save(); ctx.rotate((i * Math.PI) / 2); ctx.beginPath(); ctx.arc(32, 32, 24, Math.PI, Math.PI * 1.5); ctx.stroke(); ctx.restore(); }
  } else if (fx === "sparkle") {
    ctx.fillStyle = "#ffd23f"; ctx.strokeStyle = INK; ctx.lineWidth = 5;
    [[0, -30, 1], [-55, 10, 0.6]].forEach(([dx, dy, k], i) => {
      const tw = k * (0.75 + 0.25 * Math.sin(age * 9 + i * 2));
      ctx.save(); ctx.translate(dx, dy); ctx.scale(tw, tw); ctx.beginPath();
      for (let j = 0; j < 8; j++) { const r = j % 2 ? 12 : 40, a = (j * Math.PI) / 4; ctx.lineTo(Math.sin(a) * r, -Math.cos(a) * r); }
      ctx.closePath(); ctx.fill(); ctx.stroke(); ctx.restore();
    });
  }
  ctx.restore();
};
```

### `src/kit.tsx`
```tsx
// Набор для сцен с любыми персонажами на three.js внутри Remotion.
// Всё в метрах: x вправо, y вверх, z к камере. Задняя стена комнаты при z = −d/2.
import React, { createContext, useContext, useMemo } from "react";
import * as THREE from "three";
import { ThreeCanvas } from "@remotion/three";
import { useThree } from "@react-three/fiber";
import { EffectComposer, DepthOfField, Bloom, Vignette } from "@react-three/postprocessing";
import { LineSegments2 } from "three/examples/jsm/lines/LineSegments2.js";
import { LineSegmentsGeometry } from "three/examples/jsm/lines/LineSegmentsGeometry.js";
import { LineMaterial } from "three/examples/jsm/lines/LineMaterial.js";
import { Easing, useCurrentFrame, useVideoConfig } from "remotion";
import { CharPose, FOOT, FX_OF, SHAKY, drawChar, useChar } from "./char";

export const FPS = 30;
export const LINE = "#20180f";
type V3 = [number, number, number];

// ------------------------------------------------------------------ время шота
// Каждый шот получает t0 — секунду голоса, с которой он начинается. sec() = текущая секунда ГОЛОСА.
const ShotCtx = createContext<{ t0: number; cam: THREE.Vector3 }>({ t0: 0, cam: new THREE.Vector3() });
export const useSec = () => { const f = useCurrentFrame(); const { t0 } = useContext(ShotCtx); return t0 + f / FPS; };

// ------------------------------------------------------------------ обводка рёбер (как Line Art)
let lineMat: LineMaterial | null = null;
const getLineMat = (w: number, h: number) => {
  if (!lineMat) lineMat = new LineMaterial({ color: LINE, linewidth: 3.2, worldUnits: false });
  lineMat.resolution.set(w, h);
  return lineMat;
};

export const Box: React.FC<{ p: V3; s: V3; c: string; r?: number; rot?: V3; e?: string; ei?: number; metal?: number;
  line?: boolean; shadow?: boolean; map?: THREE.Texture | null }> =
  ({ p, s, c, r = 0.75, rot = [0, 0, 0], e, ei = 0, metal = 0, line = true, shadow = true, map = null }) => {
    const { size } = useThree();
    const edges = useMemo(() => {
      if (!line) return null;
      const eg = new THREE.EdgesGeometry(new THREE.BoxGeometry(...s));
      return new LineSegments2(new LineSegmentsGeometry().setPositions(eg.attributes.position.array as Float32Array), getLineMat(size.width, size.height));
    }, [line, s[0], s[1], s[2]]);
    return (
      <group position={p} rotation={rot}>
        <mesh castShadow={shadow} receiveShadow>
          <boxGeometry args={s} />
          <meshStandardMaterial color={c} roughness={r} metalness={metal} emissive={e ?? "#000"} emissiveIntensity={ei} map={map} />
        </mesh>
        {edges ? <primitive object={edges} /> : null}
      </group>
    );
  };

// ------------------------------------------------------------------ текстуры из Canvas (надписи, часы)
export const canvasTex = (w: number, h: number, draw: (g: CanvasRenderingContext2D) => void) => {
  const cv = document.createElement("canvas"); cv.width = w; cv.height = h;
  draw(cv.getContext("2d")!);
  const t = new THREE.CanvasTexture(cv); t.colorSpace = THREE.SRGBColorSpace; t.anisotropy = 8; return t;
};
export const label = (txt: string, bg: string, fg: string, w = 512, h = 176, sz = 110) => canvasTex(w, h, (g) => {
  g.fillStyle = bg; g.fillRect(0, 0, w, h); g.fillStyle = fg; g.font = `900 ${sz}px Arial Black, Arial, sans-serif`;
  g.textAlign = "center"; g.textBaseline = "middle"; g.fillText(txt, w / 2, h / 2 + 4);
});

// Плоская табличка/вывеска/постер на стене (смотрит на камеру, +z). glow — подсветка.
export const Sign: React.FC<{ p: V3; w: number; h: number; text: string; bg: string; fg: string; glow?: number; rotY?: number; sz?: number }> =
  ({ p, w, h, text, bg, fg, glow = 0.25, rotY = 0, sz }) => {
    const tex = useMemo(() => label(text, bg, fg, 512, Math.round((512 * h) / w), sz ?? Math.min(110, Math.round(((512 * h) / w) * 0.6))), [text, bg, fg, w, h]);
    return <mesh position={p} rotation={[0, rotY, 0]}><planeGeometry args={[w, h]} />
      <meshStandardMaterial map={tex} emissive="#ffffff" emissiveMap={tex} emissiveIntensity={glow} /></mesh>;
  };

// Настенные часы: hh:mm. bonk — масштаб всей группы (НЕ только циферблата).
export const Clock: React.FC<{ p: V3; r?: number; hh?: number; mm?: number; scale?: number }> = ({ p, r = 0.17, hh = 10, mm = 10, scale = 1 }) => {
  const tex = useMemo(() => canvasTex(256, 256, (g) => {
    g.fillStyle = "#fbf7ea"; g.beginPath(); g.arc(128, 128, 120, 0, Math.PI * 2); g.fill(); g.strokeStyle = "#1d1f23"; g.lineWidth = 14; g.stroke();
    for (let i = 0; i < 12; i++) { const a = (i * Math.PI) / 6; g.lineWidth = i % 3 ? 4 : 8; g.beginPath();
      g.moveTo(128 + Math.sin(a) * 92, 128 - Math.cos(a) * 92); g.lineTo(128 + Math.sin(a) * 106, 128 - Math.cos(a) * 106); g.stroke(); }
    g.font = "900 30px Arial"; g.fillStyle = "#1d1f23"; g.textAlign = "center"; g.textBaseline = "middle";
    [["12", 0], ["3", 3], ["6", 6], ["9", 9]].forEach(([n, i]) => { const a = ((i as number) * Math.PI) / 6; g.fillText(n as string, 128 + Math.sin(a) * 70, 128 - Math.cos(a) * 70); });
    const ha = (((hh % 12) + mm / 60) / 12) * Math.PI * 2, ma = (mm / 60) * Math.PI * 2;
    g.lineWidth = 9; g.beginPath(); g.moveTo(128, 128); g.lineTo(128 + Math.sin(ha) * 55, 128 - Math.cos(ha) * 55); g.stroke();
    g.lineWidth = 6; g.beginPath(); g.moveTo(128, 128); g.lineTo(128 + Math.sin(ma) * 85, 128 - Math.cos(ma) * 85); g.stroke();
  }), [hh, mm]);
  return <mesh position={p} scale={[scale, scale, 1]}><circleGeometry args={[r, 40]} />
    <meshStandardMaterial map={tex} emissive="#fff7e0" emissiveMap={tex} emissiveIntensity={0.35} /></mesh>;
};

// ------------------------------------------------------------------ реквизит
export const Desk: React.FC<{ p: V3; rotY?: number; w?: number }> = ({ p, rotY = 0, w = 1.4 }) => (
  <group position={p} rotation={[0, rotY, 0]}>
    <Box p={[0, 0.75, 0]} s={[w, 0.05, 0.7]} c="#b8784a" r={0.6} />
    {[-1, 1].map((s) => <Box key={s} p={[s * (w / 2 - 0.2), 0.37, 0]} s={[0.36, 0.72, 0.63]} c="#7a4a2c" />)}
    {[-1, 1].flatMap((s) => [0.2, 0.45, 0.65].map((z) => <Box key={s + "" + z} p={[s * (w / 2 - 0.2), z, 0.33]} s={[0.12, 0.025, 0.02]} c="#c9ccd1" metal={0.8} r={0.3} line={false} />))}
  </group>
);
export const Chair: React.FC<{ p: V3; rotY?: number; c?: string }> = ({ p, rotY = 0, c = "#2f9e9a" }) => (
  <group position={p} rotation={[0, rotY, 0]}>
    <Box p={[0, 0.48, 0]} s={[0.48, 0.08, 0.46]} c={c} /><Box p={[0, 0.85, -0.22]} s={[0.48, 0.6, 0.07]} c={c} />
    <Box p={[0, 0.24, 0]} s={[0.07, 0.44, 0.07]} c="#2a2d33" /><Box p={[0, 0.03, 0]} s={[0.5, 0.05, 0.5]} c="#2a2d33" />
  </group>
);
export const Shelf: React.FC<{ p: V3; rotY?: number; w?: number; h?: number }> = ({ p, rotY = 0, w = 1.0, h = 1.8 }) => {
  const cols = ["#d64545", "#2f6fd6", "#f2c230", "#3f8a4b", "#2f9e9a", "#e46b2b", "#efe6d2"];
  const books: JSX.Element[] = [];
  for (let k = 0; k < 4; k++) { let x = -w / 2 + 0.06, i = 0; while (x < w / 2 - 0.1) {
    const bw = 0.04 + ((k * 7 + i * 13) % 6) * 0.01, bh = 0.22 + ((k * 5 + i * 3) % 5) * 0.03;
    books.push(<Box key={k + "_" + i} p={[x + bw / 2, 0.05 + (k * h) / 4 + bh / 2 + 0.015, 0]} s={[bw, bh, 0.24]} c={cols[(k + i) % 7]} line={bh > 0.3} />);
    x += bw + 0.005; i++; } }
  return (<group position={p} rotation={[0, rotY, 0]}>
    {[-1, 1].map((s) => <Box key={s} p={[(s * w) / 2, h / 2, 0]} s={[0.04, h, 0.32]} c="#7a4a2c" />)}
    {[0, 1, 2, 3].map((k) => <Box key={k} p={[0, 0.05 + (k * h) / 4, 0]} s={[w, 0.03, 0.32]} c="#7a4a2c" />)}{books}
  </group>);
};
export const Plant: React.FC<{ p: V3; h?: number }> = ({ p, h = 0.6 }) => (
  <group position={p}>
    <Box p={[0, 0.15, 0]} s={[0.26, 0.3, 0.26]} c="#b5653a" />
    {Array.from({ length: 8 }, (_, i) => <mesh key={i} position={[Math.cos(i) * 0.07, 0.3 + h * 0.4, Math.sin(i) * 0.07]} rotation={[Math.sin(i) * 0.45, i, Math.cos(i) * 0.45]} castShadow>
      <boxGeometry args={[0.07, h * 0.8, 0.02]} /><meshStandardMaterial color="#3f8a4b" /></mesh>)}
  </group>
);
export const Bench: React.FC<{ p: V3; rotY?: number; c?: string; w?: number }> = ({ p, rotY = 0, c = "#d07a3a", w = 1.6 }) => (
  <group position={p} rotation={[0, rotY, 0]}>
    <Box p={[0, 0.45, 0]} s={[w, 0.06, 0.45]} c={c} r={0.6} /><Box p={[0, 0.75, -0.22]} s={[w, 0.4, 0.06]} c={c} r={0.6} />
    {[-1, 1].map((s) => <Box key={s} p={[(s * (w - 0.2)) / 2, 0.22, 0]} s={[0.06, 0.44, 0.4]} c="#8d939b" />)}
  </group>
);
export const Monitor: React.FC<{ p: V3; rotY?: number; screen?: string }> = ({ p, rotY = 0, screen = "#3d7bd6" }) => (
  <group position={p} rotation={[0, rotY, 0]}>
    <Box p={[0, 0.32, 0]} s={[0.55, 0.34, 0.04]} c="#2a2d33" />
    <mesh position={[0, 0.32, 0.022]}><planeGeometry args={[0.5, 0.29]} /><meshBasicMaterial color={new THREE.Color(screen).multiplyScalar(1.4)} toneMapped={false} /></mesh>
    <Box p={[0, 0.1, -0.03]} s={[0.05, 0.2, 0.05]} c="#2a2d33" /><Box p={[0, 0.01, -0.02]} s={[0.25, 0.02, 0.18]} c="#2a2d33" />
  </group>
);
// Зеркало — СТИЛИЗОВАННОЕ (светлое стекло + блики). Настоящее отражение = тёмная «дыра».
export const Mirror: React.FC<{ p: V3; w?: number; h?: number; scale?: number }> = ({ p, w = 0.8, h = 1.2, scale = 1 }) => (
  <group position={p} scale={[scale, scale, scale]}>
    <Box p={[0, 0, 0]} s={[w + 0.1, h + 0.1, 0.05]} c="#c8a24a" r={0.35} metal={0.5} />
    <mesh position={[0, 0, 0.03]}><planeGeometry args={[w, h]} /><meshStandardMaterial color="#cfe3f2" emissive="#e8f4ff" emissiveIntensity={0.4} roughness={0.15} /></mesh>
    {[-0.12, 0.05].map((dx, i) => <mesh key={i} position={[dx * w * 4, 0.05, 0.035]} rotation={[0, 0, 0.45]}><planeGeometry args={[i ? 0.035 : 0.08, h * 0.85]} /><meshBasicMaterial color="#ffffff" transparent opacity={0.75} /></mesh>)}
  </group>
);
export const Door: React.FC<{ p: V3; c?: string; number?: string }> = ({ p, c = "#4f7aa6", number }) => (
  <group position={p}>
    <Box p={[0, 1.05, 0]} s={[0.95, 2.1, 0.06]} c={c} r={0.4} />
    <Box p={[-0.52, 1.08, 0]} s={[0.08, 2.16, 0.1]} c="#8d939b" /><Box p={[0.52, 1.08, 0]} s={[0.08, 2.16, 0.1]} c="#8d939b" /><Box p={[0, 2.18, 0]} s={[1.12, 0.08, 0.1]} c="#8d939b" />
    <Box p={[0.33, 1.0, 0.06]} s={[0.18, 0.04, 0.05]} c="#c9ccd1" metal={0.8} r={0.3} />
    {number ? <Sign p={[-0.75, 1.6, 0.02]} w={0.3} h={0.16} text={number} bg="#f6f3ea" fg="#222" glow={0.1} /> : null}
  </group>
);
export const Lamp: React.FC<{ p: V3; len?: number; glow?: number }> = ({ p, len = 1.4, glow = 1.02 }) => (
  <mesh position={p}><boxGeometry args={[len, 0.04, 0.16]} /><meshBasicMaterial color={[glow, glow * 0.96, glow * 0.88]} toneMapped={false} /></mesh>
);

// Комната: пол, потолок, задняя и боковые стены (с рёбрами-обводкой), панель снизу, окно слева (небо-плоскость).
export const Room: React.FC<{ w?: number; d?: number; h?: number; wall?: string; floor?: string; wain?: string; ceil?: string; window?: boolean; sky?: string }> =
  ({ w = 6, d = 4, h = 3, wall = "#4f8f95", floor = "#2f4fa8", wain = "#3f7479", ceil = "#d9d4c8", window = true, sky = "#bfe0ff" }) => {
    const L = -w / 2, B = -d / 2;
    return (<>
      <Box p={[0, -0.05, 0]} s={[w, 0.1, d]} c={floor} line={false} r={0.8} />
      <Box p={[0, h + 0.05, 0]} s={[w, 0.1, d]} c={ceil} />
      <Box p={[0, h / 2, B - 0.05]} s={[w, h, 0.1]} c={wall} r={0.85} />
      <Box p={[-L + 0.05, h / 2, 0]} s={[0.1, h, d]} c={wall} r={0.85} />
      {window ? (<>
        <Box p={[L - 0.05, h / 2, B + (d / 2 - 0.7) / 2]} s={[0.1, h, d / 2 - 0.7]} c={wall} />
        <Box p={[L - 0.05, h / 2, (1.1 + d / 2) / 2]} s={[0.1, h, d / 2 - 1.1]} c={wall} />
        <Box p={[L - 0.05, 0.475, 0.2]} s={[0.1, 0.95, 1.8]} c={wall} />
        <Box p={[L - 0.05, (2.35 + h) / 2, 0.2]} s={[0.1, h - 2.35, 1.8]} c={wall} />
        <mesh position={[L - 1.6, 1.6, 0.2]} rotation={[0, Math.PI / 2, 0]}><planeGeometry args={[5, 4]} /><meshBasicMaterial color={sky} /></mesh>
      </>) : <Box p={[L - 0.05, h / 2, 0]} s={[0.1, h, d]} c={wall} />}
      <Box p={[0, 0.45, B + 0.02]} s={[w, 0.9, 0.04]} c={wain} />
      <Box p={[-L - 0.02, 0.45, 0]} s={[0.04, 0.9, d]} c={wain} />
      <Box p={[0, 0.92, B + 0.05]} s={[w, 0.04, 0.05]} c="#e9e2cf" />
    </>);
  };

// ------------------------------------------------------------------ персонажи (любые картинки, см. char.ts)
type Key = [number, number, number];   // [секунда голоса, x, z] — точка на полу в метрах
const interp = (t: number, keys: Key[]): [number, number] => {
  if (t <= keys[0][0]) return [keys[0][1], keys[0][2]];
  for (let i = 1; i < keys.length; i++) if (t <= keys[i][0]) {
    const a = keys[i - 1], b = keys[i];
    const k = Easing.inOut(Easing.ease)((t - a[0]) / Math.max(1e-6, b[0] - a[0]));
    return [a[1] + (b[1] - a[1]) * k, a[2] + (b[2] - a[2]) * k];
  }
  const l = keys[keys.length - 1]; return [l[1], l[2]];
};
// Значение ступенькой: последнее с секундой ≤ t, и когда оно началось.
const stepAt = <T,>(t: number, keys: [number, T][] | undefined, dflt: T): [T, number] => {
  let v = dflt, tc = -10;
  for (const [tt, x] of keys ?? []) if (t >= tt) { v = x; tc = tt; }
  return [v, tc];
};
export type Actor = {
  char: string; pos: V3; seed: number; h?: number;
  expr: [number, string][];                 // [[секунда, эмоция], ...] — бонк на каждой смене
  talk?: [number, number][];                // [[с, по]] — реплики
  walk?: Key[];                             // [[секунда, x, z]] — ходьба вразвалку
  hop?: number[];                           // секунды прыжков
  lean?: [number, number][];                // [[секунда, угол рад]], + влево
  face?: [number, 1 | -1][];                // [[секунда, 1 как нарисован | −1 зеркально]]
  fx?: [number, string, number?][];         // [[секунда, эффект, длительность=1.2]]
  hidden?: [number, number][];
};
// Прыжок: присед 0.1 с → полёт 0.45 с → пружинящее приземление. → [подъём в долях роста, сжатие]
const hopAt = (t: number, hops: number[] = []): [number, number] => {
  let y = 0, sq = 1;
  for (const th of hops) {
    const d = t - th;
    if (d >= -0.1 && d < 0) sq = 1 - 0.12 * (1 + d / 0.1);
    else if (d >= 0 && d < 0.45) { const k = d / 0.45; y = 4 * k * (1 - k) * 0.3; sq = 1.08; }
    else if (d >= 0.45 && d < 0.75) sq = 1 - 0.14 * Math.exp(-(d - 0.45) * 14) * Math.cos((d - 0.45) * 28);
  }
  return [y, sq];
};

export const Character: React.FC<{ a: Actor }> = ({ a }) => {
  const sec = useSec();
  const { cam } = useContext(ShotCtx);
  const set = useChar(a.char);
  const { canvas, tex } = useMemo(() => {
    const cv = document.createElement("canvas");
    const t = new THREE.CanvasTexture(cv); t.colorSpace = THREE.SRGBColorSpace; t.anisotropy = 8; return { canvas: cv, tex: t };
  }, []);
  if (set === "loading") return null;
  const f2 = Math.floor(sec * FPS / 2) * 2;                              // по двойкам — «рисованная» анимация
  const t = f2 / FPS;
  if (a.hidden?.some(([s, e]) => t >= s && t < e)) return null;
  // размеры: персонаж W×H м, холст с запасом на сжатие/наклон/значки
  const H = a.h ?? set?.def.h ?? 1.2;
  const W = set ? H * (set.box[2] - set.box[0]) / (set.box[3] - set.box[1]) : H * 0.62;
  const PW = W * 1.8, PH = H * 1.55, K = 640 / PW;
  if (canvas.width !== 640) { canvas.width = 640; canvas.height = Math.round(PH * K); }
  // эмоция и бонк на смене
  const [ex, tc] = stepAt(t, a.expr, a.expr[0]?.[1] ?? "neutral");
  const tb = (t - tc) * FPS;
  const bonk = tc > -10 && tb >= 0 ? Math.exp(-tb / 3.5) * Math.cos(tb * 1.25) : 0;
  let sd = a.seed * 9301 + f2 * 49297; const rnd = () => ((sd = (sd * 233280 + 49297) % 233280) / 233280);
  // ходьба
  let pos = a.pos, moving = 0;
  if (a.walk) {
    const [x, z] = interp(t, a.walk), [px, pz] = interp(t - 0.1, a.walk);
    moving = Math.min(1, Math.hypot(x - px, z - pz) / 0.06); pos = [x, a.pos[1], z];
  }
  const ph = t * 8;
  const [hopY, hopSq] = hopAt(t, a.hop);
  const talking = a.talk?.some(([s, e]) => t >= s && t < e);
  const talkAmt = talking ? Math.abs(Math.sin(t * FPS * Math.PI / 3.5)) : 0;
  const fx: [string, number][] = (FX_OF[ex] ?? []).map((n) => [n, t - Math.max(tc, 0)]);
  for (const [s, n, d = 1.2] of a.fx ?? []) if (t >= s && t < s + d) fx.push([n, t - s]);
  const shakeOn = SHAKY.includes(ex);
  const pose: CharPose = {
    expr: ex, blink: (f2 + a.seed * 37) % 79 < 3, talk: talkAmt, seed: a.seed,
    squash: (1 + Math.sin(f2 / 14) * 0.012) * (1 - 0.08 * bonk) * hopSq * (1 + 0.035 * talkAmt) * (1 - moving * 0.04 * Math.abs(Math.cos(ph))),
    tilt: Math.sin(t * 1.1 + a.seed) * 0.025 + 0.05 * bonk + moving * Math.sin(ph) * 0.08 + (a.lean ? interp(t, a.lean.map(([s, v]) => [s, v, 0] as Key))[0] : 0),
    flip: stepAt(t, a.face, 1 as 1 | -1)[0], fx,
    shake: shakeOn ? [(rnd() - 0.5) * 0.014 * H, (rnd() - 0.5) * 0.008 * H] : [0, 0],
  };
  drawChar(canvas.getContext("2d")!, set, pose, W, H, K);
  tex.needsUpdate = true;
  const bob = moving * Math.abs(Math.sin(ph)) * 0.04 * H;
  const yaw = Math.atan2(cam.x - pos[0], cam.z - pos[2]);
  return (
    <mesh position={[pos[0], pos[1] + (hopY * H) + bob + PH / 2 - FOOT, pos[2]]} rotation={[0, yaw, 0]} receiveShadow>
      <planeGeometry args={[PW, PH]} />
      <meshLambertMaterial map={tex} emissive="#ffffff" emissiveMap={tex} emissiveIntensity={0.5} transparent alphaTest={0.15} side={THREE.DoubleSide} />
    </mesh>
  );
};

// ------------------------------------------------------------------ камера и сцена
export type CamKey = [number, V3, V3, number?];   // [секунда, позиция, куда смотрит, fov]
const lerp3 = (a: V3, b: V3, k: number): V3 => [a[0] + (b[0] - a[0]) * k, a[1] + (b[1] - a[1]) * k, a[2] + (b[2] - a[2]) * k];
const camAt = (t: number, keys: CamKey[]) => {
  if (t <= keys[0][0]) return keys[0];
  for (let i = 1; i < keys.length; i++) if (t <= keys[i][0]) {
    const a = keys[i - 1], b = keys[i], k = Easing.inOut(Easing.ease)((t - a[0]) / (b[0] - a[0]));
    return [t, lerp3(a[1], b[1], k), lerp3(a[2], b[2], k), (a[3] ?? 74) + ((b[3] ?? 74) - (a[3] ?? 74)) * k] as CamKey;
  }
  return keys[keys.length - 1];
};
const CamRig: React.FC<{ pos: THREE.Vector3; look: THREE.Vector3; fov: number }> = ({ pos, look, fov }) => {
  const { camera } = useThree();
  const c = camera as THREE.PerspectiveCamera;
  c.position.copy(pos); c.lookAt(look); c.fov = fov; c.near = 0.05; c.far = 25;   // far=25 — иначе размытие не работает
  c.updateProjectionMatrix();
  return null;
};

export type LightPreset = "day" | "night" | "studio";
const Lights: React.FC<{ preset: LightPreset; windowX: number }> = ({ preset, windowX }) => (
  preset === "day" ? (<>
    <hemisphereLight args={["#d6e6ff", "#3a3a5a", 0.9]} />
    <directionalLight position={[windowX - 3, 4.5, 1.5]} intensity={2.6} color="#ffd9a8" castShadow shadow-mapSize-width={2048} shadow-mapSize-height={2048}
      shadow-radius={8} shadow-bias={-0.0005} shadow-camera-left={-5} shadow-camera-right={5} shadow-camera-top={5} shadow-camera-bottom={-5} />
    <directionalLight position={[3, 2.5, 5]} intensity={0.7} color="#cfe0ff" />
    <pointLight position={[0.4, 2.75, -0.3]} intensity={5} decay={1.4} color="#fff0d8" />
  </>) : preset === "night" ? (<>
    <hemisphereLight args={["#5a6fb0", "#0c1430", 0.5]} />
    <directionalLight position={[windowX - 3, 4, 1]} intensity={1.4} color="#7d95ff" castShadow shadow-mapSize-width={2048} shadow-mapSize-height={2048} shadow-radius={8}
      shadow-camera-left={-5} shadow-camera-right={5} shadow-camera-top={5} shadow-camera-bottom={-5} />
    <pointLight position={[0.5, 1.6, 0.4]} intensity={7} decay={1.3} color="#ffc477" castShadow />
  </>) : (<>
    <hemisphereLight args={["#cfd8ff", "#141a33", 0.5]} />
    <spotLight position={[-1, 6, 2]} angle={0.5} penumbra={0.6} intensity={60} decay={1} color="#ff8a7a" castShadow />
    <spotLight position={[1.5, 6, 2]} angle={0.5} penumbra={0.6} intensity={60} decay={1} color="#7af0ff" castShadow />
  </>)
);

// Сцена-шот: камера по ключам (в секундах голоса), свет-пресет, постобработка (размытие фона, свечение, виньетка).
export const Scene3D: React.FC<{ t0: number; cam: CamKey[]; focus: V3; light?: LightPreset; bg?: string; windowX?: number; children: React.ReactNode }> =
  ({ t0, cam, focus, light = "day", bg = "#9fbfe0", windowX = -3, children }) => {
    const frame = useCurrentFrame();
    const { width, height } = useVideoConfig();
    const t = t0 + frame / FPS;
    const [, p, l, fov] = camAt(t, cam);
    const pos = new THREE.Vector3(...p);
    pos.x += Math.sin(frame / 23) * 0.012; pos.y += Math.sin(frame / 17) * 0.008;   // «ручная» камера
    return (
      <ShotCtx.Provider value={{ t0, cam: pos }}>
        <ThreeCanvas width={width} height={height} shadows={{ type: THREE.PCFSoftShadowMap }}
          gl={{ toneMapping: THREE.AgXToneMapping, toneMappingExposure: 1.25, antialias: false }} style={{ backgroundColor: bg }}>
          <ShotCtx.Provider value={{ t0, cam: pos }}>
            <CamRig pos={pos} look={new THREE.Vector3(...l)} fov={fov ?? 74} />
            <Lights preset={light} windowX={windowX} />
            {children}
            <EffectComposer multisampling={4}>
              <DepthOfField target={focus} focalLength={0.012} bokehScale={7} height={1080} />
              <Bloom luminanceThreshold={1.0} intensity={0.35} mipmapBlur />
              <Vignette eskil={false} offset={0.25} darkness={0.45} />
            </EffectComposer>
          </ShotCtx.Provider>
        </ThreeCanvas>
      </ShotCtx.Provider>
    );
  };

// «Бонк» предмета на ударном слове: масштаб 1→1.35→0.85→1.
export const bonkAt = (sec: number, t: number, amt = 0.35) => {
  const tb = (sec - t) * FPS;
  return tb > 0 ? 1 + amt * Math.exp(-tb / 3.5) * Math.cos(tb * 1.25) : 1;
};
```

### `src/Video.tsx`
```tsx
// РОЛИК: шоты по таймингам голоса + голос + музыка + субтитры. Всё время — в секундах ГОЛОСА.
// Для нового ролика меняется только массив SHOTS (и сами шоты ниже).
import React from "react";
import { AbsoluteFill, Audio, Sequence, interpolate, spring, staticFile, useCurrentFrame, useVideoConfig } from "remotion";
import { loadFont } from "@remotion/google-fonts/Montserrat";
import plan from "./plan.json";
import { FPS, Scene3D, Room, Box, Sign, Clock, Mirror, Bench, Plant, Lamp, Desk, Monitor, Character, Actor, bonkAt, useSec } from "./kit";

const { fontFamily } = loadFont("normal", { weights: ["900"], subsets: ["latin", "cyrillic"] });

// ===================================================================== ШОТЫ (пример: «Зеркало в лифте»)
// Шот 1 (0–5.1 с): холл, герои довольны у лифта. Шот 2 (5.1–11.4 с): герои злые, один «Опять 40 секунд!» у часов.
// id персонажей hero/friend/boss — замени на свои из src/chars.json (нет картинок → серые заглушки).
const Elevator: React.FC<{ floor: string }> = ({ floor }) => {
  const sec = useSec();
  return (<>
    <Box p={[0.4, 1.15, -1.97]} s={[1.5, 2.3, 0.08]} c="#5d6670" r={0.4} />
    <Box p={[0.08, 1.05, -1.91]} s={[0.62, 2.1, 0.04]} c="#b9c0c8" r={0.3} metal={0.5} />
    <Box p={[0.72, 1.05, -1.91]} s={[0.62, 2.1, 0.04]} c="#b9c0c8" r={0.3} metal={0.5} />
    <group position={[0.4, 2.45, -1.92]} scale={bonkAt(sec, 1.7)}><Sign p={[0, 0, 0]} w={0.5} h={0.22} text={sec < 1.7 ? floor : "6"} bg="#15151a" fg="#ff3b2f" glow={1.2} /></group>
    <Sign p={[-1.0, 2.45, -1.99]} w={0.7} h={0.24} text="ЛИФТ" bg="#20354f" fg="#ffffff" />
  </>);
};

const Lobby: React.FC<{ shot: 1 | 2 }> = ({ shot }) => {
  const sec = useSec();
  return (<>
    <Room wall="#c96f4a" wain="#8a4630" />
    <Elevator floor="7" />
    <Clock p={[1.6, 2.35, -1.99]} hh={8} mm={59} scale={bonkAt(sec, 9.95, 0.3)} />
    <Mirror p={[-0.9, 1.35, -1.95]} scale={shot === 1 ? 1 : 0.0001} />
    <Bench p={[-1.75, 0, -1.55]} /><Plant p={[-2.65, 0, -1.5]} /><Plant p={[2.6, 0, -1.5]} h={0.8} />
    <Sign p={[2.3, 1.6, -1.99]} w={0.5} h={0.7} text="Q3" bg="#2fe0c8" fg="#f6f3ea" glow={0.15} />
    <Lamp p={[0.4, 2.97, -0.3]} />
  </>);
};

const C1: Actor = { char: "hero", pos: [-0.25, 0, -0.95], seed: 1,
  expr: [[0, "happy"], [0.9, "proud"], [3.0, "smug"], [5.1, "stern"], [9.9, "angry"], [11.0, "stern"]],
  hop: [9.75], talk: [[9.9, 11.0]] };                    // крик у часов: прыжок + злость на ударном слове
const C2: Actor = { char: "friend", pos: [0.5, 0, -0.8], seed: 2, face: [[0, -1]],
  expr: [[0, "smug"], [1.3, "happy"], [3.0, "laugh"], [4.0, "proud"], [5.1, "nervous"], [10.0, "shock"]] };
const C3: Actor = { char: "boss", pos: [1.2, 0, -1.05], seed: 3,
  expr: [[0, "proud"], [1.7, "realize"], [2.5, "happy"], [3.5, "smug"], [5.1, "stern"], [7.9, "nervous"]],
  hop: [3.5], lean: [[5.1, 0], [7.9, 0], [8.3, 0.12]] };   // радуется прыжком; нервно отклоняется

const SHOTS: { t0: number; t1: number; el: React.ReactNode }[] = [
  { t0: 0.0, t1: 5.1, el: (
    <Scene3D t0={0} cam={[[0, [1.85, 1.32, 1.85], [0.1, 1.12, -1.2], 76], [5.1, [1.5, 1.28, 1.55], [0.1, 1.12, -1.2], 76]]} focus={[0.45, 1.25, -0.9]}>
      <Lobby shot={1} />{[C1, C2, C3].map((a, i) => <Character key={i} a={a} />)}
    </Scene3D>) },
  { t0: 5.1, t1: 11.4, el: (
    <Scene3D t0={5.1} cam={[[5.1, [-1.5, 1.25, 1.8], [0.5, 1.2, -1.4], 76], [11.4, [-1.2, 1.25, 1.6], [0.5, 1.25, -1.4], 76]]} focus={[0.4, 1.25, -0.9]}>
      <Lobby shot={2} />{[C1, C2, C3].map((a, i) => <Character key={i} a={a} />)}
    </Scene3D>) },
];

// ===================================================================== субтитры (как в эталоне)
const Caption: React.FC = () => {
  const f = useCurrentFrame();
  const { fps } = useVideoConfig();
  const g = plan.groups.find((x) => f >= x.start && f < Math.max(x.end, x.start + 6));
  if (!g) return null;
  const s = spring({ frame: f - g.start, fps, config: { damping: 11, stiffness: 240, mass: 0.55 } });
  return (
    <AbsoluteFill style={{ justifyContent: "flex-end", alignItems: "center", paddingBottom: 600 }}>
      <div style={{ fontFamily, fontWeight: 900, fontSize: 92, lineHeight: 1.05, color: "white", textAlign: "center", maxWidth: 920,
        transform: `scale(${interpolate(s, [0, 1], [0.72, 1])})`, WebkitTextStroke: "16px black", paintOrder: "stroke fill",
        textShadow: "0 6px 0 rgba(0,0,0,0.55)" }}>{g.text}</div>
    </AbsoluteFill>
  );
};

export const TOTAL = Math.round(SHOTS[SHOTS.length - 1].t1 * FPS);
export const Video: React.FC = () => {
  const { durationInFrames } = useVideoConfig();
  return (
    <AbsoluteFill style={{ backgroundColor: "black" }}>
      {SHOTS.map((s, i) => (
        <Sequence key={i} from={Math.round(s.t0 * FPS)} durationInFrames={Math.round((s.t1 - s.t0) * FPS)}>{s.el}</Sequence>
      ))}
      <Audio src={staticFile(plan.voice)} />
      <Audio src={staticFile("music.wav")} loop
        volume={(f) => interpolate(f, [0, 20, durationInFrames - 45, durationInFrames], [0, 0.11, 0.11, 0], { extrapolateLeft: "clamp", extrapolateRight: "clamp" })} />
      <Caption />
    </AbsoluteFill>
  );
};
```

### `tools/whisper.mjs`
```js
// Пословные тайминги голоса (локальный whisper.cpp, бесплатно, без интернета после первой загрузки ~470 МБ).
//   node tools/whisper.mjs public/voice.mp3 <язык: ru|en|...>  → audio/words.json  [{t, s, e}]
// Грабли: в Windows установщик Remotion ломается на пробелах в пути (Expand-Archive) — поэтому распаковываем сами.
import fs from "node:fs";
import path from "node:path";
import os from "node:os";
import { execFileSync } from "node:child_process";
import { installWhisperCpp, downloadWhisperModel, transcribe, toCaptions } from "@remotion/install-whisper-cpp";

const src = path.resolve(process.argv[2] ?? "public/voice.mp3");
const lang = process.argv[3] ?? "ru";
const WDIR = path.join(os.homedir(), ".doodle-shorts", "whisper");   // без пробелов, общий для всех проектов
const MODEL = "small";
fs.mkdirSync(path.dirname(WDIR), { recursive: true });   // саму WDIR не создавать: установщик Remotion требует, чтобы её не было
const wav = path.join(os.tmpdir(), "doodle_voice_16k.wav");
execFileSync("ffmpeg", ["-y", "-loglevel", "error", "-i", src, "-ar", "16000", "-ac", "1", wav]);
const exe = process.platform === "win32" ? "main.exe" : "main";
if (!fs.existsSync(path.join(WDIR, exe))) {
  try {
    await installWhisperCpp({ to: WDIR, version: "1.5.5" });
  } catch (e) {
    const zip = fs.readdirSync(process.cwd()).find((f) => f.startsWith("whisper-bin") && f.endsWith(".zip"));
    if (process.platform === "win32" && zip) {
      execFileSync("powershell", ["-NoProfile", "-Command", `Expand-Archive -Force -LiteralPath '${path.resolve(zip)}' -DestinationPath '${WDIR}'`]);
      fs.rmSync(zip);
    } else throw e;
  }
}
await downloadWhisperModel({ model: MODEL, folder: WDIR });
const { transcription } = await transcribe({ inputPath: wav, whisperPath: WDIR, model: MODEL, tokenLevelTimestamps: true, language: lang });
const { captions } = toCaptions({ whisperCppOutput: { transcription } });
// токены → слова (новое слово начинается с пробела)
const words = [];
for (const c of captions) {
  const s = c.text;
  if (!s.trim()) continue;
  if (/^[\s]*[.,!?:;%—-]+$/.test(s) && words.length) { words[words.length - 1].t += s.trim(); continue; }
  if (/^\s/.test(s) || !words.length) words.push({ t: s.trim(), s: +(c.startMs / 1000).toFixed(2), e: +(c.endMs / 1000).toFixed(2) });
  else { words[words.length - 1].t += s; words[words.length - 1].e = +(c.endMs / 1000).toFixed(2); }
}
fs.mkdirSync(path.resolve("audio"), { recursive: true });
fs.writeFileSync(path.resolve("audio/words.json"), JSON.stringify(words, null, 1));
console.log(words.length, "слов →", path.resolve("audio/words.json"));
console.log(words.map((w) => w.s.toFixed(1) + " " + w.t).join(" | "));
```

### `tools/plan.mjs`
```js
// Монтажный план: субтитры (слова СЦЕНАРИЯ ← время из whisper через выравнивание; ошибки распознавания
// в субтитры не попадают) + громкости. Звуковых эффектов нет — только голос и музыка.
//   node tools/plan.mjs   (читает script/voice.txt, audio/words.json, script/edit.json)
import fs from "node:fs";

const FPS = 30;
const P = (f) => JSON.parse(fs.readFileSync(f, "utf8"));
const raw = fs.readFileSync("script/voice.txt", "utf8");
const words = P("audio/words.json").filter((w) => w.t.replace(/[^\p{L}\p{N}]/gu, ""));
const cfg = fs.existsSync("script/edit.json") ? P("script/edit.json") : {};
const norm = (w) => w.toLowerCase().replace(/ё/g, "е").replace(/[^\p{L}\p{N}]/gu, "");
// текст для субтитров: без тегов эмоций [..], без кавычек; числа можно писать цифрами в voice_display.txt
const disp = fs.existsSync("script/voice_display.txt") ? fs.readFileSync("script/voice_display.txt", "utf8") : raw;
const paras = disp.replace(/\[[^\]]+\]\s*/g, "").replace(/[«»"]/g, "").split(/\n\s*\n/).map((p) => p.trim()).filter(Boolean);
const mine = [];
paras.forEach((p, pi) => p.replace(/\s[—–-]\s/g, " ").split(/\s+/).filter(Boolean).forEach((w) => mine.push({ text: w, para: pi })));

const sim = (a, b) => {
  a = norm(a); b = norm(b);
  if (!a || !b) return 0;
  if (a === b) return 1;
  let k = 0; while (k < Math.min(a.length, b.length) && a[k] === b[k]) k++;
  return k >= 3 || (k >= 2 && Math.min(a.length, b.length) <= 3) ? 0.7 : 0;
};
const n = mine.length, m = words.length;
const D = Array.from({ length: n + 1 }, () => new Float64Array(m + 1));
const B = Array.from({ length: n + 1 }, () => new Int8Array(m + 1));
for (let i = 1; i <= n; i++) { D[i][0] = -0.4 * i; B[i][0] = 1; }
for (let j = 1; j <= m; j++) { D[0][j] = -0.4 * j; B[0][j] = 2; }
for (let i = 1; i <= n; i++) for (let j = 1; j <= m; j++) {
  const s = sim(mine[i - 1].text, words[j - 1].t);
  const c = [D[i - 1][j - 1] + (s > 0 ? s : -0.6), D[i - 1][j] - 0.4, D[i][j - 1] - 0.4];
  const k = c.indexOf(Math.max(...c)); D[i][j] = c[k]; B[i][j] = k;
}
const at = new Array(n).fill(null);
for (let i = n, j = m; i > 0 || j > 0;) {
  const k = B[i][j];
  if (i > 0 && j > 0 && k === 0) { if (sim(mine[i - 1].text, words[j - 1].t) > 0) at[i - 1] = words[j - 1].s; i--; j--; }
  else if (i > 0 && (j === 0 || k === 1)) i--; else j--;
}
const endT = words.length ? words[words.length - 1].e ?? words[words.length - 1].s + 0.6 : 10;
for (let i = 0; i < n; i++) if (at[i] == null) {
  let a = i - 1; while (a >= 0 && at[a] == null) a--;
  let b = i + 1; while (b < n && at[b] == null) b++;
  const ta = a >= 0 ? at[a] : 0, tb = b < n ? at[b] : endT;
  at[i] = ta + ((tb - ta) * (i - a)) / (b - a);
}
const W = mine.map((w, i) => ({ text: w.text, start: Math.round(at[i] * FPS), end: Math.round((i + 1 < n ? at[i + 1] : at[i] + 0.6) * FPS) }));
const groups = [];
let cur = [];
W.forEach((w, k) => {
  cur.push(w);
  const len = cur.reduce((s, x) => s + x.text.length, 0);
  if (/[.,:!?…]$/.test(w.text) || cur.length >= 3 || len > 13 || k === W.length - 1 || mine[k + 1]?.para !== mine[k].para) {
    groups.push({ text: cur.map((x) => x.text).join(" "), start: cur[0].start, end: Math.min(cur[cur.length - 1].end, cur[0].start + 75), words: cur });
    cur = [];
  }
});
const total = Math.round((endT + (cfg.tail ?? 0.7)) * FPS);
const plan = {
  fps: FPS, total,
  video: cfg.video ?? "video.mp4", voice: cfg.voice ?? "voice.mp3", music: cfg.music === false ? null : "music.wav",
  mix: { voice: 1.0, music: cfg.music_volume ?? 0.11 },   // голос ВСЕГДА громче музыки
  caption: { size: 92, stroke: 16, bottom: 600, highlight: cfg.highlight ?? null },
  groups,
};
fs.writeFileSync("src/plan.json", JSON.stringify(plan, null, 1));
console.log(`план: ${(total / FPS).toFixed(1)} с, ${groups.length} групп субтитров`);
console.log(groups.slice(0, 6).map((g) => `${(g.start / FPS).toFixed(1)} «${g.text}»`).join(" | "));
```

### `tools/music.mjs`
```js
// Процедурная фоновая музыка (без внешних файлов и лицензий — можно монетизировать). Звуковых эффектов нет.
//   node tools/music.mjs <durationSec> [playful|tense|calm]   → public/music.wav
import fs from "node:fs";
import path from "node:path";

const SR = 44100;
const midi = (m) => 440 * Math.pow(2, (m - 69) / 12);
const MOODS = {
  playful: { bpm: 118, root: 60, prog: [[0, "maj"], [-3, "min"], [-7, "maj"], [-5, "maj"]], pad: 0.18, pluck: 0.45, bass: 0.3, kick: 0.35, hat: 0.25, bright: 0.8 },
  tense: { bpm: 96, root: 50, prog: [[0, "min"], [-2, "maj"], [-4, "maj"], [-5, "maj"]], pad: 0.3, pluck: 0.3, bass: 0.4, kick: 0.4, hat: 0.2, bright: 0.45 },
  calm: { bpm: 84, root: 55, prog: [[0, "maj"], [-3, "min"], [-5, "maj"], [-7, "maj"]], pad: 0.3, pluck: 0.4, bass: 0.18, kick: 0, hat: 0, bright: 0.7 },
};
const chord = (root, type) => (type === "min" ? [0, 3, 7, 10] : [0, 4, 7, 11]).map((x) => root + x);

export const writeWav = (file, floatChannels) => {
  const len = floatChannels[0].length, ch = floatChannels.length;
  const buf = Buffer.alloc(44 + len * ch * 2);
  buf.write("RIFF", 0); buf.writeUInt32LE(36 + len * ch * 2, 4); buf.write("WAVE", 8);
  buf.write("fmt ", 12); buf.writeUInt32LE(16, 16); buf.writeUInt16LE(1, 20); buf.writeUInt16LE(ch, 22);
  buf.writeUInt32LE(SR, 24); buf.writeUInt32LE(SR * ch * 2, 28); buf.writeUInt16LE(ch * 2, 32); buf.writeUInt16LE(16, 34);
  buf.write("data", 36); buf.writeUInt32LE(len * ch * 2, 40);
  let o = 44;
  for (let i = 0; i < len; i++) for (let c = 0; c < ch; c++) { buf.writeInt16LE(Math.round(Math.max(-1, Math.min(1, floatChannels[c][i])) * 32767), o); o += 2; }
  fs.writeFileSync(file, buf);
};

const music = (mood, dur) => {
  const m = MOODS[mood] ?? MOODS.playful;
  const n = Math.floor(dur * SR), L = new Float32Array(n), R = new Float32Array(n);
  const beat = 60 / m.bpm, bar = beat * 4;
  let seed = 12345;
  const rnd = () => { seed = (seed * 1664525 + 1013904223) >>> 0; return seed / 4294967296; };
  let lpL = 0, lpR = 0;
  const alpha = 0.02 + m.bright * 0.08;
  for (let i = 0; i < n; i++) {
    const t = i / SR;
    const [off, type] = m.prog[Math.floor(t / bar) % m.prog.length];
    const notes = chord(m.root + off, type);
    let s = 0;
    if (m.pad) { let p = 0; for (const nn of notes) for (const det of [-0.4, 0, 0.4]) { const ph = (t * (midi(nn) + det)) % 1; p += (2 * ph - 1) * 0.08; } s += p * m.pad * (0.8 + 0.2 * Math.sin(t * 0.7)); }
    if (m.bass) { const f = midi(m.root + off - 12); const g = Math.pow(0.5, (t % beat) / 0.35); s += (Math.sin(2 * Math.PI * f * t) * 0.8 + Math.sin(4 * Math.PI * f * t) * 0.15) * m.bass * (0.6 + 0.4 * g); }
    if (m.pluck) { const st = beat / 2, k = Math.floor(t / st), tt = t - k * st; const f = midi(notes[[0, 2, 1, 3, 2, 0, 3, 1][k % 8]] + 12); s += (2 * Math.abs(2 * ((t * f) % 1) - 1) - 1) * Math.exp(-tt * (6 + m.bright * 6)) * m.pluck * 0.5; }
    if (m.kick) { const tt = t % beat; s += Math.sin(2 * Math.PI * (48 * Math.exp(-tt * 30) + 48) * tt) * Math.exp(-tt * 12) * m.kick * 0.9; }
    if (m.hat) { const st = beat / 2, tt = t % st; if (Math.floor(t / st) % 2 === 1) s += (rnd() * 2 - 1) * Math.exp(-tt * 60) * m.hat * 0.5; }
    lpL += alpha * (s - lpL); lpR += alpha * 1.05 * (s - lpR);
    const fade = Math.min(1, t / 1.5, (dur - t) / 2.5);
    L[i] = lpL * fade; R[i] = lpR * fade;
  }
  const d1 = Math.floor(0.231 * SR), d2 = Math.floor(0.347 * SR);
  for (let i = 0; i < n; i++) { if (i >= d1) L[i] += R[i - d1] * 0.28; if (i >= d2) R[i] += L[i - d2] * 0.28; }
  let peak = 0; for (let i = 0; i < n; i++) peak = Math.max(peak, Math.abs(L[i]), Math.abs(R[i]));
  const g = 0.45 / (peak || 1); for (let i = 0; i < n; i++) { L[i] *= g; R[i] *= g; }
  return [L, R];
};

const [, , a, b] = process.argv;
fs.mkdirSync("public", { recursive: true });
writeWav("public/music.wav", music(b ?? "playful", Number(a ?? 65)));
console.log("music -> public/music.wav");
```

### `tools/tts_piper.mjs`
```js
// БЕСПЛАТНАЯ озвучка без регистрации: Piper (sherpa-onnx), работает офлайн на Windows/Mac.
// Звучит заметно «роботизированнее», чем ElevenLabs (честно), но разборчиво — годится для черновиков и
// роликов без монетизации. Голоса: ru_RU-dmitri-medium (муж.), ru_RU-irina-medium (жен.), en_US-ryan-high …
//   node tools/tts_piper.mjs script/voice.txt public/voice.wav [voice] [speed]
import fs from "node:fs";
import path from "node:path";
import os from "node:os";
import { execFileSync } from "node:child_process";
import { createRequire } from "node:module";

const require = createRequire(import.meta.url);
const [, , txtFile, out = "public/voice.wav", voice = "ru_RU-dmitri-medium", speed = "1.08"] = process.argv;
const DIR = path.join(os.homedir(), ".doodle-shorts", "tts");
const vdir = path.join(DIR, `vits-piper-${voice}`);
if (!fs.existsSync(path.join(vdir, "tokens.txt"))) {
  fs.mkdirSync(DIR, { recursive: true });
  const name = `vits-piper-${voice}.tar.bz2`;
  console.log("↓ голос", voice, "(~70 МБ, один раз)");
  execFileSync("curl", ["-sL", "-f", "-o", name, `https://github.com/k2-fsa/sherpa-onnx/releases/download/tts-models/${name}`], { cwd: DIR, stdio: "inherit" });
  const winTar = path.join(process.env.SystemRoot ?? "C:\\Windows", "System32", "tar.exe");
  execFileSync(process.platform === "win32" && fs.existsSync(winTar) ? winTar : "tar", ["xjf", name], { cwd: DIR, stdio: "inherit" });
  fs.rmSync(path.join(DIR, name), { force: true });
}
const sherpa = require("sherpa-onnx-node");
const onnx = fs.readdirSync(vdir).find((f) => f.endsWith(".onnx"));
const tts = new sherpa.OfflineTts({
  model: { vits: { model: path.join(vdir, onnx), tokens: path.join(vdir, "tokens.txt"), dataDir: path.join(vdir, "espeak-ng-data"),
    noiseScale: 0.6, noiseScaleW: 0.7, lengthScale: 1.0 }, numThreads: 2, provider: "cpu" }, maxNumSentences: 1 });
// абзацы — с паузой 0.45 с между ними
const paras = fs.readFileSync(txtFile, "utf8").replace(/\[[^\]]+\]\s*/g, "").split(/\n\s*\n/).map((p) => p.trim()).filter(Boolean);
const chunks = [];
let sr = 22050;
for (const p of paras) {
  const a = tts.generate({ text: p, sid: 0, speed: Number(speed) });
  sr = a.sampleRate;
  chunks.push(Float32Array.from(a.samples), new Float32Array(Math.floor(sr * 0.45)));
}
const total = chunks.reduce((s, c) => s + c.length, 0);
const all = new Float32Array(total);
let o = 0; for (const c of chunks) { all.set(c, o); o += c.length; }
fs.mkdirSync(path.dirname(path.resolve(out)), { recursive: true });
const tmp = out.replace(/\.wav$/, "_raw.wav");
// сохраняем с реальной частотой модели, затем ffmpeg приводит к 44.1 кГц
const buf = Buffer.alloc(44 + total * 2);
buf.write("RIFF", 0); buf.writeUInt32LE(36 + total * 2, 4); buf.write("WAVE", 8); buf.write("fmt ", 12); buf.writeUInt32LE(16, 16);
buf.writeUInt16LE(1, 20); buf.writeUInt16LE(1, 22); buf.writeUInt32LE(sr, 24); buf.writeUInt32LE(sr * 2, 28); buf.writeUInt16LE(2, 32);
buf.writeUInt16LE(16, 34); buf.write("data", 36); buf.writeUInt32LE(total * 2, 40);
for (let i = 0; i < total; i++) buf.writeInt16LE(Math.round(Math.max(-1, Math.min(1, all[i])) * 32767), 44 + i * 2);
fs.writeFileSync(tmp, buf);
execFileSync("ffmpeg", ["-y", "-loglevel", "error", "-i", tmp, "-ar", "44100", out]);
fs.rmSync(tmp);
console.log("голос →", out, (total / sr).toFixed(1), "с");
```

### `tools/char.mjs`
```js
// Персонажи: кладёт картинку в public/chars/<id>/<эмоция>.png и пересобирает src/chars.json.
//   node tools/char.mjs add <id> <картинка.png|jpg|webp> [эмоция=neutral] [--h 1.3] [--bust] [--key-white]
//   node tools/char.mjs scan          — пересобрать src/chars.json по папкам public/chars/*
// Повторный запуск с теми же аргументами даёт тот же результат.
import fs from "node:fs";
import path from "node:path";
import { execFileSync } from "node:child_process";

const ROOT = "public/chars", DB = "src/chars.json", NAME = /^[a-z0-9_-]+$/i;
const die = (m) => { console.error(m); process.exit(1); };
const USAGE = "node tools/char.mjs add <id> <картинка> [эмоция] [--h 1.3] [--bust] [--key-white]  |  node tools/char.mjs scan";

const opts = {}, pos = [];
const argv = process.argv.slice(2);
for (let i = 0; i < argv.length; i++) {
  const a = argv[i];
  if (a === "--h") { opts.h = Number(argv[++i]); if (!(opts.h > 0)) die("--h: рост в метрах, напр. --h 1.3"); }
  else if (a === "--bust") opts.bust = true;
  else if (a === "--key-white") opts.key = true;
  else pos.push(a);
}
const [cmd, id, src, emo = "neutral"] = pos;
const db = fs.existsSync(DB) ? JSON.parse(fs.readFileSync(DB, "utf8")) : {};

if (cmd === "add") {
  if (!id || !src) die(USAGE);
  if (!NAME.test(id) || !NAME.test(emo)) die("id и эмоция — латиница/цифры/_/-");
  if (!fs.existsSync(src)) die("нет файла: " + src);
  const dir = path.join(ROOT, id);
  fs.mkdirSync(dir, { recursive: true });
  const dst = path.join(dir, emo + ".png");
  if (!opts.key && /\.png$/i.test(src)) fs.copyFileSync(src, dst);          // PNG как есть — ffmpeg не нужен
  else {
    const vf = opts.key ? "colorkey=0xFFFFFF:0.10:0.05,format=rgba" : "format=rgba";
    try { execFileSync("ffmpeg", ["-y", "-loglevel", "error", "-i", src, "-vf", vf, "-frames:v", "1", dst]); }
    catch { die("нужен ffmpeg (Шаг 0): он делает --key-white и переводит JPG/WEBP в PNG"); }
  }
  db[id] = { ...db[id], ...(opts.h ? { h: opts.h } : {}), ...(opts.bust ? { bust: true } : {}) };
} else if (cmd !== "scan") die(USAGE);

const out = {};
for (const cid of fs.existsSync(ROOT) ? fs.readdirSync(ROOT).sort() : []) {
  const dir = path.join(ROOT, cid);
  if (!NAME.test(cid) || !fs.statSync(dir).isDirectory()) continue;
  const pngs = fs.readdirSync(dir).filter((f) => /\.png$/i.test(f)).sort();
  if (!pngs.length) continue;
  const files = Object.fromEntries(pngs.map((f) => [path.basename(f, path.extname(f)), f]));
  out[cid] = { h: db[cid]?.h ?? 1.2, bust: db[cid]?.bust ?? false, files };
  console.log(`${cid}: рост ${out[cid].h} м${out[cid].bust ? ", бюст" : ""}; эмоции: ${Object.keys(files).join(", ")}`);
  if (!files.neutral) console.warn(`  ! у ${cid} нет neutral.png — добавь, это базовая картинка`);
}
fs.writeFileSync(DB, JSON.stringify(out, null, 2) + "\n");
console.log("→ " + DB);
```

### `script/edit.json` (необязательно)
```json
{"voice": "voice.mp3", "music_volume": 0.11, "highlight": null}
```

Пустые папки, которые нужно создать: `public/`, `audio/`, `script/`, `checks/`, `out/`.


---

# Приложение А: эталонные сценарии (Ricks Clips) с разбором

**Как использовать.** Перед тем как писать сценарий для пользователя, прочитай ВСЕ пять разборов (при доступе в интернет — посмотри и сами ролики).
Пиши в той же манере и на том же уровне. **Копировать нельзя** — ни сюжеты, ни фразы. Повторять нужно
структуру и подачу: хук-парадокс, «ты» в истории, смешные конкретные детали, загадку, бытовую аналогию,
реальный факт и финал-«кнопку».

Ниже — краткие пересказы роликов канала Ricks Clips (оригиналы — на их YouTube) и разбор приёмов. Ролик делается на языке пользователя.

---

## 1. Why the Second Draw Always Wins

> Пересказ: самый быстрый стрелок Запада никогда не стреляет первым. «Ты» выхватываешь револьвер первым и падаешь в пыль, хотя Эдди даже не быстрый — он смотрит только на твою руку. Аналогия — ловишь падающий телефон, не решая. Механизм: решение тормозит руку, рефлекс — нет.

**Разбор.**
- **Хук (1-я фраза, ~2 с):** парадокс-правило: «самый быстрый стрелок никогда не стреляет первым». Противоречие заставляет досмотреть.
- **Сюжет:** «Представь…» → ты стреляешь первым → проигрываешь → «Как такое возможно?» → Эдди даже не быстрый (смешная деталь) → странная примета (смотрит на руку) → бытовая аналогия (телефон со стола) → научное объяснение (реакция быстрее решения) → итоговая формула.
- **Сцен:** ~8: улица в полдень, Эдди кивает, рывок к кобуре, ты в пыли, гонка с пивом, очередь дуэлянтов, взгляд на руку, телефон падает со стола.
- **Длина фраз:** 8–20 слов, после длинной обязательно короткая («How is that possible?», «Eddie isn't even fast.»).
- **Темп:** ~200 слов ≈ 60 с; каждые 1–2 фразы — новая картинка.
- **Финал:** не мораль, а чёткая формула механизма («ждёт, пока дёрнется твоя рука, и тело реагирует само»).
- **Почему работает:** зритель — участник («you»); загадка поставлена на 15-й секунде, ответ дан только в конце; аналогия из жизни зрителя делает науку очевидной.

---

## 2. Every Memory Has a Price

> Пересказ: в мире, где воспоминания — деньги, герой продаёт самое счастливое воспоминание в лавке на углу. Правила мира — через смешные бытовые сделки (аренда днями рождения и т.п.). Ранняя деталь (история про лягушку у озера) возвращается в финале как удар.

**Разбор.**
- **Хук:** действие + странность («продал самое счастливое воспоминание незнакомцу за прилавком»).
- **Сюжет:** правила мира в 3–4 смешно-конкретных примерах (аренда днями рождения, Багамы 300 раз с дивана) → герой и ставка (брат болен) → эскалация (продаёт всё больше) → «последний счастливый год» → жертва → твист: брат здоров, но Джерри его не помнит.
- **Сцен:** ~10 (лавка, город со стеной, богатый на диване, фото, больница, озеро с лягушкой, прилавок, кровать брата).
- **Длина фраз:** 6–25 слов; короткие удары в кульминации («He sells it anyway.»).
- **Темп:** ~300 слов ≈ 90 с — длинный формат; для 45–60 с сокращать мир до 2 примеров.
- **Финал:** закольцованная деталь (история про лягушку) превращается в удар — заранее посаженное «ружьё».
- **Почему работает:** мир объяснён через бытовые сделки; эмоция растёт ступенями; деталь из середины стреляет в финале.

---

## 3. He Could Spot Winning Tickets Just by Looking

> Пересказ: «посмотри на два лотерейных билета — один выигрышный, и это видно без стирания». Уволенный парень на последние деньги покупает билеты, раскладывает их по полу и находит закономерность в напечатанных цифрах. Лотерея не верит — он приносит коробку нестёртых билетов и угадывает почти все. Игру убирают из магазинов.

**Разбор.**
- **Хук:** обращение к зрителю + вызов («посмотри на эти два билета — один выигрышный, и это можно узнать не стирая»). Зритель сразу пытается угадать.
- **Сюжет:** завязка (уволили, 5 долларов) → подозрение-вопрос → иронично «разумный» поступок (спустил выходное пособие) → открытие (числа-одиночки) → неожиданный выбор (звонит в лотерею, а не грабит) → доказательство (90%) → последствие (игру убрали).
- **Сцен:** ~8.
- **Длина фраз:** средние, с ироничными вставками («like a normal person», «what any sane person would do»).
- **Темп:** ~230 слов ≈ 65 с.
- **Финал:** короткое масштабное последствие («к концу недели игры не было ни в одном магазине»).
- **Почему работает:** реальная история (Мохан Шривастава) подана как анекдот; ирония рассказчика + конкретные цифры.

---

## 4. The Ancient Jar That Warned an Empire

> Пересказ: изобретатель кладёт бронзовый шар в пасть дракона на сосуде и говорит императору «если выпадет — беги». Двор смеётся; ложная тревога, изгнание; потом шары падают один за другим. Механизм (древний сейсмоскоп) раскрыт в конце как награда.

**Разбор.**
- **Хук:** странный предмет + грозное предупреждение («если шар выпадет — беги»).
- **Сюжет:** смех двора → описание прибора (визуально) → ложная тревога → изгнание → ночь, падают все шары → никто не верит (шутка стражи) → катастрофа → объяснение механизма (маятник).
- **Сцен:** ~10 — каждая фраза = картинка.
- **Длина фраз:** нарастающее «стаккато» в кульминации: «Then one night a ball dropped. Then another. Then all eight at once.»
- **Темп:** ~250 слов; объяснение механизма — в самом конце, как награда.
- **Финал:** обрывается на интриге/продолжении («один шар значил толчок издалека…») — повод досмотреть/пересмотреть.
- **Почему работает:** реальный факт (сейсмоскоп Чжан Хэна) рассказан как притча «над ним смеялись — он оказался прав».

---

## 5. Levels of Pirate Cannonballs

> Пересказ: «если в твой корабль попало ядро — тебе повезло». Формат «уровни»: обычное ядро → пиратский мусор вместо ядер → раскалённое ядро → огненный снаряд → железные корабли и гигантская пушка с ядром за 450 кг. Каждая ступень — новая картинка-«вау».

**Разбор.**
- **Хук:** контринтуитивное утверждение («если в твой корабль попало ядро — тебе ПОВЕЗЛО»).
- **Сюжет:** формат «уровни» — каждая ступень хуже/круче предыдущей: ядро → мусор → калёное ядро → «каркас» с огнём → железные корабли → пушка Родмана. Противостояние «флот vs пираты» как пинг-понг.
- **Сцен:** по 1–2 на уровень (~8).
- **Длина фраз:** длинные описательные + короткие ироничные пояснения («a fancy word for literal garbage»).
- **Темп:** ~220 слов ≈ 60 с; каждые 7–9 секунд — новый уровень (новая картинка-«вау»).
- **Финал:** максимальный уровень (1000-фунтовое ядро) — гипербола как кульминация.
- **Почему работает:** простая лестница эскалации, каждая ступень визуальна; ирония в словах-пояснениях.

---

## Наш собственный пример (сделан по этой формуле, одобрен пользователем)

> Самого надёжного машиниста Токио считают психом: весь рейс он тычет пальцем в пустоту и разговаривает сам с собой.
>
> Представь: первый день, ты стажёр в его кабине. Светофор — палец. «Зелёный — есть!» Часы — палец. «Двадцать три сорок одна — есть!»
>
> Пустая платформа — «Чисто!» Даже на кота на перроне показал. «Кот — есть!»
>
> Ты еле сдерживаешься, ведь вокруг ни души. Но у этого чудика тридцать лет без единой ошибки.
>
> А ты уже на третий день пролетаешь станцию, хотя смотрел прямо на табличку. Как такое возможно?
>
> Вспомни: ты выходишь из дома и через минуту не помнишь, закрыл ли дверь. Закрыл. Просто мозг сделал это на автопилоте и ничего не записал.
>
> Глаза видят, а голова спит. Чтобы показать пальцем и сказать вслух, мозгу приходится проснуться и по-настоящему заметить.
>
> Японские инженеры проверили: такой спектакль срезает ошибки почти на восемьдесят пять процентов.
>
> Так что на следующий день ты уже стоишь в кабине, тычешь в кота и орёшь: «Кот — есть!»

Хук-парадокс (надёжный = псих) → «Представь, ты стажёр» → смешные повторы-ритм («палец… есть!») → посаженная деталь (кот) → вопрос «Как такое возможно?» → бытовая аналогия (закрыл ли дверь) → механизм → факт (−85%) → финал-кнопка с котом. 9 абзацев = 9 шотов, ~150 слов, ~60 с с голосом ElevenLabs.
