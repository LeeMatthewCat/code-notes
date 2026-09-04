Wandb 的替代方法，

同樣用來對模型做監控，但更輕量

---

# swanlab\.init\(\) 物件初始化

```Python
SwanLab物件 = swanlab.init(
    project="專案名",
    experiment_name="此次實驗名", # 用以區分這次跑的模型跟上一次有何不同
    id="此次實驗識別碼", # 預設為 None，自動生成識別碼
    resume="must", # 斷點續訓功能是關閉，預設為 None 即 "never"
    config=dist_name, # 傳入字典，把學習率等參數打包方便日後依照參數篩選
    description="備註",
    mode="offline" # 本地記錄或在雲端看結果 -> "online"
)
```

- resume 為 **"allow" \-\> 如果有 id 就接著舊 id 進行訓練**，**"must" \-\> 比需要有新 id**

| 參數名稱 | 說明 |
| --- | --- |
| **project** | "專案名" |
| **experiment_name** | "此次實驗名"，用以區分這次跑的模型跟上一次有何不同 |
| **id** | "此次實驗識別碼"，預設為 None，自動生成識別碼 |
| **resume** | "must"，斷點續訓功能是關閉，預設為 None 即 "never" |
| **config** | dist_name，傳入字典，把學習率等參數打包方便日後依照參數篩選 |
| **description** | "備註" |
| **mode** | "offline" 本地記錄或在雲端看結果 $\rightarrow$ "online" |
