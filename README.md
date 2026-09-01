# kaoqin-punch · 小团队网页打卡工具

给 10 人以下、分散不同城市的小团队用的网页版打卡工具。不用装 App、不登录、公网可用。
打开网页 → 选名字 → 点「上班 / 休息」即可。

- 在线地址：https://sea9413.github.io/kaoqin-punch/
- 源码仓库：https://github.com/sea9413/kaoqin-punch

## 使用

1. 打开上面的链接，首次输入或选择自己的名字（本机 `localStorage` 记住，换手机重选）。
2. 点「上班」或「休息」即记当天；**当天点错了再点另一个即覆盖**（每人每天只保留一条）。
3. 「补打卡」补录之前漏掉的天 —— 选日期 → 选状态 → 点「提交补卡」。只能补自己，会打紫色「补」标。

月份牌四种状态：上班（绿） / 休息（橙） / 未打卡（灰） / 补（紫色角标）。
**未打卡不落库**，只是显示时计算得出。

---

## 首次部署要做的两件事

### 1. 建表（一次性，必须手动）

只有 publishable key、没有 service_role，**无法用 API 建表**，必须人工执行一次：

> Supabase 后台 → 左侧 **SQL Editor** → 粘贴下面这段 SQL → **Run** → 刷新网页。

页面在检测到表不存在时，会自动把这段 SQL 显示在顶部并提供「复制 SQL」按钮，直接复制即可：

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
drop policy if exists "anon_delete" on public.kaoqin_records;
create policy "anon_delete" on public.kaoqin_records for delete to anon using (true);

-- 顺带给日期/姓名建索引，记录多了也快（可反复执行，不会报错）
create index if not exists kaoqin_date_idx      on public.kaoqin_records(date);
create index if not exists kaoqin_name_date_idx on public.kaoqin_records(name, date);
```

### 2. 改配置

`index.html` 顶部的三个常量换成你自己的：

```js
const SUPABASE_URL = 'https://xxx.supabase.co';
const SUPABASE_KEY = 'sb_publishable_xxx';
const TABLE        = 'kaoqin_records';
```

---

## 数据表

表 `kaoqin_records`：

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | uuid | 主键，自动生成 |
| `name` | text | 姓名 |
| `date` | text | 日期，格式 `YYYY-MM-DD`（本地时区，不搞 UTC，避免差一天） |
| `status` | text | `work` 上班 / `rest` 休息 |
| `is_supplement` | boolean | 是否补打卡 |
| `created_at` | timestamptz | 记录创建时间 |

唯一约束 `unique (name, date)` —— 保证每人每天一条，支持覆盖。

---

## 部署

```bash
node deploy.js
```

把 `index.html` 和 `README.md` 推到 `gh-pages`（网站）和 `main`（源码备份）两个分支。
轮换 token 时用 `GITHUB_PAT=xxx node deploy.js`。

GitHub Pages 配置：`source.branch=gh-pages`，`path=/`，`build_type=legacy`。
推送后生效约需 30~60 秒。

---

## 已知限制

- **匿名读写，无真实鉴权**：拿到链接的人都能读写全部记录。仅适用于熟人小团队，别用在正式考勤/薪资场景。
- **名字自由输入**：前端已做 HTML 转义（防存储型 XSS），但无法阻止他人冒名打卡 —— 补卡只能补自己是**前端软限制**，懂技术的人能改请求体。
- **未打卡不落库**：统计时按「已过天数 − 已打卡」计算，未来日期不计入（否则 9/1 打开会显示"未打卡 29 天"）。
- **无防重复提交**：同一天快速连点会短暂读到旧值，但 upsert 最终一致。
- **名单无后台管理**：由"历史出现过的名字"自动汇总。
- **依赖 CDN** 加载 supabase-js；jsdelivr 失败时自动回退 unpkg。

若后续要增强安全，优先级建议：
① 加 service_role 走后端代理做建表与鉴权；② 名字白名单后台管理；③ 补卡留痕（谁在何时补）。
