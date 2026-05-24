# Infinite Canvas Project State

更新日期：2026-05-24

這份文件是本專案的輕量狀態紀錄。之後每次 Codex 或任何接手者開始工作，都要先讀這份文件，不要只靠對話記憶或猜測。

## 核心約定

- 本專案目前對外服務是 `https://canvas.qwenr.com`。
- 使用者說「線上」、「已部署」、「網站」時，預設先理解為 VPS 上的服務，不是 Windows 本機資料夾。
- VPS 主要路徑：
  - 專案根目錄：`/opt/canvas`
  - 應用程式碼：`/opt/canvas/app`
  - Compose 檔：`/opt/canvas/docker-compose.yml`
- Docker 服務名稱：`canvas-infinite-app`。
- 反向代理使用既有 `platform-caddy` 與外部 Docker network `edge`。
- GitHub remote：`origin https://github.com/kuojenski/Infinite-Canvas.git`
- 每次實質更新專案，都要像自動發文系統一樣同步 GitHub：本機修改完成後要建立 commit 並 push 到 `origin/main`，除非使用者明確說「這次不要 commit / 不要 push」。
- 不要把「本機檔案已改」當成「線上已生效」。線上要同步到 VPS、重建容器、再用公開網址驗證。
- 不要把「已推到 GitHub」當成「已部署到 VPS」。

## 開工前固定流程

每次開始改這個專案前，先做這些檢查：

1. 讀這份文件：`PROJECT_STATE.md`
   - 先確認目前的線上約定、部署路徑、最近狀態與禁忌事項。

2. 查本機工作樹：

```bash
git status --short --branch
git log --oneline -8
```

目前 2026-05-24 檢查到本機工作樹已有多個既有未提交變更，包含 `main.py`、多個 `static/*.html`、登入頁、API 設定頁、Mac 說明與部署檔。除非使用者明確要求整理或提交，不要把這些既有變更當成自己本次工作的成果，也不要回復它們。

3. 如果本次工作會影響線上行為，要查 VPS，而不是只看本機：

```bash
ssh geoflow-vps "cd /opt/canvas && git status --short --branch || true"
ssh geoflow-vps "cd /opt/canvas && docker compose ps"
curl -I https://canvas.qwenr.com/api/health
```

如果 SSH 或公開網址驗證失敗，要明確說「無法驗證線上狀態」，不要猜。

## 收工前固定流程

每次完成一段工作後，更新這份文件：

- 本次改了什麼。
- 有沒有 commit，commit hash 是什麼。
- 有沒有推到 GitHub。
- 有沒有部署到 VPS。
- 線上驗證結果是什麼。
- 下次接手要注意什麼。

如果只改本機文件，也要寫清楚「尚未部署 VPS」。

## GitHub 同步規則

本專案要比照自動發文系統：每次更新都必須留下 GitHub 紀錄。

標準收工順序：

1. 確認本次改動範圍：

```bash
git status --short
git diff --stat
```

2. 只 stage 本次工作相關檔案，不要把既有未提交變更混進來。

3. 建立清楚 commit：

```bash
git commit -m "<簡短描述本次更新>"
```

4. 推到 GitHub：

```bash
git push origin main
```

5. 回來更新本文件的「最近狀態」，明確寫：
   - commit hash
   - 已推到 GitHub：是
   - 已部署 VPS：是 / 否
   - 驗證結果

若當下因為 GitHub 認證、網路、衝突、或使用者要求而不能 push，要在最後回報與本文件中寫清楚「尚未推到 GitHub」以及原因。

## 目前專案結構

- `main.py`：FastAPI 主服務，包含靜態頁面、API provider 設定、線上圖片生成、ComfyUI/RunningHub/ModelScope 等流程。
- `auth_workspace.py`：會員登入、註冊、session cookie、workspace/auth gating。
- `static/index.html`：主入口與品牌呈現重點之一。
- `static/login.html`：登入頁與品牌呈現重點之一。
- `static/angle.html`：角度控制頁，雲端生成路由容易跟 provider 設定有關。
- `static/api-settings.html`：API provider 管理頁。
- `deploy/vps/docker-compose.yml`：VPS Docker/Caddy 部署設定。
- `DEPLOY_VPS.md`：VPS 同步、重建與驗證流程。

## 目前線上部署設計

- 線上網域：`https://canvas.qwenr.com`
- Compose 環境：
  - `QWENR_AUTH_REQUIRED=1`
  - `QWENR_ALLOW_REGISTRATION=0`
  - `QWENR_SESSION_COOKIE_NAME=qwenr_session`
  - `QWENR_SESSION_COOKIE_SECURE=1`
- 未登入時，`/` 與靜態頁面會導向 `/login`。
- `/api/health` 與 `/api/app-info` 必須保持公開，否則 Docker healthcheck 或首頁 bootstrap 會壞。
- 第一位帳號可作為平台管理者；註冊關閉後，會員管理應由管理員控管。

## 已知重要變更脈絡

- 此專案曾經部署到 VPS，並以 `canvas.qwenr.com` 驗證過。
- 此專案曾經加入會員登入門檻，不能隨便移除 auth gating。
- 專案對外品牌曾改成 `kiki studio`，可見文字優先在 `static/index.html` 與 `static/login.html` 檢查。
- 角度控制曾經因前端硬接 ModelScope，導致不能使用使用者設定的 OpenRouter/provider。後續遇到雲端生圖接不到 OpenRouter 時，先查 `static/angle.html` 是否有走 `/api/config`、`/api/providers`、`/api/online-image`，不要直接假設後端壞掉。
- 2026-05-24 起，主介面已移除獨立「細節增強」頁與左側入口；`static/enhance.html` 檔案暫時未刪，避免誤傷既有未提交內容或底層 workflow，但不再從主頁導覽載入。

## 部署流程摘要

更新 VPS 時，原則上使用 `DEPLOY_VPS.md` 的流程：

```bash
rsync -avz --delete \
  --exclude '.git' \
  --exclude 'python' \
  --exclude 'packages' \
  --exclude 'output' \
  --exclude 'assets' \
  --exclude 'data' \
  ./ geoflow-vps:/opt/canvas/app/

ssh geoflow-vps "cd /opt/canvas && docker compose up -d --build"
```

驗證至少要包含：

```bash
ssh geoflow-vps "docker ps --filter name=canvas-infinite-app"
curl -I https://canvas.qwenr.com/api/health
curl -I https://canvas.qwenr.com/
```

登入門檻存在時，`/` 回 `307` 到 `/login` 是合理結果；不要誤判為壞掉。

## 暫時不要主動做的事

- 不要為了「整理乾淨」回復目前本機已有的未提交變更。
- 不要把內部檔名、容器名、路徑全部改成 `kiki studio`；目前品牌改名優先是可見文字，不是深層重命名。
- 不要關掉會員門檻或把 `/api/*` 全部公開，除非使用者明確要求。
- 不要把 provider 重新硬寫死成 ModelScope、Comfly 或任何單一服務。
- 不要改掉 Caddy/edge network 部署模式，除非 VPS 架構也一起確認。
- 不要把 `output`、`assets`、`data`、API 設定資料用 `rsync --delete` 誤刪；部署排除規則要保留。

## 最近狀態

- 2026-05-24：新增本文件，建立此專案的固定狀態紀錄與交接規則。
  - commit：文件自身 commit 以 GitHub / `git log` 為準，避免為了寫入自己的 commit hash 造成反覆改檔。
  - 已推到 GitHub：是，已納入本次 `PROJECT_STATE.md` 文件提交。
  - 已部署 VPS：否，這次只是本機新增狀態文件。
  - 驗證：已檢查本機 `git status --short --branch`、`git log --oneline -8`、`README.md`、`DEPLOY_VPS.md`、`deploy/vps/docker-compose.yml` 與關鍵 API/auth/provider 路徑。

- 2026-05-24：補上 GitHub 同步規則。
  - 目標：之後每次實質更新都要 commit 並 push 到 `origin/main`，除非使用者明確要求不要。
  - commit：文件自身 commit 以 GitHub / `git log` 為準，避免為了寫入自己的 commit hash 造成反覆改檔。
  - 已推到 GitHub：是，已納入本次 `PROJECT_STATE.md` 文件提交。
  - 已部署 VPS：否，這次只是本機文件規則更新。

- 2026-05-24：修正普通畫布工具列漏掉 `API生成` 入口。
  - 原因：`static/canvas.html` 仍有 `addGeneratorNode()` 與 `generator` 節點邏輯，但上方快捷工具列和新增選單漏掉 API 生成按鈕，使用者只能看到 `MS生成`、`視頻生成`、`ComfyUI` 等入口。
  - 修正：在快捷工具列補回 `API生成`，並在右鍵/新增選單補回 `API生成`，都接到既有 `generator` 節點。
  - 驗證：用靜態 HTTP server 開啟 `http://127.0.0.1:8000/static/canvas.html`，確認 `#quickToolbar` 與 `#createMenu` DOM 都包含 `API生成`。完整 FastAPI 本機驗證未完成，因全域 Python 的 FastAPI/Pydantic 版本不相容、專案內嵌 Python 缺 `requests`。
  - commit：`5deb47b Restore API generate canvas entry`
  - 已推到 GitHub：是。
  - 已部署 VPS：是，已只同步 `static/canvas.html` 到 `/opt/canvas/app/static/canvas.html` 並執行 `docker compose up -d --build`。
  - 線上驗證：`canvas-infinite-app` healthy；`https://canvas.qwenr.com/api/health` 回 `{"ok":true,"service":"infinite-canvas"}`；容器內 `/app/static/canvas.html` 第 858 與 913 行都包含 `API生成`。

- 2026-05-24：移除主介面的獨立「細節增強」頁入口。
  - 原因：使用者覺得細節增強頁對目前工作流沒有用，左側按鈕和該頁面入口都可以不要。
  - 修正：從 `static/index.html` 移除左側「細節增強」導覽項、`frame-enhance` iframe，以及 `PAGE_IDS` 中的 `enhance`，讓舊的 localStorage enhance 狀態回到預設 `online`。
  - 注意：未刪除 `static/enhance.html` 檔案，也未移除畫布內 ComfyUI 的細節增強模式，避免誤傷底層 workflow 或既有未提交變更。
  - commit：`0b41736 Remove standalone enhance page entry`
  - 已推到 GitHub：是。
  - 已部署 VPS：是，已只同步 `static/index.html` 到 `/opt/canvas/app/static/index.html` 並執行 `docker compose up -d --build`。
  - 線上驗證：`canvas-infinite-app` healthy；`https://canvas.qwenr.com/api/health` 回 `{"ok":true,"service":"infinite-canvas"}`；容器內 `/app/static/index.html` 已找不到 `frame-enhance` 與 `nav.enhance`，`PAGE_IDS` 為 `['angle','online','canvas','api-settings','comfyui-settings']`。

- 2026-05-24：在普通無限畫布新增「提示詞庫」。
  - 目標：讓使用者可保存常用提示詞，日後用名稱辨識、編輯、刪除，並可套用到選中的提示詞節點或新增成新的提示詞節點。
  - 修正：在 `static/canvas.html` 快捷工具列新增「提示詞庫」按鈕，加入提示詞庫 modal、列表、名稱/內容表單，以及保存、編輯、刪除、套用、新增到畫布等前端流程。
  - 儲存：目前使用瀏覽器 `localStorage` 的 `canvas_prompt_library_v1`，屬於同一瀏覽器本機保存；尚未接後端帳號同步。
  - commit：`972c3f8 Add canvas prompt library`
  - 已推到 GitHub：是。
  - 已部署 VPS：是，已只同步 `static/canvas.html` 到 `/opt/canvas/app/static/canvas.html` 並執行 `docker compose up -d --build`。
  - 驗證：本地 Playwright 靜態頁測試確認「提示詞庫」按鈕、modal、保存、localStorage、列表卡片與「套用 / 新增到畫布 / 編輯 / 刪除」按鈕可渲染；線上 `canvas-infinite-app` healthy，`https://canvas.qwenr.com/api/health` 回 `{"ok":true,"service":"infinite-canvas"}`，容器內 `/app/static/canvas.html` 已包含 `promptLibraryModal`、`PROMPT_LIBRARY_KEY` 與「提示詞庫」。
