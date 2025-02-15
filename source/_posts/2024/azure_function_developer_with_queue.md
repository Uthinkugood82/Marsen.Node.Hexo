---
title: " [實作筆記] Azure Function Queue Trigger 開發(以Python為例) "
date: 2024/09/09 17:47:01
---

## 前情提要

Azure Functions 提供了在雲端執行無伺服器函數的強大能力，  
但在本地環境中開發和測試這些函數可以大大提高開發效率。  
為了模擬 Azure Storage 服務，我們可以使用 Azurite，這是一個 Storage 的本地模擬器。  
本文將記錄我如何在本機上設置建立本機的 Azurite Queue 進行開發。

註:本文假設你已具備[建立 Azure Functions 的前置基礎](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-azure-developer-cli?pivots=programming-language-python&tabs=linux%2Cget%2Cbash%2Cpowershell)

## 實作記錄

### 1. 建立和設定本機開發環境

#### 安裝 Azurite

Azurite 是用於模擬 Azure Storage 服務的本地工具，您可以通過以下命令進行安裝：

```terminal
npm install -g azurite
```

#### 啟動 Azurite

啟動 Azurite 並指定 Storage 位置和日誌文件。

下面的語法會建立 `.azurite` 資料夾為 Azurite 的默認資料夾，  
你可以根據需要修改路徑或刪除並重建此資料夾：

```terminal
azurite --silent --location ./.azurite --debug ./.azurite/debug.log
```

啟動後，你會看到以下輸出，表示 Azurite 成功啟動並監聽相關端口：

```terminal
Azurite Blob service is starting at http://127.0.0.1:10000
Azurite Queue service is starting at http://127.0.0.1:10001
Azurite Table service is starting at http://127.0.0.1:10002
```

#### 設定本機 Azure Storage 連線字串

為了方便操作本機 Azure Storage，  
我們需要設置 AZURE_STORAGE_CONNECTION_STRING 環境變數：  
這裡要查看[微軟官方文件取得地端連線字串](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite?tabs=visual-studio%2Cblob-storage#connect-to-azurite-with-sdks-and-tools)  
你們可以看到它包含了一組 AccountKey 與 Account(devstoreaccount1)
這個例子中我們只使用了 Azurite Queue Service  

```terminal
export AZURE_STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;"
```

開發完成後，記得刪除此連線字串：

```terminal
unset AZURE_STORAGE_CONNECTION_STRING
```

#### 常用語法

檢視所有 Queue

```terminal
az storage queue list
```

建立 Local Queue

```terminal
az storage queue create --name myqueue
```

建立資料到指定 Queue 中, 下面的例子會建立一包 JSON 檔，當然你也可以使用純文字(text)

```terminal
az storage message put --queue-name myqueue --content "{\"message\": \"Hello, World\!\", \"id\": 123, \"status\": \"active\"}"
```

顯示前 5 筆指定 Queue 中的資料

```terminal
az storage message peek -q myqueue --num-messages 5
```

取出 Queue 中的資料

```terminal
az storage message get --queue-name myqueue --num-messages 1
```

刪除 Queue 中的資料

```terminal
az storage message delete --queue-name myqueue --id <message-id> --pop-receipt <pop-receipt>
```

補充說明:

popReceipt 是 Azure Queue 中用來確認消息取出和刪除操作的唯一識別碼。  
當取出消息時，Azure 會返回 popReceipt，確保只有取出的客戶端能夠刪除該消息。  
如果 popReceipt 顯示為 null，通常表示消息尚未取出或命令不正確。  
要獲取 popReceipt，使用 az storage message get 命令取出訊息。

### 2. Azure Functions 的本機開發與執行

啟動 Azure Functions

當本機環境設置完成後，你可以使用以下命令來啟動 Azure Functions：

```terminal
func start
```

這條命令會啟動你的本地 Azure Functions 執行環境，使你可以在本機上測試和調試你的函數。

### 3. 加碼，代碼檢查和格式化

#### 安裝與配置 Pylint

Pylint 是一個 Python 代碼靜態分析工具，可以檢查代碼中的錯誤和不符合最佳實踐的地方。首先，安裝 Pylint：

```terminal
pip install pylint
```

配置 Pylint，創建或修改 .pylintrc 文件來包含你的檢查規則：

```conf
[MESSAGES CONTROL]
disable=C0114,C0115,C0116

[FORMAT]
max-line-length=120
```

使用 Pylint 進行代碼檢查

運行以下命令來檢查所有 Python 文件：

```terminal
pylint *.py
```

#### 安裝與使用 Black 進行代碼格式化

Black 是一個自動格式化 Python 代碼的工具，能夠保持代碼風格的一致性。首先，安裝 Black：

```terminal
pip install black
```

格式化整個專案的所有 Python 文件：

```terminal
black .
```

整合檢查和格式化工具

你可以將代碼檢查和格式化工具整合到一個命令中，這樣可以簡化工作流程：

```terminal
pylint *.py && black .
```

#### 例外

在 Azure Functions 中，某些函數參數不符合 Pylint 的命名規則，這可能會導致部署失敗。你可以忽略這些特定的 Pylint 警告，例如：

```python
# pylint: disable=C0103
def dispatch_worker(inputQueue: func.QueueMessage, watchQueue: func.Out[str]):
```

## 問題與排除

- 如果 Azurite 無法啟動，請檢查是否已正確安裝 Azurite 以及是否有其他應用程序佔用了相關端口。
- 如果 Azure Functions 無法啟動，請確保所有相關的配置文件和依賴項已正確設置。

## 參考

- [Azurite 官方文檔](https://github.com/Azure/Azurite)
- [Azure Functions 本地開發指南](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=macos%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-python)

(fin)
