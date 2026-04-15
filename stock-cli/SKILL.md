---
name: stock-cli
description: 终端股价查询 CLI，基于 terminal-stocks API
triggers:
  - "stock cli"
  - "terminal stock price"
  - "股价 CLI"
---

# Stock CLI - 终端股价查询

## 方案

使用 [terminal-stocks](https://github.com/shweshi/terminal-stocks) 的免费 web API，无需任何 API key。

```bash
# 直接查询
curl terminal-stocks.dev/MSFT

# 多股票
curl terminal-stocks.dev/MSFT,AAPL,NVDA
```

## 创建 CLI 工具

```bash
mkdir -p ~/bin
```

复制以下代码到 `~/bin/stock`:

```python
#!/usr/bin/env python3
"""stock CLI - 简洁股价查询"""

import sys, urllib.request, re

def main():
    if len(sys.argv) < 2:
        print("用法: stock <代码>")
        sys.exit(1)
    
    ticker = sys.argv[1].upper()
    url = f"https://terminal-stocks.dev/{ticker}"
    
    try:
        with urllib.request.urlopen(url, timeout=10) as f:
            raw = f.read().decode('utf-8')
    except Exception as e:
        print(f"Error: {e}")
        sys.exit(1)
    
    raw = re.sub(r'\x1b\[[0-9;]*m', '', raw)
    
    # 标题
    print(f"{'代码':<10} {'价格':>10}  {'涨跌':>8} {'涨跌幅':>10}  日内区间")
    print("-" * 55)
    
    for line in raw.split('\n'):
        if not line.startswith('│'):
            continue
        parts = line.split('│')
        if len(parts) < 6:
            continue
        
        name = parts[1].strip().split()[0][:10]
        price = parts[2].strip()
        change = parts[3].strip()
        pct = parts[4].strip()
        day = parts[5].strip()
        
        if not price.startswith('$'):
            continue
        
        # 颜色: 红跌绿涨
        if change.startswith('-'):
            col = '\033[31m'
        else:
            col = '\033[32m'
        
        print(f"{name:<10} {price:>10}  {col}{change:>8}\033[0m ({pct:>8})  {day}")
    
    # 时间
    m = re.search(r'UTC.*?\d{4}', raw)
    if m:
        print("-" * 55)
        print(f"📈 {m.group().replace('UTC ', '')}")

if __name__ == "__main__":
    main()
```

配置权限和 PATH：
```bash
chmod +x ~/bin/stock
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 使用

```bash
stock MSFT
stock MSFT,AAPL,NVDA,GOOGL
```

## 输出示例

```
$ stock MSFT,AAPL,NVDA
代码                 价格        涨跌        涨跌幅  日内区间
-------------------------------------------------------
Microsoft         $393.11     $8.74 (    2.27)  $386.52 - $394.69
Apple            $258.83    -$0.37 (   -0.14)  $257.19 - $261.93
NVIDIA           $196.51     $7.20 (    3.80)  $190.80 - $196.51
📈 Apr 14, 2026
```

## 注意事项

- 数据来源 Yahoo Finance，有 15 分钟延迟
- 免费无 API key 要求
- 比 yfinance 快（直接 HTTP）
- 涨跌幅颜色在某些终端可能不显示

## 贡献到社区

如果想贡献到原始项目 ([shweshi/terminal-stocks](https://github.com/shweshi/terminal-stocks))：

```bash
# 1. Fork 原仓库
gh repo fork shweshi/terminal-stocks

# 2. 克隆 fork
gh repo clone 用户名/terminal-stocks

# 3. 添加上游
git remote add upstream https://github.com/shweshi/terminal-stocks.git

# 4. 创建分支，添加你的代码
git checkout -b python-cli
# 添加文件...

# 5. 推送到 fork
git push -u origin python-cli

# 6. 创建 PR 到原仓库
gh pr create --repo shweshi/terminal-stocks \
  --title "Add Python CLI" \
  --body "基于 terminal-stocks web API 的 Python 实现..."
```

## 经验总结

- NPM 包有依赖问题，但 **web API** 直接可用（无需安装）
- 解析 Unicode │ 分隔符时，shell awk/cut 有兼容问题 → 用 **Python** 解决
- 想贡献时先 fork 原仓库，加上游 remote，保持同步