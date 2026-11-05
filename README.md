# shorts-remotion

Скилл для [Claude Code](https://claude.com/claude-code): делает готовый вертикальный шортс
(YouTube Shorts / Reels / TikTok, 40–65 с) по идее — **с любым персонажем**.

- Персонаж — твои картинки PNG (маскот, стикер, рисунок): одна `neutral.png` или набор эмоций.
  Код сам оживляет их: дыхание, моргание, «бонк» на смене эмоции, реплики, походка, прыжки,
  наклоны, разворот, значки (пот, «!», «?», злость, блёстки, «…»). Можно несколько героев или ни одного.
- 3D-локации на three.js внутри Remotion: мягкий свет, тени, размытие фона, движущаяся камера.
- Голос: ElevenLabs (через браузер) или бесплатный Piper. Тайминги слов — whisper.cpp.
- Субтитры по словам, процедурная фоновая музыка без лицензий.
- Весь код проекта лежит внутри `SKILL.md` — Claude записывает его на диск сам.

## Установка

```bash
git clone https://github.com/yeainworks/shorts-remotion ~/.claude/skills/shorts-remotion
```

Перезапусти Claude Code. Нужны Node.js 18+ и ffmpeg.

## Использование

Скажи Claude: «сделай шортс про …» или `/shorts-remotion`. Свой персонаж — дай путь к PNG:

```bash
node tools/char.mjs add hero путь/к/персонажу.png --h 1.2
node tools/char.mjs add hero путь/к/персонажу_злой.png angry
```

Эмоции: `neutral` (обязательно), `happy`, `smug`, `proud`, `laugh`, `nervous`, `shock`, `realize`,
`confused`, `angry`, `sad`, `blank`; служебные `talk`, `blink`, `<эмоция>_talk`, `<эмоция>_blink`.
