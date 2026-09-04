# Dataset 類別

```Python
from datasets import load_dataset
```

負責把**資料轉維張量形式**

**繼承**自他的類別需要實作：

1. **\_\_len\(\)\_\_\(\)：有多少筆數據**

    > ```Python
    > ... len(基於此類別的物件) 
    > ```

2. **\_\_getitem\_\_\(\)：把數據做編號**

    > ```Python
    > ... = 基於此類別的物件[索引]
    > 
    > ... = DataLoader(基於此類別的物件, ...)
    > ```

---

# load\_dataset\(\) 數據載入

```Python
... = load_dataset(
    path="json", # 數據來源（必填）：本地填格式名稱，雲端填路徑
    data_files={"train": "train.jsonl", "test": "test.jsonl"}, # 當 path 填本地時，此處填檔案路徑
    split="train", # 指定要載入哪一個子集（選填）
    name="default", # 子數據集選擇（選填）
    streaming=False, # 串流模式（不下載到本地，選填）
    cache_dir="./my_cache", # 快取路徑目錄（要存在哪，選填）
    trust_remote_code=True # 信任遠端腳本（選填）
)
```

取出來的資料形式：

- **未指定 split **且有區分子集時：為 **DatasetDict 型別**，和**字典**相似

    > \.\.\. = dataset\["train"\] 就能把裡面的訓練集取出來
    > 
    > len\(\) 的結果是有幾個資料集

- 如果**指定 split**：為 **Dataset 型別**，屬於**單一數據得表格**

    > \.\.\. = dataset\[index\] 取出資料（像一個字典），要再對其的 text 鍵取出文字
    > 
    > len\(\) 的結果是有幾行數據

---

# DatasetDict 分段容器基本

> **DatasetDict **是**容器**，而 **Dataset** 是裡面的**內容物**

一個可以管理多個**資料集分段**（Splits）的容器，類似內建的**字典**

以下方式獲得：

1. **load\_dataset\(\)** 匯入訓練資料

2. 另一個 DatasetDict **切割**

3. 從 **CSV、JSON **檔拼接

---

## 屬性

### 查看物件

```Shell
print(raw_datasets)
# 輸出範例:
# DatasetDict({
#     train: Dataset({
#         features: ['sentence1', 'sentence2', 'label', 'idx'],
#         num_rows: 3668
#     })
#     validation: ...
# })
```

---

### keys\(\) 查看分段

```Python
print(資料集名.keys())
# 輸出: dict_keys(['train', 'validation', 'test'])
```

---

### column\_names 查看欄位名

```Shell
print(raw_datasets.column_names)
# 輸出: {'train': ['sentence1', 'sentence2', 'label', 'idx'], 'validation': ...}
```

---

### 存取分段

```Python
train_data = raw_datasets["train"]
```

---

# map\(\) 數據迭代

```Python
... = 資料集名.map(對資料的操作, batched=True, num_proc=預使用 cpu 核心數)
```

1. **自動迭代**

2. **附加新欄位（把回傳的結果放在新的欄位，直接合併到原本的資料集中）**

- **batched=True**：分批次**多線程**處理

- num\_proc=**os\.cpu\_count\(\)**：獲取**最大** CPU 使用

使檔案內新增兩個欄位：input\_ids、attention\_mask，

前者就是轉為 token 的文字，後者則是注意力遮罩，

還有可能會有 token\_type\_ids（視模型而定），用來辨識這是第幾句話

> 當 token 少於設定的截斷 token 前，會補代表上空的 token（0），而對此，注意力遮罩會標記為 0（模型訓練時不需對其思考），反之則為 1
> 
> ```Python
> def tokenize_function(examples):
>     """ 
>     將輸入的資料的 “markdown” 欄位進行 tokenize
>     超過 2048 個 token 的話，就截斷
>     """
>     return tokenizer(examples["markdown"], truncation=True, max_length=2048)
> ```

---

# filter\(\) 條件篩選





---

**\.select\_columns\(\) / \.remove\_columns\(\) 欄位整理**



---

# set\_format\(\) 轉為 tensor











---

# select\_columns\(\) 留下指定欄位

將指定的欄位留下，其餘的刪除

> Tokenzied 後通常會留下舊資料，為了最佳化儲存空間，要將其刪除

```Python
... = 資料集名.select_columns([a, b, ...]) # 單個用字串，多個要用串列包起
```

---

## remove\_columns\(\) 刪除指定欄位

和 select\_columns\(\) 相反，選擇要刪除的欄位

---

# save\_to\_disk 儲存

使用 **Apache Arrow** 格式將資料儲存

```Python
資料集名.save_to_disk("路徑")
```

```Plain Text
my_saved_dataset/
├── dataset_dict.json      # 宣告這個目錄下有哪些子集
├── train/                 # 訓練集資料夾
│   ├── dataset_info.json --> 資料集的詮釋資料 metadata 
│   ├── state.json --> 紀錄處理狀態
│   └── dataset.arrow_1      # 訓練集的實際資料
│   └── dataset.arrow_2
│   └── ...
└── test/                  # 測試集資料夾（要看資料集裡面有沒有）
    ├── dataset_info.json
    ├── state.json
    └── dataset.arrow_1    # 測試集的實際資料   
    └── dataset.arrow_2
    └── ... 
```







---

