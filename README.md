# 📌 校园时光地图 · 邯郸市第一中学中华南校区

一个基于 **Leaflet.js** 的交互式校园地图网站。访客点击地图任意位置即可留下"校园时光"（照片 + 文字），所有数据保存在浏览器本地（localStorage），刷新不丢失。

## ✨ 功能

- 🗺️ 高德地图底图（国内可访问）+ OpenStreetMap 可切换
- 🎯 地图锁定在学校约 500 米范围，不可拖出
- 📍 点击地图 → 弹窗表单（上传 JPG/PNG 照片 + 描述文字）
- 🧷 保存后生成图钉标记，点击可查看照片与描述
- 💾 默认数据存于浏览器 localStorage（刷新不丢失）
- ☁️ 已接入 **Supabase** 云端，实现**所有访客共享同一份数据**（云端不可用时自动回退本地）
- 🎨 清新校园风，淡紫配色 + 毛玻璃弹窗

## 🚀 部署到 GitHub Pages（免安装 Git，网页直接上传）

> 如果你不想安装任何软件，用 GitHub 网页上传是最快的上线方式。

1. **注册/登录 GitHub**：打开 https://github.com ，注册账号并登录。
2. **创建仓库**：
   - 点右上角 **＋ → New repository**
   - Repository name 填：`<你的用户名>.github.io`（例如 `zhangsan.github.io`）
   - 选 **Public**，**不要勾选** "Add a README file"，点 **Create repository**
3. **上传文件**：
   - 在新仓库页面点 **Add file → Upload files**
   - 把本文件夹里的 **`index.html`** 拖进去
   - 下方 Commit changes → 直接点 **Commit changes**
4. **开启 Pages**：
   - 进入仓库 **Settings → Pages**（左侧菜单）
   - **Build and deployment** 的 Source 选 **Deploy from a branch**
   - Branch 选 **main** 和 **/(root)**，点 **Save**
   - 等 1~2 分钟后，访问 `https://<你的用户名>.github.io` 即可看到网站 ✅

> ⚠️ 仓库名必须是 `<用户名>.github.io` 时，网站就托管在该域名根路径；
> 如果用的是其他仓库名（如 `campus-map`），网址则是 `https://<用户名>.github.io/campus-map/`。

## ☁️ 云端共享（所有访客共享同一份数据）

本项目已接入 **Supabase**（免费 BaaS）作为云端数据源：访客添加/删除标记时，数据会同步到云端，所有访客看到同一份数据。云端连不上时自动回退到本地模式，网站照常可用。

**配置说明（已内置在 `index.html` 的 `SUPABASE_CONFIG` 中）：**

1. **注册 Supabase**：打开 https://supabase.com ，用 GitHub 或邮箱注册（免费，无需手机号/付费）。
2. **创建项目**：New project → 填项目名、选区域（国内建议 **新加坡 Singapore**）、生成数据库密码 → 创建。
3. **拿密钥**：项目首页点 **API Keys / Copy project URL and API keys**，复制 **Project URL** 和 **Publishable key**。
4. **建数据表**：进入 **SQL Editor**，执行以下 SQL（含建表 + 匿名读写权限）：
   ```sql
   create table if not exists public.campus_markers (
     id uuid primary key default gen_random_uuid(),
     lat double precision not null,
     lng double precision not null,
     crs text not null default 'gcj02',
     photo text, description text,
     created_at timestamptz not null default now()
   );
   alter table public.campus_markers enable row level security;
   create policy "public_read" on public.campus_markers for select using (true);
   create policy "public_insert" on public.campus_markers for insert with check (true);
   create policy "public_update" on public.campus_markers for update using (true) with check (true);
   create policy "public_delete" on public.campus_markers for delete using (true);
   ```
5. **填入 `index.html`**：打开 `index.html`，找到 `SUPABASE_CONFIG`，把 `url` 和 `anonKey` 换成你的项目值，`enabled` 保持 `true`，保存后重新上传到 GitHub。

配置好后，页面左上角会显示 **「☁️ 云端共享 · N 条」**；云端不可用时显示 **「⚠️ 云端不可用 · 本地模式」**，未配置时显示 **「💾 本地模式」**（功能照常可用）。

> 🔒 安全提示：`Publishable key`（匿名公钥）是公开给浏览器的（Supabase 官方做法）。数据表已开启 RLS，目前匿名可读写；如需更细的权限（比如只允许部分人删除），可以在 Supabase 的 RLS 策略里进一步限制。

## 🔁 以后更新网站

- **方式一（网页）**：在仓库页面 **Add file → Upload files**，把新的 `index.html` 覆盖上传（勾选 "Commit directly to the main branch"）。
- **方式二（VS Code + Git，推荐长期维护）**：
  1. 安装 Git for Windows：https://git-scm.com/download/win
  2. 在 VS Code 打开本文件夹，用 **源代码管理** 面板初始化仓库、提交
  3. 添加远程仓库并推送（或直接用 VS Code 的 **GitHub** 扩展一键发布）

## 🧭 技术说明

- 单文件架构（HTML + CSS + JS 全在一个 `index.html` 里）
- Leaflet 1.9.4，多 CDN 容错加载（jsDelivr → unpkg → bootcdn → staticfile），国内访问优先走 jsDelivr
- 底图瓦片：高德（默认）/ OSM 镜像，支持切换
- 内置 **WGS-84 ↔ GCJ-02** 坐标转换，保证两种底图下标记位置一致
- 图片自动压缩（640px）后再存储，兼顾清晰度与存储容量
- 已接入 **Supabase REST API** 云端共享：云端不可用或未配置时自动回退本地模式

## 📁 文件

| 文件 | 说明 |
| --- | --- |
| `index.html` | 网站全部代码（地图 + 交互 + 样式） |
| `README.md` | 本说明文档 |
