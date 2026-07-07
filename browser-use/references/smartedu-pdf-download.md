---
description: 国家中小学智慧教育平台受保护 PDF 的下载方法
---

# 国家中小学智慧教育平台 PDF 下载

## 案例背景

国家中小学智慧教育平台 (basic.smartedu.cn) 的教材详情页嵌入了 pdf.js 阅读器来展示 PDF。对于需要认证的受保护 PDF，直接下载 URL 会返回 401 错误。

## 目标教材

道德与法治八年级上册: https://basic.smartedu.cn/tchMaterial/detail?contentType=assets_document&contentId=5a29b928-d6da-4131-a69e-4c54941f7651&catalogType=tchMaterial&subCatalog=tchMaterial

## 解决方案

通过浏览器内置的 pdf.js 阅读器提取已加载的 PDF 数据：

```javascript
(async () => {
  const iframe = document.querySelector('iframe');
  if (!iframe) return console.error('未找到 iframe');
  const win = iframe.contentWindow;
  if (!win || !win.PDFViewerApplication) return console.error('PDF.js 未初始化');
  const app = win.PDFViewerApplication;
  const doc = app.pdfDocument;
  if (!doc) return console.error('PDF 文档未加载');
  const data = await doc.getData();
  const blob = new Blob([data], {type: 'application/pdf'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  // 从页面标题或 URL 中提取文件名
  const title = document.title || 'download';
  a.download = title.replace(/[\\/:*?"<>|]/g, '_') + '.pdf';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
})();
```

## 操作步骤（Playwright CLI）

1. **打开页面**：`playwright-cli open "https://basic.smartedu.cn/tchMaterial/detail?contentType=assets_document&contentId=5a29b928-d6da-4131-a69e-4c54941f7651&catalogType=tchMaterial&subCatalog=tchMaterial"`
2. **确认标签页**：`playwright-cli tab-list` — 确认 PDF 预览页面的标签页已打开
3. **切换到目标标签页**（如果有多标签）：`playwright-cli tab-select <tab_index>`
4. **确认 PDF 已加载**：`playwright-cli eval "document.querySelector('iframe')?.contentWindow?.PDFViewerApplication?.pdfDocument ? 'loaded' : 'not loaded'"`
5. **检查页数**（可选确认）：`playwright-cli eval "document.querySelector('iframe').contentWindow.PDFViewerApplication.pagesCount"`
6. **执行提取脚本**：将上方 JavaScript 代码通过 `playwright-cli eval` 执行（注意转义引号，或写入 `.js` 文件后用 `--file` 参数加载）
7. **确认下载**：`playwright-cli snapshot` 检查页面状态，确认文件已触发下载

### 注意事项

- PDF 必须在 pdf.js 中完全加载后才能提取；步骤 4-5 先确认 `pdfDocument` 存在且 `pagesCount` > 0
- 该方法利用的是用户在浏览器中已合法访问的资源
- JavaScript 代码已内置空值检查，任一步骤失败会打印明确错误
- Windows 上 URL 带 `&` 可能被 shell 截断，用双引号包裹或改用 Git Bash
