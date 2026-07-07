# 雨课堂论坛发评论经验

## 操作流程（Playwright CLI）

1. **打开论坛页面**：`playwright-cli open "https://ustcshe.yuketang.cn/pro/lms/{course_id}/{leaf_id}/forum/{leaf_id}"`
2. **获取交互元素**：`playwright-cli snapshot` — 在快照中找到 placeholder 为"发表你的观点..."的 textarea（如 `e5`）
3. **点击聚焦输入框**：`playwright-cli click e5`
4. **填写评论内容**：`playwright-cli fill e5 "你的评论内容"`
5. **获取快照找发送按钮**：`playwright-cli snapshot` — 找到 innerText 为"发送"的 button（如 `e8`）
6. **点击发送**：`playwright-cli click e8`
7. **验证发送成功**：等待 3 秒后 `playwright-cli snapshot`，检查页面文本是否还包含"未发言"，若不包含则发送成功

## 常见问题

- **`fill` vs `type`**：优先用 `fill`（清空后写入），输入中文等复杂内容时更可靠；`type` 逐字符模拟按键，适合需要触发 keydown 等事件的场景
- **发送按钮可能不在可见区域**：需要先 `playwright-cli click eN` 滚动到按钮位置，或用 `playwright-cli eval "document.querySelector('button').click()"` 直接 JS 点击
- **会话过期会重定向到登录页**：需要重新登录（可用 `--persistent` 保持 profile）
- **每道题间隔 5±3 分钟随机**：实际用 240-360 秒不等

## 论坛 URL 规律

`https://ustcshe.yuketang.cn/pro/lms/{course_id}/{leaf_id}/forum/{leaf_id}`

已开放的 leaf_id 列表：77993312, 77993318, 77993326, 77993327, 77993328, 77993333, 77993341, 77993335, 77993336, 77993337, 77993350, 77993351, 77993353, 77993354, 77993345, 77993346
