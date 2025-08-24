---
title: "Github Pages 自動部署"
date: 2025-08-24
tags: ["Github", "Deploy"]
categories: ["技術"]
toc: false
---
### workflow 核心邏輯
1. Trigger 觸發條件
    - 當 push 到 main 時
    - 手動執行
    - 定時排程
2. Build 建置
3. Deploy 部署

### GitHub Actions Workflow
> 如何啟用 Github Action?

- 只需要在專案的根目錄新增 .github/workflow 的路徑
- Github 會執行路徑下的所有 YAML 文件
> 大致架構？
- [Github 官方參考文件](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax)
1. 基本結構
    ```yaml
    name: Workflow 名稱   # 工作流程名稱，會顯示在 Actions tab (可省略)

    on:                  # 觸發條件
      push:
        branches: [main] # 當 push 到 main branch 觸發
      workflow_dispatch: # 手動觸發
      schedule: # 定時觸發
        # * is a special character in YAML so you have to quote this string
        - cron:  '30 5,17 * * *'

    permissions:         # GitHub Token 權限
      contents: read
      pages: write
      id-token: write

    concurrency:         # 避免多個 workflow 同時跑
      group: "pages"     # 被歸類在 "pages" 這個 group
      cancel-in-progress: false

    defaults:            # 預設執行環境
      run:
        shell: bash

    ```
2. jobs（工作）
- Workflow 可以由多個 jobs 組成
```yaml
jobs:
  build:                  # Job ID
    runs-on: ubuntu-22.04 # runner OS
    steps:                # 逐步執行的動作
      - name: Checkout    # step 描述
        uses: actions/checkout@v4   # 使用別人做好的 action
        with:             # 傳入參數
          submodules: recursive
          fetch-depth: 0

      - name: Run command
        run: echo "Hello World"     # 執行 shell 指令

  deploy:                 # 第二個 job
    runs-on: ubuntu-latest
    needs: build          # 依賴於 build job（先跑 build 再跑 deploy）
    steps:
      - name: Deploy
        uses: actions/deploy-pages@v4
```
```yaml
# 執行順序 job-1 -> job-2 -> job-3
jobs:
  job-1:
  job-2:
    needs: job-1
  job-3:
    needs: [job-1, job-2]
```

3. steps（步驟）

每個 job 裡面包含 steps，step 可以分兩種：

- uses → 使用現成 action
    ```yaml
    - name: Checkout
      uses: actions/checkout@v4
    ```
- run → 自己寫命令
    ```yaml
    - name: Build Hugo site
      run: hugo --minify --gc
    ```

4. 常用語法
- <code style="color:red;">with:</code> 傳參數給 uses
    ```yaml
    with:
      hugo-version: '0.148.2'
      extended: true
    ```
- <code style="color:red;">id:</code> 為某個 step 設定一個 ID，讓之後可以引用它的輸出
    ```yaml
    - name: Setup Pages
      id: pages
      uses: actions/configure-pages@v5

    - name: Build site
      run: hugo --baseURL "${{ steps.pages.outputs.base_url }}/"
      ```
- <code style="color:red;">env:</code> 設定環境變數（只在該 step 有效）
    ```yaml
    - name: Build
      env:
        HUGO_ENVIRONMENT: production
        HUGO_ENV: production
      run: hugo --minify
    ```
