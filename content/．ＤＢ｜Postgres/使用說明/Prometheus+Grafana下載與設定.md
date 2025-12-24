---
theme:
  - default
class: 
paginate: true
header: Prometheus+Grafana下載與設定
---
<style scoped>h1 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size:60px
}
p { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
font-size: 16px;
}
</style>


# Prometheus+Grafana
黃暄惠 2024/11

---
<style>h2 { 
display: flex; 
justify-content: center; /* 水平居中 */ 
align-items: center; /* 垂直居中 */ 
}
</style>
## 安裝 postgres_exporter

1. **下載 postgres_exporter**：
    - 前往 [postgres_exporter GitHub](https://github.com/prometheus-community/postgres_exporter/releases)下載最新版本的安裝包
2. **安裝 postgres_exporter**：
    - 雙擊下載的 MSI 文件，按照提示完成安裝。

---

3. **建立postgres_exporter.yml（可選）**

	參考[postgres_exporter 說明文件](https://github.com/prometheus-community/postgres_exporter)
    
	- `web.systemd-socket` Use systemd socket activation listeners instead of port listeners (Linux only). Default is `false`
	- `web.listen-address` Address to listen on for web interface and telemetry. Default is `:9187`.
	- `web.config.file` Configuration file to use TLS and/or basic authentication. The format of the file is described [in the exporter-toolkit repository](https://github.com/prometheus/exporter-toolkit/blob/master/docs/web-configuration.md). 
...
---

4. **設置環境變數、開啟postgres_exporter：**
	```powershell
	$env:DATA_SOURCE_NAME = "user=yourusername`n" + `
	                        "password=yourpassword`n" + `
	                        "host=localhost port=5432`n" + `
	                        "dbname=yourdatabase sslmode=disable"
	
	.\postgres_exporter.exe --config.file="postgres_exporter.yml"
	```
	![](．紀錄｜PostgreSQL/picture/exporter.png)

---

5. **確認 postgres_exporter 是否正常工作**：
    
    - 在瀏覽器中訪問 `http://localhost:9187/metrics`


---

## 安裝 Prometheus

1. **下載 Prometheus**：
	- 前往 [Prometheus 官方網站](https://prometheus.io/download/) 下載適合 Windows 的版本。
2. **解壓安裝包**：
    - 將下載的 ZIP 檔案解壓到您選擇的目錄，例如 `C:\\Prometheus`。
---

3. **配置 Prometheus**：
	- 在 `C:\Prometheus` 目錄中創建一個名為 `prometheus.yml` 的配置檔案，並添加以下內容：
		```yaml
		global:
		  scrape_interval: 15s
		scrape_configs:
		  - job_name: 'postgres_exporter'
			static_configs:
			  - targets: ['localhost:9187']
		```
	- 這裡的 `targets` 是您將要監控的 postgres_exporter 的地址，可以查看啟動的postgres_exporter紀錄的port。

---

4. **啟動 Prometheus**：
	- 打開命令提示字元，導航到 `C:\\Prometheus`，運行以下命令啟動 Prometheus：
		```powershell
		.\prometheus.exe --config.file=prometheus.yml
		```
		
	- 您可以在瀏覽器中訪問 `http://localhost:9090` 查看 Prometheus 的界面。![](．紀錄｜PostgreSQL/picture/prometheus.png)

---

## 安裝 Grafana

1. **下載 Grafana**：
    - 前往 [Grafana 官方網站](https://grafana.com/grafana/download) 下載適合 Windows 的版本。
2. **安裝 Grafana**：
    - 解壓下載的檔案並按照說明進行安裝。安裝完成後，Grafana 通常會在端口 3000 上運行。
3. **啟動 Grafana**：
    - 打開命令提示字元，導航到 Grafana 的安裝目錄，運行 `grafana-server.exe` 啟動服務。

---

4. **配置 Grafana**：
    - 在瀏覽器中訪問 `http://localhost:3000`，使用預設帳號（admin/admin）登錄。
    - 添加數據源：選擇 "Configuration" > "Data Sources" > "Add data source"，選擇 Prometheus，並填入 URL 為 `http://localhost:9090`。
    - 保存並測試數據源連接。

---

5. **創建儀表板**：
    1. 在[Grafana dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/?search=PostgreSQL)找一個模板
		![width:700px height:500px](．紀錄｜PostgreSQL/picture/dashboard.png)
---
5. **創建儀表板**：
	1. 複製模板ID
	2. 回到自己的server，點左側功能表欄
	4. 單擊儀錶板圖示以導入儀錶板 ID 或上傳我們從 Grafana PostgreSQL 儀錶板下載的儀錶板 JSON 檔。
	![width:700px height:300px](．紀錄｜PostgreSQL/picture/dashboard2.png)

---
### Reference

1. [https://prometheus.io/download/](https://prometheus.io/download/)
2. [https://github.com/prometheus-community/postgres_exporter/releases](https://github.com/prometheus-community/postgres_exporter/releases)
3. [https://github.com/prometheus-community/postgres_exporter](https://github.com/prometheus-community/postgres_exporter)
4. [https://grafana.com/grafana/download?edition=oss&platform=windows](https://grafana.com/grafana/download?edition=oss&platform=windows)
5. [Grafana dashboards | Grafana Labs](https://grafana.com/grafana/dashboards/?search=PostgreSQL)
6. [使用 Prometheus 和 Grafana 監控 PostgreSQL](https://www.ashnik.com/monitoring-postgresql-with-prometheus-and-grafana/)
