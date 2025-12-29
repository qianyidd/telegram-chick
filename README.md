# **Telegram 账号批量检测工具 (TG Batch Checker)**
示例网站https://tgchks.bzi.pp.ua/
这是一个基于 Web 的轻量级 Telegram 账号状态批量检测工具。采用纯 HTML/JS 编写，界面使用 Tailwind CSS 构建，支持部署在 Cloudflare Pages 上。

本项目前端为纯静态页面，需要配合后端 API 使用。后端 API 可部署在 Cloudflare Workers 上，实现免费、即时的批量检测。

*(建议截图后替换此预览图)*

## **✨ 功能特点**

* **批量检测**：支持一次性粘贴多个用户名或链接（支持换行、逗号分隔）。  
* **状态分类**：自动区分正常（Active）、异常/封号（Banned/Not Found）和请求错误。  
* **数据持久化**：使用 LocalStorage 保存检测列表，刷新页面不丢失数据。  
* **结果导出**：支持一键导出检测结果为 CSV 表格。  
* **UI 美观**：基于 Tailwind CSS 的深色玻璃拟态风格（Glassmorphism）。  
* **API 可配置**：轻松替换后端检测接口，支持自建 API。

## **🚀 部署指南**

该项目分为 **前端（本仓库）** 和 **后端（API）** 两部分。

### **第一步：部署后端 API**

你需要一个 API 服务来处理 Telegram 网页的抓取和状态判断。我们推荐使用开源项目 [telegram-chick-api](https://www.google.com/url?sa=E&source=gmail&q=https://github.com/qianyidd/telegram-chick-api) 直接部署到 Cloudflare Workers。

**方案 A：使用开源 API 项目（推荐）**

后端逻辑已封装在 telegram-chick-api 项目中：

**API 项目地址**: [https://github.com/qianyidd/telegram-chick-api](https://www.google.com/url?sa=E&source=gmail&q=https://github.com/qianyidd/telegram-chick-api)

1. 访问上面的 API 项目仓库。  
2. 按照该仓库的说明，将其部署到你的 Cloudflare Workers。  
3. 部署成功后，你将获得一个 Worker 域名，例如：https://你的名字.workers.dev。  
4. **测试 API**：访问 https://你的名字.workers.dev/check?u=durov (具体路径请参考 API 项目文档)，确认能返回 JSON 数据。

**方案 B：手动创建简易 Worker**

如果你只想快速测试，可以在 Cloudflare 后台创建一个新 Worker，并粘贴以下简易代码：

\<details\>  
\<summary\>点击展开简易 Worker 代码\</summary\>  
export default {  
  async fetch(request, env, ctx) {  
    const corsHeaders \= {  
      "Access-Control-Allow-Origin": "\*",  
      "Access-Control-Allow-Methods": "GET,HEAD,POST,OPTIONS",  
      "Access-Control-Max-Age": "86400",  
    };

    if (request.method \=== "OPTIONS") {  
      return new Response(null, { headers: corsHeaders });  
    }

    const url \= new URL(request.url);  
    const username \= url.searchParams.get('u'); // 假设参数名为 u

    if (\!username) {  
      return new Response(JSON.stringify({ error: "Missing username" }), {  
        status: 400,  
        headers: { "Content-Type": "application/json", ...corsHeaders }  
      });  
    }

    try {  
      const tgRes \= await fetch(\`https://t.me/${username}\`, {  
        headers: { 'User-Agent': 'Mozilla/5.0' }  
      });  
      const text \= await tgRes.text();  
      // 简单判断逻辑  
      const exists \= text.includes('tgme\_page\_title') || text.includes('tgme\_page\_extra');

      return new Response(JSON.stringify({ username, exists }), {  
        headers: { "Content-Type": "application/json", ...corsHeaders }  
      });  
    } catch (err) {  
      return new Response(JSON.stringify({ error: "Error" }), { status: 500, headers: corsHeaders });  
    }  
  },  
};

\</details\>

### **第二步：配置前端**

在部署前端之前，你需要将前端代码中的 API 地址替换为你自己的后端地址。

1. 打开本项目中的 index.html 文件。  
2. 搜索 const apiUrl（大约在第 200 行附近）。  
3. 将默认地址替换为你第一步中部署的 Worker 地址。

// 修改前  
// const apiUrl \= \`https://tgchk.pages.dev/api/check?u=${encodeURIComponent(userObj.username)}\`;

// 修改后 (示例)  
const apiUrl \= \`https://你的api域名.workers.dev/check?u=${encodeURIComponent(userObj.username)}\`;

*注意：请根据后端 API 的实际要求调整 URL 路径（例如是否需要 /api/ 前缀，或者参数名是 u 还是 username）。*

### **第三步：部署前端到 Cloudflare Pages**

1. 将修改后的 index.html 及相关文件上传到 GitHub 仓库。  
2. 登录 Cloudflare Dashboard，进入 **Workers & Pages** \-\> **Pages**。  
3. 点击 **Connect to Git**，选择你的仓库。  
4. **Build Settings（构建设置）**：  
   * **Framework preset**: None (不选)  
   * **Build command**: (留空)  
   * **Output directory**: (如果 index.html 在根目录则留空)  
5. 点击 **Save and Deploy** 完成部署。

## **🛠 本地开发**

1. 克隆本仓库到本地。  
2. 修改 index.html 中的 API 地址。  
3. 直接双击打开 index.html，或使用 VS Code 的 Live Server 插件运行。

## **🤝 贡献与致谢**

* 后端 API 参考/推荐：[telegram-chick-api](https://www.google.com/url?sa=E&source=gmail&q=https://github.com/qianyidd/telegram-chick-api)  
* 界面构建：Tailwind CSS

## **📄 License**

MIT License
