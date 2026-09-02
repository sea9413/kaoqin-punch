# 考勤打卡 · 产品开工令 v1.0.0

> **文档定位**：本文件是产品的「唯一权威说明」。任何人/AI 接手都能凭此继续开发、修复、迭代。
> **整理日期**：2026-09-02
> **当前线上版本**：v1.0.0（首次版本管理 + 页面展示版本号/开发者，2026-09-02 推送）
> **上一版本**：无（此前未做版本管理）
> **代码位置**：`C:\Users\19101\CodeBuddy\20260901203730\index.html`（单文件 ~780 行）
> **线上地址**：https://sea9413.github.io/kaoqin-punch/（GitHub Pages）
> **GitHub 仓库**：https://github.com/sea9413/kaoqin-punch

---

## 二·补充、产品快速信息卡（接手前必读）

| 项 | 值 |
|---|---|
| 线上地址 | https://sea9413.github.io/kaoqin-punch/ |
| GitHub 仓库 | https://github.com/sea9413/kaoqin-punch |
| 数据库 | Supabase（独立项目 `hmqhkpgwngxmfolesvyp`） |
| 表名 | `public.kaoqin_records` |
| 切换密码 | `6666`（前端明文，仅防误触） |
| 开发者 | sea9413 |
| 部署命令 | `node deploy.js`（推送到 `gh-pages` + `main`） |
| 部署分支 | 网站 `gh-pages` / 源码备份 `main` |
| 本地 git | 本目录非 git 仓库，靠 `deploy.js` 直接写 GitHub API |
| 与装货登记关系 | **完全独立**。不同 Supabase 项目、不同仓库、不同 Pages，互不干扰 |

---

## 〇、产品一句话

> 10 人以下小团队用手机网页打卡：打开 → 选名字 → 点「上班/休息」即可。数据存在独立 Supabase 项目，与装货登记项目物理隔离。

---

## 一、产品定位

### 1.1 解决什么问题

小团队分散在不同城市/工地，考勤打卡不想装 App、不想注册账号，希望一个网页点开就用，大家看同一份数据。

### 1.2 目标用户

- **主要用户**：小团队负责人和同事（10 人以下）
- **使用设备**：手机为主
- **技术水平**：不懂技术，要"傻瓜式"

### 1.3 产品形态

| 项 | 选择 | 原因 |
|---|---|---|
| 形态 | 手机网页应用 | 无需装 App，浏览器即用 |
| 部署 | GitHub Pages（`gh-pages` 分支） | 用户已有 GitHub 账号，免费 |
| 后端 | Supabase（独立项目） | 免费额度够，**与装货登记项目完全隔离** |
| 更新 | 代码改完自动上线 | 不用用户操作 |
| 单/多文件 | 单 HTML 文件 | 方便调试、复制、改动 |

---

## 二、核心使用场景

### 场景 1：今天打卡

```
打开网页 → 首次输入/选择名字（本机记住）
     ↓
点「上班」或「休息」→ 当天记录完成
```

### 场景 2：补打卡

```
点「补打卡」→ 选日期 → 选状态 → 提交
     ↓
月份牌对应日期出现紫色「补」标
```

### 场景 3：切换身份代人打卡

```
点「切换」→ 输密码 6666 → 选/输入同事名字
     ↓
后续打卡以该同事名义写入（设计内允许代打卡，无留痕）
```

### 场景 4：看团队状态

```
「同事今天」列表 + 「当月统计」表 + 月份牌
     ↓
全员可见谁上了、谁休了、谁没打
```

---

## 三、功能清单（v1.0.0）

### 3.1 打卡

| 功能 | 说明 |
|---|---|
| 上班/休息打卡 | 点按钮即记当天 |
| 当天覆盖 | 同一天再点不同状态会覆盖；相同状态提示已打卡 |
| 改状态确认 | 已打卡后改状态，弹窗二次确认 |
| 补打卡 | 只能补今天及以前，会标记 `is_supplement=true`，月份牌显示紫色「补」 |

### 3.2 展示

| 功能 | 说明 |
|---|---|
| 今天我状态卡 | 顶部大色块显示今天状态（上班绿 / 休息橙 / 未打卡灰） |
| 月份牌 | 7 列网格，四种颜色状态；支持上一月/下一月翻页 |
| 同事今天 | 列表显示今天全员状态 |
| 当月统计 | 每人出勤 / 休息 / 未打卡天数 |

### 3.3 身份

| 功能 | 说明 |
|---|---|
| 首次选名 | 输入或选择名字，本机 `localStorage` 记住 |
| 修改名字 | 同一人改名，历史记录跟着迁移 |
| 切换身份 | 换另一个人打卡，需密码，**不迁移数据** |

### 3.4 安全与降级

| 功能 | 说明 |
|---|---|
| XSS 防护 | 所有用户输入渲染前 HTML 转义 |
| 表不存在提示 | 自动显示建表 SQL 并提供「复制 SQL」按钮 |
| 切换密码 | `6666`（前端明文，仅防误触） |

---

## 四、数据字段与数据库

### 4.1 字段表

| 字段名（DB） | 中文名 | 类型 | 说明 |
|---|---|---|---|
| `id` | 主键 | uuid | 自动生成 |
| `name` | 姓名 | text | 谁打的卡 |
| `date` | 日期 | text | 格式 `YYYY-MM-DD`（本地时区，不搞 UTC，避免差一天） |
| `status` | 状态 | text | `work` 上班 / `rest` 休息 |
| `is_supplement` | 是否补卡 | boolean | `true` 表示补打卡 |
| `created_at` | 创建时间 | timestamptz | Supabase 默认 |

### 4.2 唯一约束

`unique (name, date)` —— 保证每人每天一条，支持覆盖。

### 4.3 建表 SQL（幂等，可反复执行）

```sql
create table if not exists public.kaoqin_records (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  date text not null,                 -- 格式 YYYY-MM-DD
  status text not null check (status in ('work','rest')),
  is_supplement boolean not null default false,
  created_at timestamptz not null default now(),
  unique (name, date)                -- 同一天每人一条，点错覆盖
);

alter table public.kaoqin_records enable row level security;

-- 小团队内部工具：允许匿名读写（软限制，可接受）
drop policy if exists "anon_select" on public.kaoqin_records;
create policy "anon_select" on public.kaoqin_records for select to anon using (true);
drop policy if exists "anon_insert" on public.kaoqin_records;
create policy "anon_insert" on public.kaoqin_records for insert to anon with check (true);
drop policy if exists "anon_update" on public.kaoqin_records;
create policy "anon_update" on public.kaoqin_records for update to anon using (true) with check (true);

-- 删除策略默认关闭：前端从不需要删除，开着等于任何人能清空全表
-- drop policy if exists "anon_delete" on public.kaoqin_records;
-- create policy "anon_delete" on public.kaoqin_records for delete to anon using (true);

-- 索引
create index if not exists kaoqin_date_idx      on public.kaoqin_records(date);
create index if not exists kaoqin_name_date_idx on public.kaoqin_records(name, date);
```

---

## 五、关键业务规则

### 5.1 打卡覆盖

以 `(name, date)` 为唯一键，`upsert` 覆盖。点错当天再点即可改状态。

### 5.2 未打卡计算

**未打卡不落库**，显示时按「已经过去的天数 − 已打卡天数」计算：
- 未来月份：未打卡 = 0
- 当月：未打卡 = 今天日期 − 已打卡数
- 历史月：未打卡 = 当月总天数 − 已打卡数

### 5.3 切换 vs 改名（重要）

| 操作 | migrate 参数 | 数据库行为 |
|---|---|---|
| 修改名字 | `true` | 旧名历史记录迁到新名（同一人） |
| 切换身份 | `false` | 只换本机身份，**不动数据库** |

⚠️ 若修改名字时目标名已存在，自动降级为「切换」语义，避免 `unique(name,date)` 冲突导致身份错乱。

### 5.4 切换密码

`6666` 硬编码在前端 JS 里，F12 可见。只挡误触，不防恶意。

---

## 六、用户决策记录

| 决策 | 用户选择 | 原因 |
|---|---|---|
| 是否允许代打卡 | ✅ 允许 | 小团队内部工具，换人打卡方便 |
| 切换密码 | ✅ 简单 4 位 | 先能用，后续可升级 |
| 匿名读写 | ✅ 接受 | 小团队，简化部署 |
| 未打卡是否落库 | ❌ 不落库 | 节省空间，显示时计算 |
| 补卡留痕 | ✅ 补标 | 区分正常打卡与补卡 |
| 多项目隔离 | ✅ 独立 Supabase 项目 | 绝对不能影响装货登记 |

---

## 七、技术架构（开发者视角）

### 7.1 技术栈

| 层 | 选型 | 原因 |
|---|---|---|
| 前端 | 单页 HTML + CSS + 原生 JS | 不分离前后端，方便调试 |
| 后端 | Supabase（PostgreSQL + REST API） | 免费额度够 |
| 部署 | GitHub Pages（`gh-pages` 分支） | 用户已有 GitHub 账号 |
| CDN | jsDelivr（Supabase JS） | 主 CDN；失败自动回退 unpkg |

### 7.2 数据库表

仅一张表：`public.kaoqin_records`。

### 7.3 RLS 策略

| 策略 | 状态 | 说明 |
|---|---|---|
| `anon_select` | ✅ 开启 | 列表/统计需要读 |
| `anon_insert` | ✅ 开启 | 打卡需要写 |
| `anon_update` | ✅ 开启 | 覆盖当天需要更新 |
| `anon_delete` | ❌ 关闭 | 前端无删除功能，关闭防数据被清空 |

### 7.4 关键代码位置

> 行号会漂移，以函数名/常量名为准。

| 模块 | 定位方式 |
|---|---|
| 版本号 | `const VER = 'v1.0.0'` |
| Supabase 配置 | `SUPABASE_URL` / `SUPABASE_KEY` / `TABLE` |
| 切换/改名逻辑 | `changeNameTo` |
| 打卡写入 | `punch` / `punchToday` |
| 月份牌渲染 | `renderCalendar` |
| 统计渲染 | `renderStats` |

---

## 八、验收清单（v1.0.0 必须全通过）

### 8.1 基础

- [ ] 打开网页能正常加载
- [ ] 首次打开弹出选名字弹窗
- [ ] 输入名字后进入主界面
- [ ] 页面顶部显示「团队打卡 v1.0.0 | 开发者: sea9413」

### 8.2 打卡

- [ ] 点「上班」成功记录当天
- [ ] 点「休息」成功记录当天
- [ ] 同一天已打卡后改状态，弹窗确认
- [ ] 同一天相同状态重复点，提示已打卡

### 8.3 补打卡

- [ ] 点「补打卡」弹窗
- [ ] 只能选今天及以前日期
- [ ] 提交后月份牌对应日期显示紫色「补」

### 8.4 月份牌与统计

- [ ] 月份牌正确显示本月每天状态
- [ ] 上一月/下一月翻页正常
- [ ] 「同事今天」列表显示全员
- [ ] 「当月统计」出勤/休息/未打卡数字正确

### 8.5 身份切换

- [ ] 点「切换」弹窗
- [ ] 密码错误时切换无效
- [ ] 密码 6666 切换成功
- [ ] 切换后打卡以新名字写入
- [ ] 切换不影响原身份的历史记录

### 8.6 修改名字

- [ ] 点「修改」改名后，历史记录跟着迁移
- [ ] 改成已存在名字时，自动切换不迁移，并提示

---

## 九、明确不做（Out of Scope）

为避免范围蔓延，以下功能**暂不开发**：

- ❌ 真实账号体系 / Supabase Auth
- ❌ 后台名字白名单
- ❌ 代打卡留痕（updated_by）
- ❌ 薪资/工时计算
- ❌ 消息通知
- ❌ 考勤报表导出

如未来要加，需另开工令。

---

## 十、版本历史

| 版本 | 日期 | 主要变更 |
|---|---|---|
| v1.0.0 | 2026-09-02 | 首次版本管理；页面顶部展示版本号与开发者；整理开工令；默认关闭 `anon_delete` 策略 |

---

## 十一、风险与已知问题

| 风险 | 等级 | 缓解 |
|---|---|---|
| anon key 公开，可读写业务表 | 中 | 独立 Supabase 项目，不影响装货登记；小团队内部使用 |
| 切换密码明文在前端 | 中 | 仅防误触，不防恶意；后续可升级为后端校验 |
| 代打卡无留痕 | 中 | 设计取舍 |
| 依赖 CDN | 低 | jsdelivr + unpkg 双保险 |
| 改了 `database.sql` 却忘记执行 | 高 | 表不存在时页面自动提示并附 SQL；升级后仍建议手动整段重跑 |

---

## 十二、部署

```bash
node deploy.js
```

把 `index.html` 和 `README.md` 推到 `gh-pages`（网站）和 `main`（源码备份）。
轮换 token 时用 `GITHUB_PAT=xxx node deploy.js`。

GitHub Pages 配置：`source.branch=gh-pages`，`path=/`。
推送后生效约需 30~60 秒。
