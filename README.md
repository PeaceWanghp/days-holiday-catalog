# Days Holiday Catalog

自建节日数据源（供「提醒日」App 导入）。主语言为 **English**，国家名另提供 **zh-Hans**、**zh-Hant**（台湾繁体）、**ja**、**ko**、**de**、**fr**；后续可继续追加语言键。

## GitHub 仓库怎么放

1. 新建仓库，例如 `days-holiday-catalog`（Public，便于 `raw.githubusercontent.com` 拉取）。
2. 把本目录内容放到仓库 **根目录**（或某个分支如 `main`）：

```text
/
  countries.json
  countries/
    CN.json
    US.json
    …
  README.md
```

3. 推送到 GitHub 后，目录地址形如：

```text
https://raw.githubusercontent.com/<USER>/<REPO>/<BRANCH>/countries.json
https://raw.githubusercontent.com/<USER>/<REPO>/<BRANCH>/countries/CN.json
```

4. 在 App 的 `HolidayCatalogConfig.remoteBaseURL` 中改成你的：

```text
https://raw.githubusercontent.com/<USER>/<REPO>/<BRANCH>
```

不要末尾斜杠。

### 可选：GitHub Pages

也可开 Pages，用 `https://<user>.github.io/<repo>` 作为 base（同样提供 `countries.json` 与 `countries/*.json`）。

## 文件格式

### `countries.json`（国家列表）

```json
{
  "schemaVersion": 1,
  "version": 1,
  "defaultLanguage": "en",
  "languages": ["en", "zh-Hans", "zh-Hant", "ja", "ko", "de", "fr"],
  "countries": [
    {
      "code": "CN",
      "hasData": true,
      "names": { "en": "China", "zh-Hans": "中国" },
      "file": "countries/CN.json",
      "regionHints": ["CN"]
    }
  ]
}
```

- `version`：目录内容版本。国家列表或节日数据有实质变化时递增
- `code`：ISO 3166-1 alpha-2
- `hasData`：该国文件是否有节日条目。列表里只保留 `true` 的国家
- `names`：多语言国名；缺键时回退 `en` → 任意已有键
- `file`：相对仓库根的国家文件路径
- `regionHints`：用于匹配系统 `Locale.region`（可含 `HK`/`MO` 等）

### `countries/XX.json`（单国数据）

```json
{
  "schemaVersion": 1,
  "code": "CN",
  "names": { "en": "China", "zh-Hans": "中国" },
  "sections": [
    {
      "id": "festival",
      "names": { "en": "Festivals", "zh-Hans": "节日" },
      "items": [
        {
          "id": "cn-spring-festival",
          "names": { "en": "Spring Festival", "zh-Hans": "春节" },
          "calendar": "lunar",
          "month": 1,
          "day": 1,
          "leapMonth": false,
          "recurrence": "yearly"
        },
        {
          "id": "cn-womens-day",
          "names": { "en": "Women's Day", "zh-Hans": "妇女节" },
          "calendar": "gregorian",
          "month": 3,
          "day": 8,
          "recurrence": "yearly"
        }
      ]
    }
  ]
}
```

#### `sections`

App **只显示数据里存在的 section**；不同国家 section 数量可以不同。`id` 建议稳定英文短码（`festival` / `solarTerm` / `publicHoliday`…），展示名走 `names`。

#### `items` 字段

| 字段 | 说明 |
|------|------|
| `id` | 全球唯一稳定 ID（导入去重用） |
| `names` | 多语言名称；主语言 `en` |
| `calendar` | `gregorian` 公历 / `lunar` 农历（中国历） |
| `month` / `day` | 月日；农历按中国农历月日。`lastDay` 为真时可不写 `day`。有 `dateRule` 时，`day` 只给不认识该规则的客户端做回退 |
| `lastDay` | 可选。农历该月最后一天（除夕为腊月最后一天，即上一年最后一天） |
| `leapMonth` | 可选，农历是否闰月，默认 `false` |
| `recurrence` | `yearly` 或 `none` |
| `date` | 可选，一次性公历 `YYYY-MM-DD`（`recurrence: none`） |
| `dateRule` | 可选。公历日期不固定时使用，优先于固定 `month`/`day` |

公历不固定的日期用三种 `dateRule.kind`：

`yearMod100Linear`：设 `Y` 为年份后两位，

```text
日 = ⌊Y × factor + constant⌋ − ⌊Y / 4⌋
```

`ranges` 按公历年选择 `constant`，`exceptions` 替换公式结果，然后再加 `dayOffset`（默认 0）。月份用 `dateRule.month`。清明、冬至、秋分、日本春分日和秋分日、韩国寒食都用这条式子，只是常数不同。寒食是清明的前一天，`dayOffset` 为 `-1`，例外仍写清明当天。韩国寒食是韩国标准时冬至后再过 105 天。节气在公历闰日之前时设 `beforeLeapDay` 为 true，闰年不把当年 2 月 29 日算进闰年数；日本节分是立春前一天。

`nthWeekday`：该月的第 `nth` 个 `weekday`。`weekday` 取 `sunday` 到 `saturday`，`nth` 为 1–5，`-1` 表示最后一个。母亲节是 5 月第二个星期日，美国阵亡将士纪念日是 5 月最后一个星期一。

`westernEaster`：公历复活节，春分后第一次满月之后的星期日。月份会在 3 月和 4 月之间变化。`dayOffset` 从复活节当天再偏移：耶稣受难日为 `-2`，复活节星期一为 `1`，英国母亲节为 `-21`，德国玫瑰星期一为 `-48`，耶稣升天节为 `39`，圣灵降临节为 `49`，圣灵降临节星期一为 `50`。

农历节日按农历月日显示；除夕用 `lastDay`，表示腊月最后一天。带 `dateRule` 的条目每年按规则重算。

## 维护建议

- 先改 JSON → 本地用 App 拉 raw 或 bundle 校验 → 再 push。
- 改 `schemaVersion` 仅在破坏兼容时递增。
