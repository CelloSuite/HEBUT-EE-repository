# 体育理论考试系统



## 一、概述

简单来说就是，如果题目被收藏，就可以看到题目。利用这个特性可以使用代码进行自动收藏和自动读取题目，从而获得全部题目与答案信息。

---



## 二、自动收藏并弹窗通知（代码1）

### 1.功能概述
该脚本通过 Microsoft Edge 浏览器自动连续访问一系列网页，并对可能出现的 `alert` 弹窗进行确认。其目的可能是批量触发某些页面操作或确保后续爬取过程中不会因未处理的弹窗而中断。

### 2.主要流程
1. 配置 Edge 浏览器的启动选项（使用 Chromium 内核，禁用 GPU 和沙箱）。
2. 启动浏览器，访问首页 `http://tiyu.hebut.edu.cn/` 并等待 10 秒，以确保页面完全加载（可能用于登录或初始化会话）。
3. 设定起始 ID和结束 ID，依次拼接 URL：
   ```python
   http://tiyu.hebut.edu.cn/FrmAddMyRubricSC.aspx?ID={id}
   ```
4. 每访问一个页面，尝试检测并接受 `alert` 弹窗（通过 `Alert(driver).accept()`）。
   - 若弹窗存在，则接受并打印提示信息；
   - 若不存在，则捕获异常并跳过。
5. 每次访问后暂停 0.1 秒，循环结束后关闭浏览器。

### 3.关键点说明
- **弹窗处理**：使用 `Alert` 对象处理 JavaScript 原生弹窗，避免由于未处理的弹窗导致后续操作阻塞。
- **等待时间**：首页等待 10 秒较长，是为了输入账号密码信息；循环内仅等待 0.1 秒，较为激进，实际使用时可适当增加以避免请求过快。
- **注释说明**：代码中注释掉了 `driver.execute_script("window.close();")`，避免了误关闭浏览器主窗口。

---



## 三、抓取页面数据并保存为 CSV（代码2）

### 1.功能概述
该脚本同样使用 Selenium 驱动 Edge 浏览器，访问特定信息展示页面，利用 BeautifulSoup 解析 HTML，提取页面中的多个字段，并将结果保存到桌面上的 CSV 文件中。可用于批量收集评分标准或题目信息。

### 2.主要流程
1. 配置并启动 Edge 浏览器，最大化窗口。
2. 访问首页并等待 10 秒（作用与代码一相同）。
3. 创建 CSV 文件（`rubric_info.csv`）到当前用户的桌面，写入表头：
   `ID, lab, subject, topictype, rubric_doc, option_answer, ok_answer`
4. 设定 ID 范围，循环拼接 URL：
   ```python
   http://tiyu.hebut.edu.cn/RubricInfo/FrmShowRubricInfo.aspx?ID={id}
   ```
5. 每访问一个页面，等待 0.2 秒后获取页面源代码，用 BeautifulSoup 解析。
6. 根据预设的 `id` 定位 `span` 标签，提取：
   - `labRubricDoc` → 评分标准文档
   - `labFlag` → 实验室/标识
   - `labSubjectName` → 科目名称
   - `labRubricType` → 题目类型
   - `labOptionAnswer` → 选项答案
   - `labOKAnswer` → 正确答案
7. 若任一字段未找到（出现 `AttributeError` 或 `TypeError`），则对应字段设为 `"N/A"`。
8. 当提取的 `lab` 字段为空字符串时，跳过写入（`continue`），以避免无效数据。
9. 将有效数据写入 CSV 行，循环结束后关闭浏览器并打印完成提示。

### 3.关键点说明
- **数据提取**：依赖 `BeautifulSoup` 的 `find` 方法精确匹配 `id` 属性，要求页面结构固定，否则可能无法提取数据。
- **容错处理**：使用 `try-except` 捕获定位失败异常，同时增加对 `lab` 为空的过滤，提高了脚本的健壮性。
- **编码与存储**：CSV 使用 `utf-8-sig` 编码，确保在 Excel 中正确显示中文；文件保存路径直接指向桌面，方便查找。
- **等待策略**：每次请求后等待 0.2 秒，平衡了抓取速度与服务器压力，但仍可结合实际响应时间调整。

---


>代码1:  
>```python
>from selenium import webdriver
>from selenium.webdriver.common.alert import Alert
>from selenium.webdriver.edge.service import Service as EdgeService
>from selenium.webdriver.edge.options import Options
>import time
>
># 配置 Edge 启动项（可选）
>options = Options()
>options.use_chromium = True  # 指定使用 Chromium 内核
>options.add_argument('--disable-gpu')
>options.add_argument('--no-sandbox')
>
># 启动 Edge 浏览器
>driver = webdriver.Edge(options=options)
>
># 打开首页
>driver.get("http://tiyu.hebut.edu.cn/")
>time.sleep(10)
>
># 设置起始和结束ID
>start_id = #####
>end_id = #####
>
># 循环访问网页
>for id in range(start_id, end_id + 1):
>    url = f"http://tiyu.hebut.edu.cn/FrmAddMyRubricSC.aspx?ID={id}"
>    driver.get(url)
>
>    print(f"Accessed page with ID {id}")
>
>    try:
>        alert = Alert(driver)
>        alert.accept()
>        print("Notification accepted")
>    except:
>        print("No notification found")
>
>    time.sleep(0.1)
>
># 退出浏览器
>driver.quit()
>
>```

>代码2:  
>
>```python
>import os
>import csv
>import time
>from selenium import webdriver
>from selenium.webdriver.edge.service import Service as EdgeService
>from selenium.webdriver.edge.options import Options
>from bs4 import BeautifulSoup
>
># 配置 Edge 启动项
>options = Options()
>options.use_chromium = True
>options.add_argument('--disable-gpu')
>options.add_argument('--no-sandbox')
>
># 启动 Edge 浏览器
>driver = webdriver.Edge(options=options)
>driver.maximize_window()  # 将浏览器窗口最大化
>
>driver.get("http://tiyu.hebut.edu.cn/")
>time.sleep(10)
>
># 设置起始和结束ID
>start_id = #####
>end_id = #####
>
># 创建 CSV 文件
>desktop_path = os.path.join(os.path.expanduser("~"), "Desktop")
>csv_file = os.path.join(desktop_path, "rubric_info.csv")
>
>with open(csv_file, "w", newline="", encoding="utf-8-sig") as f:
>    writer = csv.writer(f)
>    writer.writerow(["ID", "lab", "subject", "topictype", "rubric_doc", "option_answer", "ok_answer"])
>
>    # 循环访问网页
>    for id in range(start_id, end_id + 1):
>        url = f"http://tiyu.hebut.edu.cn/RubricInfo/FrmShowRubricInfo.aspx?ID={id}"
>        driver.get(url)
>        time.sleep(0.2)
>
>        html = driver.page_source
>        soup = BeautifulSoup(html, "html.parser")
>
>        try:
>            rubric_doc = soup.find("span", id="labRubricDoc").text.strip()
>            lab = soup.find("span", id="labFlag").text.strip()
>            subject = soup.find("span", id="labSubjectName").text.strip()
>            topictype = soup.find("span", id="labRubricType").text.strip()
>            option_answer = soup.find("span", id="labOptionAnswer").text.strip()
>            ok_answer = soup.find("span", id="labOKAnswer").text.strip()
>        except (AttributeError, TypeError):
>            rubric_doc = "N/A"
>            lab = "N/A"
>            subject = "N/A"
>            topictype = "N/A"
>            option_answer = "N/A"
>            ok_answer = "N/A"
>
>        if lab == "":
>            continue
>
>        writer.writerow([id, lab, subject, topictype, rubric_doc, option_answer, ok_answer])
>        print(f"Extracted data for ID {id}")
>
>driver.quit()
>print("Data saved to Desktop/rubric_info.csv")
>
>```
