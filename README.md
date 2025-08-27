<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>自定义协议重定向</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 0;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .container {
            background: white;
            padding: 40px;
            border-radius: 12px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
            max-width: 500px;
            width: 90%;
            text-align: center;
        }

        .icon {
            font-size: 48px;
            margin-bottom: 20px;
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            font-size: 24px;
        }

        .status {
            padding: 15px;
            border-radius: 8px;
            margin: 20px 0;
            font-size: 16px;
        }

        .loading {
            background-color: #e3f2fd;
            color: #1976d2;
            border: 1px solid #bbdefb;
        }

        .error {
            background-color: #ffebee;
            color: #c62828;
            border: 1px solid #ffcdd2;
        }

        .success {
            background-color: #e8f5e8;
            color: #2e7d32;
            border: 1px solid #c8e6c9;
        }

        .link-info {
            background-color: #f5f5f5;
            padding: 15px;
            border-radius: 8px;
            margin: 15px 0;
            word-break: break-all;
            font-family: monospace;
            font-size: 14px;
        }

        .manual-redirect {
            margin-top: 20px;
        }

        .redirect-btn {
            background-color: #1976d2;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 16px;
            margin: 10px;
            transition: background-color 0.3s;
        }

        .redirect-btn:hover {
            background-color: #1565c0;
        }

        .redirect-btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }

        .spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #1976d2;
            border-radius: 50%;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
            display: inline-block;
            margin-right: 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .instructions {
            margin-top: 30px;
            padding: 20px;
            background-color: #f9f9f9;
            border-radius: 8px;
            text-align: left;
            font-size: 14px;
            line-height: 1.6;
        }

        .instructions h3 {
            margin-top: 0;
            color: #333;
        }

        .url-example {
            background-color: #e8e8e8;
            padding: 10px;
            border-radius: 4px;
            font-family: monospace;
            font-size: 12px;
            word-break: break-all;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="icon">🔗</div>
        <h1>自定义协议重定向</h1>
        
        <div id="status" class="status loading">
            <div class="spinner"></div>
            正在解析链接...
        </div>
        
        <div id="linkInfo" class="link-info" style="display: none;"></div>
        
        <div id="manualRedirect" class="manual-redirect" style="display: none;">
            <p>自动重定向失败，请手动点击下方按钮：</p>
            <button id="redirectBtn" class="redirect-btn">打开应用程序</button>
        </div>

        <div class="instructions">
            <h3>使用说明：</h3>
            <p><strong>URL格式：</strong></p>
            <div class="url-example">
                https://zyfang2020.github.io/?link=zotero://select/items/ABC123
            </div>
            <p><strong>支持的协议：</strong></p>
            <ul>
                <li>zotero://</li>
                <li>edge://</li>
                <li>其他自定义协议</li>
            </ul>
            <p><strong>参数说明：</strong></p>
            <ul>
                <li><code>link</code> - 要重定向的自定义协议链接</li>
                <li><code>delay</code> - 重定向延迟时间（秒，可选，默认2秒）</li>
            </ul>
        </div>
    </div>

    <script>
        class ProtocolRedirector {
            constructor() {
                this.statusElement = document.getElementById('status');
                this.linkInfoElement = document.getElementById('linkInfo');
                this.manualRedirectElement = document.getElementById('manualRedirect');
                this.redirectBtn = document.getElementById('redirectBtn');
                
                this.init();
            }

            init() {
                try {
                    const params = new URLSearchParams(window.location.search);
                    const customLink = params.get('link');
                    const delay = parseInt(params.get('delay')) || 2;

                    if (!customLink) {
                        this.showError('未找到要重定向的链接参数。请确保URL中包含 "link" 参数。');
                        return;
                    }

                    if (!this.isValidProtocol(customLink)) {
                        this.showError('无效的协议链接格式。请确保链接以有效的协议开头（如 zotero://、edge:// 等）。');
                        return;
                    }

                    this.showLinkInfo(customLink);
                    this.attemptRedirect(customLink, delay);

                } catch (error) {
                    this.showError('解析URL参数时发生错误：' + error.message);
                }
            }

            isValidProtocol(link) {
                const protocolPattern = /^[a-zA-Z][a-zA-Z0-9+.-]*:\/\/.+/;
                return protocolPattern.test(link);
            }

            showLinkInfo(link) {
                this.linkInfoElement.textContent = `目标链接: ${link}`;
                this.linkInfoElement.style.display = 'block';
            }

            attemptRedirect(link, delay) {
                this.statusElement.innerHTML = `
                    <div class="spinner"></div>
                    将在 ${delay} 秒后重定向到应用程序...
                `;

                setTimeout(() => {
                    try {
                        window.location.href = link;
                        
                        setTimeout(() => {
                            this.showManualRedirect(link);
                        }, 2000);

                    } catch (error) {
                        this.showError('重定向失败：' + error.message);
                        this.showManualRedirect(link);
                    }
                }, delay * 1000);
            }

            showManualRedirect(link) {
                this.statusElement.innerHTML = `
                    <div class="status success">
                        如果应用程序没有自动打开，请点击下方按钮手动打开。
                    </div>
                `;

                this.manualRedirectElement.style.display = 'block';
                this.redirectBtn.onclick = () => {
                    try {
                        window.location.href = link;
                    } catch (error) {
                        alert('无法打开链接：' + error.message);
                    }
                };
            }

            showError(message) {
                this.statusElement.className = 'status error';
                this.statusElement.innerHTML = `❌ ${message}`;
            }
        }

        document.addEventListener('DOMContentLoaded', () => {
            new ProtocolRedirector();
        });

        window.addEventListener('focus', () => {
            const params = new URLSearchParams(window.location.search);
            if (params.get('link')) {
                document.getElementById('status').innerHTML = `
                    <div class="status success">
                        ✅ 重定向成功！应用程序应该已经打开。
                    </div>
                `;
            }
        });
    </script>
</body>
</html>
