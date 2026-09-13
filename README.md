# 隐私政策与支持页

这个目录负责把两个**必填的公开 URL** 变成审核员和用户能在浏览器里打开的网页。

| 文件 | 用途 | ASC 里填到哪 |
| --- | --- | --- |
| `index.html` | **隐私政策** | `App 信息` → 隐私政策网址（URL） |
| `support.html` | **支持页** | `App 商店` → 版本页 → 技术支持网址（URL），**必填** |
| `demo.html` | 审核演示视频页 | 只放在审核备注里，不进 ASC 字段 |

## 两个 ASC 字段（原样复制）

```
隐私政策网址   https://sunny2023z.github.io/FlickRoll-privacy/
技术支持网址   https://sunny2023z.github.io/FlickRoll-privacy/support.html
```

> ⚠️ **GitHub Pages 路径区分大小写。** 全小写的 `flickroll-privacy` 会 404，
> 必须原样粘贴带大写的 `FlickRoll-privacy`，不能手打。

## 线上状态

| URL | 内容 |
| --- | --- |
| `/` | 隐私政策（= `index.html`） |
| `/support.html` | 支持页 |
| `/demo.html` | 审核演示视频页，由 `DEPLOY-DEMO.md` 部署 |

核对线上与本地是否一致（字节数应相同）：

```sh
for f in "" support.html demo.html; do
  printf "%-16s " "/$f"
  curl -s -o /dev/null -w "%{http_code}  %{size_download}\n" \
    "https://sunny2023z.github.io/FlickRoll-privacy/$f"
done
```

> **支持页为什么是必填的**：Apple 官方原文要求 Support URL
> "must lead to **actual contact information** (legal address, email address, telephone number)"，
> 且 "**This property is required** and can be localized"。
> `support.html` 里放的是 `1056128378@qq.com`，并配了六条与代码实际行为一致的常见问题。

## 语言切换怎么工作

纯 CSS 的 `:target`，**没有 JavaScript**，禁用 JS 也能切换。

- 默认（URL 无 hash）：显示中文
- URL 带 `#en`：显示英文

**改 HTML 时必须守住两条结构约束：**

1. `<section id="en">` 必须排在 `<section id="zh">` **之前** ——
   `#en:target ~ #zh` 用的是后续兄弟选择器，顺序反了规则就不成立
2. 语言按钮高亮用 `body:has(#en:target)`。`:has()` 需要 Safari 15.4+ / Chrome 105+，
   **不支持时只是按钮不高亮，切换本身仍然正常**（渐进增强，不是依赖项）

---

## ⚠️ 改隐私政策时要同步四处

漏改任何一处都会造成「App 内说的」和「ASC 上填的」不一致 —— 这正是审核员会交叉比对的地方。

| # | 位置 | 用途 |
| --- | --- | --- |
| 1 | `docs/privacy-policy.md` | 中文源文本 |
| 2 | `docs/privacy-policy.en.md` | 英文源文本 |
| 3 | `docs/privacy-site/index.html` | **审核员和用户实际看到的** |
| 4 | `FlickRoll/Views/SettingsView.swift` 的 `PrivacyPolicyText.sections` | **App 内入口**（Guideline 5.1.1(i) 要求） |

**同步的标准是「实质主张一致」，不是逐字相同。** 第 4 份有意更简短、且不含系统权限弹窗的
原文引用（弹窗就在用户眼前，App 内再抄一遍没意义），但它与前三份的**五个事实主张完全一致**：

1. 不收集、不上传、不分享任何数据
2. **不包含任何网络请求代码**（全仓无 `URLSession` / `URLRequest` / `NWConnection`）
3. **没有账号系统**
4. 删除是**移到系统「最近删除」**，不是永久删除
5. **30 天**内可从系统「最近删除」恢复（出处 <https://support.apple.com/118558>）

## 措辞禁令（改的时候别踩）

- ❌ 不得写「永久删除 / permanently delete」—— 实际是移到系统「最近删除」
- ❌ 不得写「人脸识别 / face recognition」—— 全仓无 `VNDetectFace*`，
  画面分析只有 `VNGenerateImageFeatureRequest` 类的图像特征指纹
- ❌ 英文不得新增中文里没有的承诺 —— 英文是同一份政策的呈现，不是独立文本

## 另一处联动的文案

系统权限弹窗文案在 `FlickRoll.xcodeproj/project.pbxproj` 的
`INFOPLIST_KEY_NSPhotoLibraryUsageDescription`，被前三份**逐字引用**。
改了它，那三处的引用也要跟着改。

```sh
# 权限文案与部署页是否逐字一致（期望输出 2：中英各一次）
python3 - <<'PY'
import re
pbx = open('FlickRoll.xcodeproj/project.pbxproj', encoding='utf-8').read()
s = re.search(r'INFOPLIST_KEY_NSPhotoLibraryUsageDescription = "([^"]*)";', pbx).group(1)
html = open('docs/privacy-site/index.html', encoding='utf-8').read()
print('部署页中出现次数:', html.count(s), '(应为 2)')
PY
```

## 部署（GitHub Pages）

1. 仓库 `sunny2023z/FlickRoll-privacy`（**public**，路径大小写别改）
2. 把改动的 HTML / 视频传到仓库根目录
3. `Settings → Pages` → Source 选 `Deploy from a branch` → `main` / `root`
4. 等 1–2 分钟，用上面的 curl 命令核对字节数

## 部署自检

- [ ] 网页能打开，中文正常显示
- [ ] 点「English」能切英文，能切回来
- [ ] 深色模式下中英文都可读
- [ ] 邮箱正确：`1056128378@qq.com`
- [ ] 手机浏览器（不只是电脑）打开正常

> Guideline 2.1 要求 URL 必须 **fully functional**，明确禁止
> "placeholder text, empty websites, and other temporary content"。
> 所以不能用网盘分享链接或占位页。
