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
  "defaultLanguage": "en",
  "languages": ["en", "zh-Hans", "zh-Hant", "ja", "ko", "de", "fr"],
  "countries": [
    {
      "code": "CN",
      "names": { "en": "China", "zh-Hans": "中国" },
      "file": "countries/CN.json",
      "regionHints": ["CN"]
    }
  ]
}
```

- `code`：ISO 3166-1 alpha-2
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
| `month` / `day` | 月日；农历按中国农历月日 |
| `leapMonth` | 可选，农历是否闰月，默认 `false` |
| `recurrence` | `yearly` 或 `none` |
| `datesByYear` | 可选，`{"2026":"04-05"}`，节气等逐年公历日；有则优先 |
| `date` | 可选，一次性公历 `YYYY-MM-DD`（`recurrence: none`） |
| `notes` | 可选多语言备注 |

农历条目由 App 转成**下一次**公历日再写入提醒日；公历 `yearly` 使用年循环。

## 维护建议

- 先改 JSON → 本地用 App 拉 raw 或 bundle 校验 → 再 push。
- 改 `schemaVersion` 仅在破坏兼容时递增。
- 中国节气优先维护近若干年的 `datesByYear`。
