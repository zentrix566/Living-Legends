# 在世传奇

一个 Vue 前端小项目，用来记录参与过重大事件、曾被认为已经死去但其实仍然活着的人物。

## 人物文档

人物数据位于 `public/data/living-legends.json`。每个人物建议包含：

- `name`：姓名
- `country`：国家或主要身份归属
- `birthDate`：出生日期，格式为 `YYYY-MM-DD`
- `deathDate`：去世日期；仍在世时填 `null`
- `event`：关联事件
- `majorDeed`：主要事迹，控制在一两句话
- `sourceUrl`：资料来源

当未来有人去世时，只需要把对应人物的 `deathDate` 从 `null` 改成日期，前端会自动把状态显示为红色。

## 开发

```bash
npm install
npm run dev
```

## 作者

zentrix566

## 许可证

本项目基于 MIT License 开源，详见 [LICENSE](LICENSE)。
